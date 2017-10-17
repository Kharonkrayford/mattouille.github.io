---
title: "S3 Bucket Security and Best Practices"
layout: post
date: 2017-10-16 20:52
headerImage: false
tag:
- security
- aws s3
star: true
category: blog
author: mattouille
description: I cover S3 bucket ACL's and policies and a definitive strategy for using them.
---

There's been a litany of companies with unsecured S3 buckets including [Verizon](https://www.theregister.co.uk/2017/09/22/verizon_falls_for_the_old_unguarded_aws_s3_bucket_trick_exposes_internal_system/), [Accenture](https://www.theregister.co.uk/2017/10/10/accenture_amazon_aws_s3/), [TimeWarner](https://threatpost.com/four-million-time-warner-cable-records-left-on-misconfigured-aws-s3/127807/), and the list goes on.

## So what is a vulnerable S3 bucket?

### ACLs and policies

S3 has some quirks. First and foremost that I've identified is it's _permissions system_ known as _[ACL's](http://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html)_ and _[Policies](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-iam-policies.html)_. I'm going to kick this off with **vanilla S3 buckets DENY by default.** There's a nice little ACL that comes with every bucket that allows access account wide and that's it. If you create an S3 bucket and do nothing else with it, this bucket is largely secure (unless someone gains access to your account, but that's out of scope here).

ACL's are just as they sound, they're _access control lists_ where you can grant permissions or denies to accounts, users, groups, etc...

Policies are very fine grained, expressive, and written in JSON. They're very comparable to ACL's, except they require referencing _existing_ resources with ARN's (Amazon Resource Names).

**So what's the difference?** [Amazon regards S3 ACL's as legacy](https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/), so I don't use them aside for the given ACL (_ALLOW_ to resources in your account). In fact, in most cases I'll remove that ACL and define a policy to reflect it if it's needed. The reason I do this is maintaining a _single source of truth_. **In many cases if you mix ACL's and policies determining the outcome can become very tricky and cumbersome.** Policies reflect on all the objects in an S3 bucket, Amazon phrases this as they're _postured at the bucket level_. **This means if you find yourself in a predicament where some objects need a policy and others don't they should likely be in different buckets.**

Yes, I know you can apply policies to folders, but really in the age of IaaS I'm sure you've discovered this can be cumbersome as well. Let's examine that scenario below:

Let's say I'm using Terraform (or CloudFormation) to provision an S3 bucket. The whole goal with IaaS is to provision and require _zero_ manual steps. I define my S3 Bucket, next I need to create a policy and specify the bucket it attaches to. _Keep in mind I can only reference things in my policy that already exist_. I would actually have to create a script that goes and creates a folder and then executes the policy creation and attachment. In this case it's not only easier but safer to provision different buckets for different policies.

### Reining in the policies

One of the unfortunate things that I see frequently is engineers or developers using `Principal: "*"` in policies. _Principal_ is the _who_ in _**who** can use **what**_ while _Resource_ refers to _what_. The `"*"` actually indicates that _any resource_ can use the bucket you've assigned this to, including resources or users outside your account. I can literally log onto another computer with AWS CLI installed and read or post files to your S3 bucket if your policies aren't specified correctly.

> You can try this by using a computer with no (or different) AWS credentials and the CLI installed. Simply create a file, apply a junk principal policy and copy it in with `aws s3 cp file.txt s3://yourbucket/`. Voila!

`Principal: "*"` is useful if you have a website hosted off of Amazon S3. I might use a policy like this:

 ```json
 {
   "Version":"2012-10-17",
   "Id":"http referer policy example",
   "Statement":[
     {
       "Sid":"Allow get requests originating from www.example.com and example.com.",
       "Effect":"Allow",
       "Principal":"*",
       "Action":"s3:GetObject",
       "Resource":"arn:aws:s3:::examplebucket/*",
       "Condition":{
         "StringLike":{"aws:Referer":["http://www.example.com/*","http://example.com/*"]}
       }
     },
      {
        "Sid": "Explicit deny to ensure requests are allowed only from specific referer.",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": "arn:aws:s3:::examplebucket/*",
        "Condition": {
          "StringNotLike": {"aws:Referer": ["http://www.example.com/*","http://example.com/*"]}
        }
      }
   ]
 }
 ```

 In this example not only have I attributed an explicit _ALLOW_ but I've also attributed an explicit _DENY_.

 ### Reciprocity is worth mentioning

 A lot of folks that I find making unsafe S3 policies are doing so because they can't figure out the mess between ACL's and Policies (which I've now explained) as well as reciprocity between _IAM Policies_ and _Bucket Policies_. Anything that you reflect in your _Bucket Policy_ should reflect in your _IAM Policy_. IAM Policies are used conjoiningly with STS:AssumeRole and Trusts. If I grant _s3:GetObject_ in my Bucket Policy, I need to grant _s3:GetObject_ in my IAM Policy TO that resource. **AGAIN**, make it to the specific resource rather than `"*"` unless you have a very good reason.

 ### Lock everything down to roles

 I always make a habit of **referencing roles with Principals**. A principal can be assigned to a variety of things such as an _account_, _user_, _role_, and _service_. That said, just because you _can_ doesn't mean you _should_. Rather than giving all your EC2 instances access with `Service: ec2.amazonaws.com` specify the rule and assign an STS Trust to that IAM Profile. You could equate this to the "Principal of least privilege", much like buckets and bucket permissions could be described as "Doing one thing and doing it well".

### KMS and SSE

AWS provides the _Key Management Service_ (For everything) and _Server Side Encryption_ (For S3 Buckets). When you upload and download files/objects you can specify an AWS KMS Key to use. Furthermore you can restrict keys using policies! The same approach you used with bucket policies can be used with KMS Key policies, restricting them to certain roles is _always_ a hardy best practice. This keeps everything segregated but also continuing to flow nicely without stoppages. Additionally, should an instance or resource with a particular role ever be compromised you can simply roll that _one_ KMS key. You can refer to AWS KMS Key's using Bucket Policy conditionals.

```json
{
   "Version":"2012-10-17",
   "Id":"PutObjPolicy",
   "Statement":[{
         "Sid":"DenyUnEncryptedObjectUploads",
         "Effect":"Deny",
         "Principal":"*",
         "Action":"s3:PutObject",
         "Resource":"arn:aws:s3:::YourBucket/*",
         "Condition":{
            "StringNotEquals":{
               "s3:x-amz-server-side-encryption":"aws:kms"
            }
         }
      }
   ]
}
```

The above policy allows you to use _any_ or the _default_ KMS key, however, you can further specify keys in your conditional: `"s3:x-amz-server-side-encryption-aws-kms-key-id": "arn:aws:kms:region:acct-id:key/key-id"`.

I won't delve too much into [KMS Key Policies but they can be explained here](http://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) and, again, are much like the bucket policies I explained above.

## Related Links
* [Hacking S3 Buckets](https://blog.rapid7.com/2013/03/27/open-s3-buckets/)
