---
title: "Using return values inside CloudFormation UserData"
layout: post
tags:
- AWS
- CloudFormation
category: [DevOps, AWS]
excerpt: "I've been working on a project that uses CloudFormation exclusively, so I don't get to do variable interpolation that's as simple as Terraform makes it. Thus, I've had to do some improvising when generating files based off my infrastructure orchestration."
---

I've been working on a project that uses CloudFormation exclusively, so I don't get to do variable interpolation with return values that's as simple as Terraform makes it. Thus, I've had to do some improvising when generating files based off my infrastructure orchestration.

Unfortunately CloudFormation is not as versatile as AWS would like you to believe, though it does constantly improve. One of the things I really like having is the ability to drop some return values in my UserData that's based on my infrastructure orchestration. Chiefly the problem arises when I'm doing things from a _reusable infrastructure_ perspective, so ideally this same CloudFormation template could be brought anywhere.

Traditionally I think people would tell you to separate the code and just create two stacks, but this doesn't make sense from a reusability standpoint and is honestly quite confusing to look at. An example of this is when I'm creating an RDS instance for use with an ECS cluster. The first reason is that this RDS instance is quite custom to what I'm doing, it's meant to be low cost and therefor does not setup clustering. I could spend all my time making an extremely intuitive RDS template but that's a lot of work to just solve for variable interpolation.

The second reason is inside my repositories I like my code to actually reflect what the environment looks like, unilaterally it is the single source of truth. Anything outside this truth should not be present. That said, I group my templates by purpose rather than abstractly (Like, rds.yaml with some kind of generified overrides file to be my single source of truth). I started playing with some ways in YAML that CloudFormation would allow me to do some interpolation and found one that works quite well. Although this articlew as purposed for return values on existing objects within a CloudFormation template, you could actually create variables with some custom logic.

{% highlight yaml linenos %}
DB:
...
ContainerInstances:
  Type: AWS::AutoScaling::LaunchConfiguration
  Properties:
    ...
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash -xe

        mkdir -p /opt/anchore

        cat << EOF > /opt/anchore/config.yaml
          database:
            db_connect: 'postgresql+pg8000://${DBUser}:${DBPassword}@$<AWS dns friendly url>:<database port from AWS>/postgres'
            db_connect_args:
              timeout: 120
              ssl: False
            db_pool_size: 30
            db_pool_max_overflow: 100
        EOF

        echo ECS_CLUSTER=${ECSCluster} >> /etc/ecs/ecs.config
        yum install -y aws-cfn-bootstrap
        /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource ECSAutoScalingGroup --region ${AWS::Region}
{% endhighlight %}

An explanation of what's happening here:

* Line 1: _DB_ is my RDS instance that I've created. It has some sweet [return values](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html#aws-properties-rds-database-instance-returnvalues) where I can get the AWS friendly URL and port.
* Line 8: Historically speaking I use the `!Sub |` short text to do multiline UserData.
* Line 15: This is where it'd be nice to have those return values, namely the Endpoint.Address one.
* All the other variables you see here are normal Parameters and can be used with !Sub.

The solution I came up with was to escape from `!Sub |`. Give this a good once over and I'll explain the changes:

{% highlight yaml linenos %}
DB:
...
ContainerInstances:
  Type: AWS::AutoScaling::LaunchConfiguration
  Properties:
    ...
    UserData:
      Fn::Base64:
        Fn::Sub:
          - |
            #!/bin/bash -xe

            mkdir -p /opt/anchore

            cat << EOF > /opt/anchore/config.yaml
              database:
                db_connect: 'postgresql+pg8000://${DBUser}:${DBPassword}@${DBAddress}:${DBPort}/postgres'
                db_connect_args:
                  timeout: 120
                  ssl: False
                db_pool_size: 30
                db_pool_max_overflow: 100
            EOF

            echo ECS_CLUSTER=${ECSCluster} >> /etc/ecs/ecs.config
            yum install -y aws-cfn-bootstrap
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource ECSAutoScalingGroup --region ${AWS::Region}
          - {
            DBAddress: !GetAtt DB.Endpoint.Address,
            DBPort: !GetAtt DB.Endpoint.Port
            }
{% endhighlight %}

* Line 9: I've changed to use the JSON form of !Sub (Fn::Sub). This is thanks to the versatility of YAML really.
* Line 10: I made a YAML list with a multiline bar (\|)
* Line 17: Check out those shiny new Address and Port variables, then look at Line 29-30.
* Line 28: I now open up some curly braces, this is essentially allowing me to jump back into CloudFormation.
* Line 29-30: I can now use !GetAtt as normal with a custom variable name, pointing to that _DB_ Object I create in Line 1.

With other variables, outside of return values, you could actually do some basic logic and return different values depending on other values, etc... It's a neat way to get the data you need in without creating excessive stacks so you can utilize _Outputs_.
