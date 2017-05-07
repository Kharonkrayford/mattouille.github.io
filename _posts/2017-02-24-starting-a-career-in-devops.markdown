---
title: "Starting a Career in DevOps"
layout: post
date: 2017-02-24 22:27
headerImage: false
tag:
- devops
star: true
category: blog
author: mattouille
description: My opinion on the qualities and skills of a DevOps based profession
---

A few years ago I got out of the military as a radio technician, but before I got out I had a ominous conversation with a long time friend explaining that I thought the future of an IT career lied in a mix of systems, programming and virtualization. At the time I really knew nothing about virtualization, had really only web and some Perl/Python experience and a life long love of Linux. Since then, my experience has turned it into a beautiful well-rounded career that’s only growing.

*“What is DevOps?”* is probably one of the chief questions most people ask when they hear it. Some people regard DevOps as a fad rather than an actual solution, or something that programmers are already doing. That said, many people regarded Lean, Agile, and OOP as fads as they emerged and now they’re quite the staples of our industry. Explaining what exactly DevOps is isn’t really the direction of this article, but there are many resources at hand to help you find your way if you feel lost.

#### For a junior DevOps Engineer there are a few things you need to know and understand in order to be successful.

**Ruby and Python.** There are many other languages that are popular but these two are chief in automation and metrics collection. They are virtually system agnostic, although, libraries can tend to favor certain operating systems. This is why having a good foundation in reading and understanding code is imperative. Believe it or not many Linux System Engineers cannot read and understand non-procedural code or don’t understand OOP design principles.

**Linux.** In this writers humble opinion Linux is the old trusty of DevOps. Nearly every enterprise is using Linux and even Microsoft has it’s own take on the bourne shell with powershell. Linux has been the beacon of efficiency and security for decades and will only continue to grow. Familiarize yourself with the standards and practices of both GNU/Linux and Redhat variations. That is, the five most important, being Debian, Ubuntu, CentOS, Redhat (RHEL), and Fedora. It's also worth looking at distros like Alpine which are advertised as container ready and minimal.

**Virtualization.** Whether you’re in development or production a working knowledge of virtualization is paramount. Tools like Virtualbox in combination with Hashicorp’s Vagrant have revolutionized the way we test expected outcomes. Even classical developers are starting to use Vagrant, realizing the power that it holds to mimic a production environment.

**Configuration Correction & Automation.** The modern enterprise no longer desires to consist of hundreds of teams of systems administrators. Now, using code such as Ruby, Python and Perl, small teams are orchestrating wide arrays of configuration management for hundreds if not thousands of servers at a time. Just a few titles in this category are Puppet, Chef, Ansible and Salt but there are many more. This in combination with automated provisioning puts these tools above all else. I have not mentioned automated provisioning as it’s not really entirely required at a junior level for most organizations.

**DNS.** Any configuration management or automation tool such as Puppet, Chef, Ansible or Salt all require some sort of private authoritative DNS server, so it’s important to know and understand how DNS works. In order to understand DNS you will need a very basic understanding of subnetting. That said, a lot of DevOps Engineers in their junior years will also be tasked with managing day to day operations with a DNS CDN such as Cloudflare. You should have a working knowledge of how DNS is managed, the difference between A records and CNAME‘s as well as some DNS management principles.

**Monitoring.** Monitoring resources on a virtual machine is pretty important too. Lots of organizations use Open Source tools such as Bosun, Zabbix, Icinga or, the old trusty, Nagios. Some organizations prefer a simple yet powerful implementation of Ganglia and collectd or statsd. All of these are acceptable solutions and really you just need to be familiar with common terms in statistics collection and configuration to make useful outputs. Whether you’re monitoring system resources, file changes or API uptimes the concepts are virtually all the same.

**Database Management.** Commonly DevOps Engineers are asked to build queries. This isn’t so much a DBA job, as DBA’s usually manage the configuration of large scale long term or short term storage technology, it’s more coming up with stored procedures that can be useful. Most organizations will accept a working knowledge of mySQL and common commands/queries. If you already know a lot of things I’ve listed then check out MongoDB, Redis, and Hadoop – you will surely see this further down your career path.

**Logging.** Aggregating logs is important in the troubleshooting process. Now I can go to one simple place and query the logs of a particular server, group of servers, etc… Some of the tools organizations are using today are ELK Stack, Greylog and Splunk. All of these have their own advantages and disadvantages but GreyLog is by far the easiest to get up and running on RHEL, while ELK will take some specific knowledge.

**Git & SSH.** Git is a code repository software used to manage changes, push to production and other useful things. Many coders have turned their entire website into Git Repositories. With this comes a working knowledge of SSH. That’s any implementation of ssh such as SCP, tunneling, key pairs and more. Check out a fun video of what Bo Jeanes did with SSH!

### Other things you can expect out of a DevOps Position

* Be ready to be challenged every day
* Know the basics of Agile
* Once you’re familiar with all of the above, think about how you can implement DevOps philosophies
* Be ready to code!

### Resources

* UDemy (Same as Code Academy but with academic courses)
* Code Academy (Git, Python, Ruby, Shell) It’s free!
* DevOps Bookmarks
* The Agile Admin blog
* Reddit DevOps
* Freenode IRC ##devops

DevOps is a beautiful career path with lots of daily challenges and problem solving. DevOps Engineers usually rest in a space between Developers and Systems Administrators with a special adeptness for details. It’s a fun and exciting career field and I look forward to everyone making their way there.
