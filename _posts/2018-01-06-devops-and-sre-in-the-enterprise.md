---
title: "DevOps and SRE in the Enterprise"
layout: post
tags:
- DevOps
- Enterprise
- Site Reliability Engineering
- Scrum
category: [DevOps, OpEd, Site Reliability Engineering]
excerpt: "At start ups we have the luxury of starting from the ground up. Philosophical and cultural revolutions are always easier to consider when simply nothing exists. This is a common complaint of large enterprises when they examine DevOps, SRE, or Production Engineering for their own organizations. Subsequently they end up morphing the core beliefs of those philosophies to fit their existing culture. While this can be done it really results in a lot of confusion and frustration especially at the ground level where, in the aforementioned disciplines, most of the work and decision making takes place. The question remains what does a properly scaled Enterprise grade version of DevOps looks like?"
---

At start ups we have the luxury of starting from the ground up. Philosophical and cultural revolutions are always easier to consider when simply nothing exists. This is a common complaint of large enterprises when they examine DevOps, SRE, or Production Engineering for their own organizations. Subsequently they end up morphing the core beliefs of those philosophies to fit their existing culture. While this can be done it really results in a lot of confusion and frustration especially at the ground level where, in the aforementioned disciplines, most of the work and decision making takes place. The question remains what does a properly scaled Enterprise grade version of DevOps looks like?

## Defining DevOps Skillsets

DevOps is a philosophy not a job title or even a skillset. That said, DevOps has some implied skillsets and some roles that go along with it. While the inventors of DevOps left this rather open ended I'm going to try to issue some standardization for you.

### Platform Engineering

Platform Engineers are mostly born systems, network, and security folks. They design the infrastructure that Software Engineers deploy their applications on. They may even design many platforms, such as one that's virtual machine based, one that's container based, and another that's based on a different type of container orchestration.

In AWS this might look like:
* EC2 based platform with Autoscaling groups and ELB's
* Kubernetes based platform
* Lambda based platform

At the end of the day their goal is to abstract away the infrastructure and enable self-service. The developers merely need to name their service (or application) and know the names of other services and applications they wish to interface with.

Common tools for Platform Engineers are secrets management, service discovery, object storage, CI/CD servers and pipelines, infrastructure orchestrators, and container orchestrators. This is by far not a comprehensive list.

The Platform Engineering teams product _is_ the platform. Platforms are versioned and released just like applications.

Although Platform Engineers come from a systems, network, and/or security background they are still considered Software Engineers. They're not incredibly in depth software engineers but they know how to use software to accomplish the task at hand efficiently and effectively.

### Automation Engineering

Automation Engineers are of the same skillset as a Platform Engineer and highly fluid organizations may choose to rotate Platform Engineers and Automation Engineers. The goal of an Automation Engineer is to embed on a development team and provide them with resources for adding or interfacing with infrastructure.

For example, the Automation Engineer may arrive on a development team who is in need of a messaging service. For whatever reason messaging was not a requirement built into the existing platform version. While an easy way out may be to use something like _AWS SQS_ it may be more worthwhile to remain multi-cloud functional, so that Automation Engineer begins building the appropriate messaging server. Once complete the Automation Engineer ensures, through integration testing, that the development teams can successfully interface with the new messaging service and that it's registered in service discovery. At that time the changes to the infrastructure are then PR'd to the main trunk of the upcoming platform version.

The sub-function of Automation Engineers is to help coach and train software engineers to work in an ever-changing environment while maintaining standards that build trust in the overall applications. This may be doing influencing on how to interface with other teams, describing production requirements (Production Deployment Checklists), and proper use of CI/CD. That Automation Engineer is _not_ there to take over these functions for a team. If you find yourself in the situation where you have one Automation Engineer per development team at all times then something is inherently wrong and the processes should be evaluated.

The process of embedding, rather than statically rotating, is key to this role. Automation and Platform Engineering should not scale horizontally with an organization. Instead, they should focus on self-service and eliminating user pain _long term_.

### Software Engineering

DevOps and SRE both enable a lot of empowering for engineers. That said, with great freedom comes great responsibility. This means that Software Engineers are now tasked with owning production issues. Ideally they can self-service alerts and logging, therefor any pain they feel is changes they're capable of making on their own.

These types of teams are fairly dynamic and can work in both Microservice Architecture as well as Monolithic Architecture. In a microservice environment a team may represent a particular service while one developer represents a microservice. In a monolithic environment you can segregate teams by functionality as you traditionally would.

### Site Reliability Engineering

Everything I've discussed prior to this excludes SRE. If you include SRE then certain things will change. SRE's own production environments, product versions that reside in production, standards for release, and tools related to assuring the production environment among other things. While I'm not going to cover everything SRE's do (that's for the SRE book), I'll cover what changes in the team dynamics though it's not a comprehensive list.

I've heard people say before that SRE simply cannot coexist with DevOps. On the contrary, DevOps is very complimentary to SRE in this organizational format. In a change, Platform Engineering would pass ownership of a platform version to an SRE-SE team for production. The SRE-SE team would see the product through to the end of it's lifecycle as well as approve of it's release. SRE-SWE's would perform embeds as necessary to help solve performance issues in applications or services and would also evaluate versions of those same applications or services for production release, this would replace your existing Automation Engineers though you could possibly keep them around in a limited capacity. It is worth mentioning that SRE typically does not support every team, so SRE does not own production for _all things_. Ultimately if the SRE team becomes overwhelmed then even supported developers must be ready to take over support for production.

If you've read anything on SRE's, their unit mantras are similar to PE and Automation Engineers. They do not scale horizontally with the organization, they should focus on reliability, performance in the _long term_ and eliminating user pain in the _short term_.

Obviously I'm not covering a lot on SRE here or their goals, as their mission statement is truly business wide. For that and more you should [read the SRE Book](https://landing.google.com/sre/book/index.html).

## Scaling Agile & Scrum

There's a lot of talk on growing Agile with models like SCALE. The problem with these types of models is the same problem I mentioned in the first paragraph, you're carrying with you remnants of the past. Instead I suggest a model that enables engineers on teams to elect team mates to represent them in high level scrums. Additionally, due to the fluidity of the teams, engineers can create work for themselves on other teams boards. This allows engineers to create PR's on other teams code and cuts down the need for large operational overhead. The more team to team interactions and decision making and less high level meetings the more fluid the development work becomes. Teams agree on standards and common interfaces while executives worry about vision for the future.

The stressor here is allowing organic growth rather than _what some guy on a blog says (stares intently at self)_. These strategies are more about staying inline with the core beliefs of Agile. For more inspiration you should [read the Agile Manifesto](http://agilemanifesto.org/) as well as [the Scrum Guide](http://www.scrumguides.org/docs/scrumguide/v2017/2017-Scrum-Guide-US.pdf#zoom=100).

## Unsolicited Advice

If you're considering a cultural change to your organization consider the following:

* Enable and empower engineers for day to day and technical decisions; avoid top-down decision making.
* Educate and enforce cultural changes among middle managers first.
* Ensure all executives and board members of your organization agree this is the way to go.
* Avoid doing small implementations over time. Stand the PE and SRE teams up and enable development teams as you go _if you must_.
* Develop quarter based KPI's for teams prior to launch.
* Don't dive straight in, educate yourself and the organization first.
* Don't focus on the tool, focus on the problem. Allow engineers to make decisions on tool sets even if sprawl occurs. Sometimes sprawl is a good thing, teams will automatically begin to consolidate in well-functioning organizations.
* Utilize truly anonymous surveys to discover and highlight pain points. Engineers should feel free to discuss what hurts them.
* Again, _educate, educate, educate_. Send your engineers to training and conferences.

That's all I've got for today. If you have any questions feel free to use my contact form and I'll definitely post follow on articles!

Note: It should be noted that while I didn't provide specific examples, this structure is largely based on Lehman's Laws of Software Evolution. If you haven't taken a look at them, be sure to!
