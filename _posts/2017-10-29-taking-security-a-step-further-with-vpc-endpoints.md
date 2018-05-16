---
title: "Taking Security a Step Further with VPC Endpoints"
layout: post
tags:
- S3
- AWS
- Security
category: [Tutorials]
comments: true
---

Ever since I wrote [S3 Bucket Security and Best Practices]({{ site.baseurl }}{% post_url 2017-10-16-s3-bucket-security-and-best-practices %}) I've been playing with how to extend security within a given AWS account.

Policies and KMS are great but to me assurance comes in threes. That said I don't see VPC Endpoints discussed very much when people are talking about S3 Bucket Security, especially for something that's served over a public API!

<!--more-->

Consider the following diagram, where the blue line is how traffic normally reaches S3 and red is how traffic flows when using a VPC Endpoint:

![VPC Endpoint Diagram][VPC Endpoint Diagram]

**Scenario**

1. I have an EC2 instance that utilizes CloudInit and needs some shell files to boot and appear healthy. This instance has no public address and utilizes a NAT gateway to get to the internet.
2. My shell files are stored in S3 because they're, of course, tested and uploaded as part of my CI/CD process.
3. EC2 instance boots from an AMI and executes UserData. CloudInit makes the call to fetch my shell scripts.
4. A HTTPS request is sent from my EC2 instance to my S3 Bucket for the object.
5. The request is picked up by my NAT Gateway, which is in a public subnet, due to the requests destination address being public.
6. S3 receives the HTTPS request, responds with the object and it makes it's way back to the EC2 instance in the reverse order.

What if I told you that you can now skip step 5? Welcome to VPC Endpoints. Let's look at some of the benefits of VPC Endpoints, specifically with regard to Amazon S3:

* Skipping the NAT Gateway
* S3 is directly accessed by a private address
* Filter access to S3 based on VPC or instance.
* Traffic never leaves Amazon's network, therefore for large organizations, the cost savings are dramatic.

_**Warning:** if you implement this form of security, you cannot do cross-region requests and requests can only be in IPv4._
{: .notice--warning }

Let's look at that last point because it's what gets me most excited. Here's the words directly from Amazon:

> Use your route tables to control which instances can access resources in Amazon S3 via the endpoint.

> For bucket policies, you can restrict access to a specific endpoint or to a specific VPC

That's three different ways I can control access and from two different perspectives. VPC Endpoints can be applied to select or all route tables. While I do like to maintain a single source of truth this, at least the second part, is Amazon's effort to make access to S3 Buckets from EC2 instances fall in line with the way they treat relationships between IAM and S3 Buckets. Basically the concept boils down to reciprocation so be especially careful to be clear and concise.

### Bucket Policy via VPC Endpoint

_**Note**: Keep in mind every subnet has it's own route table. Every AZ also has a subnet._
{: .notice--info }

_**Note**: You cannot use aws:sourceVpc to authenticate addresses through a VPC Endpoint. This will fail._
{: .notice--info }

I love this because I like having bucket policies as my single source of truth, they're very expressive and easy to understand. Let's get started.

{% highlight json %}
{
  "Version": "2012-10-17",
  "Id": "Policy1415115909152",
  "Statement": [
    {
      "Sid": "Access-to-specific-VPCE-only",
      "Principal": "*",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-1a2b3c4d"
        }
      }
    }
  ]
}
{% endhighlight %}

The above policy example restricts access to a specific VPC Endpoint. This is the equivalent of restricting access to a specified VPC. I have not tested this with cross account VPC Endpoints in the same region, but I didn't read anything that says you can't do it. I imagine this could be extremely useful for those using Amazon Organizations. This allows you to be very specific about which route tables/subnets would have access.

### Bucket Policy via VPC

{% highlight json %}
{
  "Version": "2012-10-17",
  "Id": "Policy1415115909152",
  "Statement": [
    {
      "Sid": "Access-to-specific-VPC-only",
      "Principal": "*",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpc": "vpc-111bbb22"
        }
      }
    }
  ]
}
{% endhighlight %}

A little bit more generally we can strict the bucket policy to a given VPC. If both your public and your private subnets need access to a bucket, then this is a good method to use.

### Endpoint Policy

This works in a similar way that IAM policies do with S3 Bucket Policy.

{% highlight json %}
{
  "Statement": [
    {
      "Sid": "Access-to-specific-bucket-only",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"]
    }
  ]
}
{% endhighlight %}

Every VPC Endpoint has a policy attached to it. In this case you can restrict the buckets that can be accessed through this policy. I think this is a good thing to do regardless of your circumstance.

### Multiple VPC Endpoints

Another strategy is to have multiple VPC endpoints even for the same service. While route tables cannot overlap you could segregate which route tables/subnets have access to privileges. You then check the VPC Endpoint inside the bucket policy and assign it's privilege.

I think by now we've learned that systems being compromised is almost inevitable, whether it's an employee at the company, a customer, deliberate, or unintentional - it doesn't matter. Using VPC Endpoint policies helps limit the effect that compromise can have on your environment. When played in combination with expressive bucket policies and KMS keys we're able to hone the instances access to a limited subset of actions and resources. Identifying least privilege is often the easy part, enforcing it is where the work comes in.

I hope this has been an educational caveat on my previous article, as always if you have questions/comments/concerns you can get a hold of me on Reddit, LinkedIn, or IRC!

[VPC Endpoint Diagram]: /img/posts/2017-10-29-taking-security-a-step-further-with-vpc-endpoints/VPC%20Endpoint%20Diagram.png
