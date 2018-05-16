---
title: "User error caused a massive S3 blackout"
layout: post
tag:
- AWS
- S3
- Post Mortem
category: [Opinions]
comments: true
---

> At 9:37AM PST, an authorized S3 team member using an established playbook executed a command which was intended to remove a small number of servers for one of the S3 subsystems that is used by the S3 billing process. Unfortunately, one of the inputs to the command was entered incorrectly and a larger set of servers was removed than intended. The servers that were inadvertently removed supported two other S3 subsystems.
source: https://aws.amazon.com/message/41926/

Is this where we echo one of the great pillars of Linux? **With great power comes great responsibility.**

<!--more-->

Amazon actually has DevOps Engineering departments that build and manage their internal tooling. It's very interesting that, even with a DevOps department, one of the foremost constructs of DevOps was not followed:

> The emphasis is on the performance of the entire system, as opposed to the performance of a specific or single department or individual. Focus is also on all business value streams that are enabled by IT. **A key aspect of this approach is ensuring that known defects are never passed along, e.g., from QA to operations.**
Source: http://itrevolution.com/the-three-ways-principles-underpinning-devops/

---

What I'm pointing at is that there was nothing built into the tooling to indicate to the user that this action they had performed was going to yield the desired result. It was known that this was a dangerous and presumptive tool but it took nearly 24 hours of downtime to realize the tool needed to be changed.

> We are making several changes as a result of this operational event. While removal of capacity is a key operational practice, in this instance, the tool used allowed too much capacity to be removed too quickly. We have modified this tool to remove capacity more slowly and added safeguards to prevent capacity from being removed when it will take any subsystem below its minimum required capacity level. This will prevent an incorrect input from triggering a similar event in the future. We are also auditing our other operational tools to ensure we have similar safety checks. We will also make changes to improve the recovery time of key S3 subsystems
Source: https://aws.amazon.com/message/41926/

I think we can all learn a valuable lesson in building internal tooling. Though the tools are built for internal use only, you should never assume the user is any less capable of error.

**Source**: https://aws.amazon.com/message/41926/
