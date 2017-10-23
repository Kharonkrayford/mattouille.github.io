---
title: "Variations in High Availability"
layout: post
tags:
- Programming
- Infrastructure
- Site Reliability Engineering
category: [Programming, Infrastructure, Site Reliability Engineering]
excerpt: "High availability is a term thrown around quite a bit these days. Many professionals conflate high availability with the idea of a theoretical 100% availability. I'll be the barer of bad news in saying such a thing rarely exists, is even harder/more expensive to obtain, and often not worth it. Rather, I encourage teams to identify what high availability means to them. This article will be an exercise in exactly that."
image:
  feature: posts/2017-10-23-variations-in-high-availability/high-availability-poll.png
  credit: PerconaDB
  creditlink: https://www.percona.com/blog/2016/05/23/take-perconas-one-click-high-availability-poll/
---

High availability is a term thrown around quite a bit these days. Many professionals conflate high availability with the idea of a theoretical 100% availability. I'll be the barer of bad news in saying such a thing rarely exists, is even harder/more expensive to obtain, and often not worth it. Rather, I encourage teams to identify what high availability means to them. This article will be an exercise in exactly that.

**Note**: _Please keep in mind definitions of high availability change service to service and use case to use case._
{: .notice--info }

# How do I define high availability?

Generally speaking high availability means that a service has been built (whether developed in software or built with infrastructure) that implies the following:

1. A high level of testing
2. Redundancy features which reduce downtime
3. Accommodations to increase stability

High availability can be achieved through software, infrastructure, or a combination of the two.

# Measuring availability

In SRE we often spend a considerable amount of time defining and refining metrics on a particular service. Google separates these activities into _white box monitoring_ and _black box monitoring_.

White box monitoring collects metrics on an atomic level. You are often left directly querying the application for information or querying sources of information that the application directly utilizes. For example, if I was tasked with developing metrics for a web server such as Nginx I might query log output from Nginx to calculate _requests per second_, establish a stub service consisting of Nginx statistics, and I might also collect statistics directly related to Nginx's resource utilization (such as CPU jiffies consumed).

These types of metrics don't always indicate whether a service is actually up but more foreshadow whether a service _will_ experience problems.

Black box monitoring on the other hand generally reflects an applications availability through the eyes of the user. One form of black box monitoring is to literally consume the API endpoints of a service. In this way I can look at the HTTP response code, I can examine data, and I can even use the very tests I use in my CI/CD pipeline to assure that the service is continuing to operate as expected.

These metrics are very specific to describing _user pain_ while the former more accurately indicate when a service is approaching a threshold that causes user pain. This is the difference between _proactive_ and _reactive_ monitoring.

Determining availability is really the job of both of these practices, you really cannot have one without the other. Availability is a broader question than just "is it up or not" though. A colleague of mine, Daniel Torres developed a fundamental set of metrics which he describes as "VALET". VALET stands for Volume, Availability, Latency, Errors, and Tickets. The term "high availability" is really a combination of the businesses tolerance for the above. The business will establish an SLO for each portion of VALET and what "highly available" means in each category.

Notice I brought SLO's into the picture, high availability really cannot be established without a baseline and a goal. "High availability" isn't any one architectural model but more the practice of defining baselines and what the minimal acceptable benchmark for availability is. In VALET "availability" is a category but the term "high availability" is really a description of all the categories.

# Dissecting VALET

We'll use our example web service, Nginx. In this case we're going to assume Nginx actually is a service and is built in a way that it can accept whatever endpoints may occur in something such as [Consul Service Discovery](https://consul.io/). If you're not aware, this is a pretty awesome pattern for microservices if you do it right. Nginx is a beast.

**Volume**. You have to look at what the primary product of the given service is. In this case Nginx's product is handling requests. We want to shrink that metric down to the [smallest observable number](https://plato.stanford.edu/entries/measurement-science/#StaSciPro). This makes working with and understanding that number substantially easier. In our case we'll use the ratio of _requests per second_. It's important that this number reflect _all_ requests, even if they're frivolous or overhead. If you remove metrics from the equation then your predictions will become less certain.

**Availability**. This definition will generally grow over time and with your service. Firstly I might start with the number of 200's because I know that if something is _not found_ it will result in a 400 and if something _times out_ it will result in a 500. Again noting, this definition of availability is highly immature. Availability will be a percentage of total requests that return a 2xx response code.

**Latency**. Nginx spits out some numbers that are quite useful, but for other services you might have to calculate latency in a different way. For instance, Nginx tells us, for free, how long the request took from the user to the server, it tells us processing time with a backend service, and the time it took for when it received a response from a backend to deliver it to the user. In this instance I really want to narrow my metrics to "when Nginx was handling the request, how long did it take?" Thus this policy can trickle down to monitoring backends as well.

**Errors**. I generally keep this number a count but it could be a percentage. Sometimes knowing the error _count_ can be useful when plotting _errors over time_ whereas to a business person they may be thinking in terms of how error prone is this service. Really it comes down to who your consumer is for a given metric and how to properly represent an _error budget_. In terms of Nginx it's the total number of 4xx's and 5xx's returned as compared to the total number of requests. You might be saying, but 4xx's could be the result of an application. Yes, and this is where you have to mature your metrics to represent an accurate SLO.

**Tickets**. If you're using something like [PagerDuty](https://pagerduty.com) or [JIRA](https://www.atlassian.com/software/jira) and creating tickets for automatically identified issues then that count goes here. The point is to determine the strain this service puts on a team supporting it - Google defines this as toil. This can be a great indicator for development prioritization.

# Achieving high availability

As I said before, high availability can be achieved through software, infrastructure, or a combination of the two. In another restatement, attaining a higher level of availability can be especially challenging and burdensome for the engineers charged with building and maintaining it. The operational cost should be weighed with the organizational benefit.

I'll supply a real world example:

I was refactoring the code which spun up and maintained our bastion hosts at work. They were originally coded in a way that made them organizationally complex with a twenty-plus step process for provisioning and a fourteen step process for ensuring uptime. SSH hosts cannot be placed behind load balancers as load balancers frequently do not work well with streamed data (such as an SSH tunnel that needs to remain open.) This makes this problem quite challenging from a traditional point of view.

I would be able to code a Lambda on AWS that managed bastion hosts in a round robin, removing bad hosts as they were discovered. During a benchmark I found that an Auto Scaling Group takes about 1-2 minutes to identify an unhealthy host. This means that users would inevitably receive intermittently poor service for 1-2 minutes at the expense of a considerable amount of work. For some context, Route 53 is not the most friendly service to work with and I would need to be pulling user data from the bastion host in order to perform operations on it.

That said I made a decision to live with 1-2 minutes of downtime and instead spend my time building a machine image that would come up quickly and an effective health check to discover problems more quickly. Additionally I noted that the automation currently applied to our bastion hosts was actually ineffective and hadn't been working for some time yet no one had been complaining about a lack of availability. Upon diving into the metrics I found that the bastion host rarely ever went down and when it did it usually correlated to events in which availability zones had become unstable. In this regard it was better to spend my time producing a simple, secure, and yet effective bastion solution rather than concentrating on how many nines I could tack on to 99.9%.

# Conclusion

It isn't always impressive numbers that matter, it's at what cost and benefit to the business that a certain level of availability may imply. Be wise to evaluate and weigh each of these and be ready to rework your strategy occasionally as organizational requirements may change.

Consider this: Google once had a service that by its very nature was "highly available". This wasn't on purpose, just by accident the service was very stable. A surprise outage in the service revealed that other services had created hard dependencies which created a larger outage. From the business' point of view this service was not one which deserved highly available support and a decision was made to intentionally take the service down for "maintenance periods" to ensure a lower availability as a means to removing hard external dependencies from their other services.
