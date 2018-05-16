---
title: "Splitting up Terraform & state file separation"
layout: post
tags: [Infrastructure, DevOps, Terraform]
category: [Workshops]
comments: true
---

Terraform is fundamentally operated off of these things called "state files". State files literally are the single source of truth, as far as Terraform is concerned, in regard to your infrastructure. They're formatted in regular JSON but carry a lot of metadata that helps Terraform plan and predict what will happen with your infrastructure.

All infrastructure orchestrators have to work off of some sort of state. Interpreting live infrastructure state is [complicated and sluggish](https://github.com/hashicorp/terraform/issues/10474) at scale so many of them resort to some sort of centrally hosted file system that indicates this state to them. CloudFormation actually does much of the same thing with CloudFormation templates in S3 Buckets, you just see less of it exposed to you.

What Terraform really enables is an organization of infrastructure that can be reused, version controlled, and browsed by anyone in the organization. Nobody goes to the S3 bucket and views the Terraform state file, rather, they view the code that generated that state file hence it's important not to make manual modifications to infrastructure once an orchestrator has run.

<!--more-->

#### State files: some things to know

State files as you may have guessed are almost entirely plain text. Terraform does not attempt to do any encryption for you, rather Terraform assumes that you would handle this on a remote state backend and even combine it with things like CredStash or Vault.

Due to the nature of state files it is possible to have two users execute on the same state file at once which is pretty problematic. To solve for this Terraform introduces [locking](https://www.terraform.io/docs/state/locking.html). How locking is handled is entirely dependent on whatever backend you choose but more or less the state file is locked so no other user can modify it. This works in the same way that earlier VCS's did.



#### State file contents and jargon

Let's take a deep dive into some Terraform state files. Below is the basic outline of a state file.

```json
{
    "version": 3,
    "terraform_version": "0.11.5",
    "serial": 7,
    "lineage": "xxx",
    "modules": [
        {
            "path": [
                "root"
            ],
            "outputs": {

            },
            "resources": {

            },
            "depends_on": []
        }
    ]
}
```

Immediately we can see some things we'd recognize right away such as "terraform_version", "outputs", and "resources". Don't focus on the word "modules" too much as it doesn't really mean the same thing as an actual Terraform Module. So far though, this data is fairly readable.

Other things like "version", "lineage", "serial" may not be so obvious:

> "Version" indicates the format in which the state file is maintained which allows backward compatibility to older state files and the ability to move state files forward as those versions progress.

> "Serial" meanwhile is a sort of versioning system in that if the serial in the destination state is higher than that of the state being pushed, Terraform will reject that change.

> "Lineage" actually identifies the state file in a unique way which provides protection against accidentally pushing one state file and overriding another on a remote backend.

#### State file separation

Consider the directory and file structure below.

```
terraform_code
|_ global
   |_ iam
      |_ terraform.tfstate
   |_ s3
      |_ terraform.tfstate
|_ modules
|_ shared
   |_ vpc
      |_ terraform.tfstate
   |_ ci_cd
      |_ terraform.tfstate
|_ database
   |_ rds
      |_ terraform.tfstate
|_ prod
   |_ vpc
      |_ terraform.tfstate
   |_ service_discovery
      |_ terraform.tfstate
   |_ secrets_management
      |_ terraform.tfstate
   |_ app
      |_ terraform.tfstate
|_ qa
   |_ vpc
      |_ terraform.tfstate
   |_ service_discovery
      |_ terraform.tfstate
   |_ secrets_management
      |_ terraform.tfstate
   |_ app
      |_ terraform.tfstate
```

Some astute Terraformers may be asking why I'm talking about directory separation before I bring up something like workspaces. Workspaces to me are like environment separation. They create entirely different versions of your infrastructure and what I'm about to get into is a bit of *environment-ception*. In infrastructure management we have two basic things we're dealing with:

1. Environments that software exists in. In an effort to keep these things clear I'm going to refer to these as "staging environments". This for instance is "prod" where I run production code and "stage" where I run staging code.
2. The environment stage of a particular Terraform directory. This is what you'll read more about in Workspaces. This basically implies, "I want to work on the QA VPC, but I want to do so without creating excessive directories and I want to be able to just kind of switch into it." If you're familiar with Trunk Based Development, workspaces should click for you really easily as the flow is moderately the same.

Now that we've got that out of the way we can discuss the separation of state files themselves. On it's facade state file separation may not seem inherently important, however, when you begin to use variables to affect large swaths of infrastructure it can get very important. One key use case is that when you have to "recreate" a VPC or subnet it actually causes all instances in that subnet to be recreated. One adverse affect of modifying a variable could be that you could unintentionally recreate a subnet or VPC and experience an unintentional and unmanaged outage. These kind of use cases become more and more common as your infrastructure grows larger and more complex.

**Gotta keep 'em separated**. Thanks to Terraform Data we can actually grab remote state and link it to existing infrastructure that's stored in a totally different state file. This makes it seem as if we're using one big Terraform document when we're actually not at all. In fact, we have the safety to assure that our `terraform destroy` on the app state file won't go suddenly blow away the vpc state file just because it uses information from it.

In the below example I'm wanting to get information on a VPC. In my remote state file for my Shared VPC I actually output the VPC ID.

```perl
data "terraform_remote_state" "shared_vpc" {
  backend = "s3"
  config {
    bucket = "terraform-state.mydomain.com"
    key    = "shared/vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

data "aws_vpc" "shared_vpc" {
  id = "${data.terraform_remote_state.shared_vpc.vpc_id}"
}
```

Now my `data.aws_vpc.shared_vpc` gives me all the arguments from [here](https://www.terraform.io/docs/providers/aws/d/vpc.html).

While it does slightly increase complexity it relieves some major pain in the long run.

#### Workspaces

Given how I thought about state file separation it's now worth some time to explain what I think of workspaces. In infrastructure I'm creating staging environments where developers put different levels of *their* code, but where do I put different levels of *my* code to verify my changes are non-breaking? Aha! Such a thing does exist and intro workspaces.

The default workspace is quite literally called "default". I suggest running production code in this workspace as it is the one workspace that can never be deleted. In your code you can always call `terraform.workspace` in order to adjust counts, however, names will never need to be custom for workspaces. Instead, a new workspace sees totally new infrastructure and is synced to the remote state file (in the case that you're using one). The nice thing here is that deleting one workspace will never affect another and you can continue to reference other live infrastructure since you're directory separated, so you don't get any infrastructure dependency hell where you're creating infinite resources just to try things out.

While this doesn't really count as state file *separation* per se it is separation of environments for different purposes. In one case you're separating for *dev* code and in this case you're separating for *your* code.

#### Conclusion

Hopefully you've learned a bit on state files, separation through directories, workspaces, and why they're important to be used together. As always if you have questions or want to tell me I suck, I'm freely available on Twitter at @codencombovers or on LinkedIn!
