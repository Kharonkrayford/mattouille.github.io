---
layout: post
title: "Let's talk DevOps job titles"
excerpt: "I've watched the community struggle on and off with job titles for those working in the DevOps market. DevOps is and always will be simply a set of philosophies, beyond that are three major systems-type fields I see companies using DevOps in: Operations, Platform Engineering, and Release Management."
categories: Opinions
tags: [DevOps, Site Reliability Engineering]
comments: true
---

I've watched the community struggle on and off with job titles for those working in the DevOps market. DevOps is and always will be simply a set of philosophies, beyond that are three major systems-type fields I see companies using DevOps in: Operations, Platform Engineering, and Release Management.

#### Site Reliability Engineering

To avoid some initial confusion I'm going to address the topic of _Site Reliability Engineering_. For those that've [read the book](https://landing.google.com/sre/book.html) and heard the hype you'll realize that SRE is Google's implementation of _Operations_. It's best simply described as Operations through the lense of a _Software Engineer_.

If you've worked in software first companies before then Google's approach probably sounds no different than what you've typically understood Operations to be. Unfortunately, much of the rest of the world has fundamentally not worked this way, hence DevOps has been such an awakening for many.

SRE also does not mean that a Software Engineer is suddenly considered more capable than someone who has worked in Systems Engineering at troubleshooting the common infrastructure problems in distributed systems. It simply means that people of Systems Engineering backgrounds must _be capable of resolving issues through software and building process/automation_. 

This is a departure from the days of just coding scripts to solve problems, rather you should know some fundamentals of building software to help you solve systems issues. Again, it doesn't mean you need to a Senior Software Engineer; but you should be good at developing tooling, maintaining repositories, working with API's (both system/RPC and REST), working with/on automation frameworks, etc...

There are companies such as Facebook and Amazon that practice disciplines close in theory to SRE; however, they are a fundamental departure from that role and thusly named differently. If you change the meaning of the position then consider changing the position title to something more generic as my point is that SRE is quite specific.

#### Operations

Companies _are_ wanting Operations folks to reinvent themselves. As more companies adopt distributed systems the days of being able to hand-spin infrastructure effectively and efficiently are ending. The cloud and abstracted datacenters have additionally presented new challenges, tools, and paradigms to work within. That said, not every company is looking for SRE level talent and nor they should be.

A lot of times companies are looking for someone who can basically operate the CI/CD tooling, has a moderate understanding of the chosen OS command line, and the systems knowledge to snuff out common problems. There are many scenarios where this sort of model works fine, though in my own personal opinion, as software continues to grow in complexity I'm not quite sure of this models longevity.

#### Platform Engineering

This term is quite unique to the cloud, however, it doesn't have to be. Any place where developers need a platform to deploy on this position is applicable.

What's a platform you may ask? It's the combination of infrastructure for running software such as service discovery, networking, artifact storage, datastores, etc on (currently) virtual machines or containers. There's some prebuilt stacks out there such as HashiStack, SaltStack, VMWare, OpenStack, Pivotal Cloud Foundry, Nginx has their own, and last but certainly not least Kubernetes.

It boils down to providing developers an abstracted endpoint to deploy their product on while needing to maintain only a moderate degree of infrastructure knowledge. Platforms are versioned products just like software that consist of infrastructure and code to glue the process together.


#### Release Management

One of the key things you might notice is I left the "Engineering" title off of this one. That's because, to me, engineering is quite a specific title. I'm the type of person who draws a distinction between _Software Development_ and _Software Engineering_. Really it comes down to a _need_ for precision versus a _want_ for precision. In any case, I don't mean to imply that anyone is a less capable person without the title Engineer; just that it applies to a specific practice.

Release Management _is_ important. In many software first organizations release cadence is constant and happening concurrently with other releases. I personally view this as a function of leadership within an extreme programming (or agile software development) team rather than a position in itself, however, it is nonetheless a very common position among Fortune 500's. 

You need to be familiar with the _Software Development Life Cycle_ and CI/CD. Some positions actually consist of these people writing pipelines for teams or pipelines used by many teams (although personally I believe this is an anti-pattern), others it is more relatable to _Release Coordination_, or making sure all the i's are dotted and the t's are crossed for a release. I think much of the way that enterprises view Release Management are characterized to what the software industry calls release readiness checklists. 

#### Conclusions

There are some things I didn't cover, which is that there's actually a massive software side to DevOps. I didn't cover it because there's not as much confusion or question around what the software side of DevOps is. 

My goal and intent here is to hone what these positions are so we have a common language to speak. By having a common language we make it easier to convey interests in jobs and the expectations of a minimum criteria.