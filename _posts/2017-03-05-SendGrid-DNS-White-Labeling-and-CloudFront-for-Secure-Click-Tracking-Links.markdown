---
title: "SendGrid DNS White Labeling and CloudFront for Secure Click Tracking Links"
layout: post
tags:
- SendGrid
- AWS
- CloudFront
star: true
category: [Infrastructure]
excerpt: "At StarLeaf we had a need to secure our SendGrid click tracking links, unfortunately our provider, SendGrid, had no way of sending HTTPS traffic with their in place white labeling solution. This is how we solved that."
---

At StarLeaf we had a need to secure our SendGrid click tracking links, unfortunately our provider, SendGrid, had no way of sending HTTPS traffic with their in place white labeling solution. This is how we solved that.

*This solution does not solely apply to SendGrid as it's a generic way to proxy HTTP URL's to HTTPS. You can apply the solution I build here to nearly anything that requires proxying. CDN's are a wonderful thing :)*

### Requirements
* An SSL Certficate on AWS

SendGrid really only provides an out of the box solution for working with Cloudflare and like many other enterprises we use Dyn / Route 53 for DNS and CloudFront for CDN services. We obviously like our fine grained control. We had some motivation to change things because one of our customers apparently cannot follow any links that are plain text (HTTP). Our original setup with SendGrid was using their legacy whitelabeling system which involved a CNAME to refer to SendGrid.net.

### Our Original SendGrid Diagram
<img src="{{ site.url }}/img/posts/old_way.png" alt="" width="621" height="81" class="aligncenter size-full wp-image-49" />

Simple right? Simple isn't always best but it does get the job done fast. SendGrid does not have the ability to handle our SSL Certificate, therefore, we had to establish our own intermediary.

When we looked at enabling HTTPS on the click tracking links we found out that SendGrid recommends a proxy method via CDN but only officially supports Cloudflare and another provider. CloudFlare lacks some GeoDNS and round robin functionality which can be quite nice in enterprise so obviously this solution was not for us.

Another solution we looked at was possibly including Cloudflare at the bottom of our resolution list and only letting it handle those records for click tracking. To us this was kind of messy and now involved a third DNS provider and eliminates the possibility of failover with another provider. We also got another challenge from management which was that this should be an exercise in maintaining legacy support. We had sent out thousands of emails with HTTP links so we should continue to support HTTP and just proxy the link to HTTPS.

My team mate and I came up with a solution to convert the DNS white label on SendGrid over to their new CNAME based system.

SendGrid provided us with the destination address and the destination for the CNAME records. Seemingly, they didn't care about what happens in the middle so long as the request is finally received via HTTPS with a trusted certificate. We built out the following plan:

<img src="{{ site.url }}/img/posts/new.png" alt="" width="707" height="171" class="aligncenter size-full wp-image-60" />

We take on a bit of cost via CloudFront, but we gain end to end encryption and we're retaining any old plain text traffic at the same time. The other challenge is that these URL's are production and SendGrid has a window of time that they implement your SSL links in. This can make for a theoretical downtime of sorts. A lot of this hinges on the fact that we use DynDNS whose changes propagate all around the world in minutes.

1. Setup a new SendGrid Domain and start using it (notify.whatever.com) (*This is your sending SMTP server and only matters if you're migrating from their old CNAME method*)
2. Create Replica Amazon CloudFront Distribution from existing Beta Cloud Distribution. This may need up to one hour to propagate so complete it one hour prior to planned changeover. (*another_hash.cloudfront.net*)

    * Do not enable HTTP to HTTPS proxying (server side) at first, enable HTTP to HTTPS redirection (client side)

3. Setup verification and point to link.whatever.com to point at CloudFront (*Old links will still work*)

    * This will initiate a couple minutes of downtime.

4. Have SendGrid do the change over to SSL.
5. Enable proxying of requests in CloudFront.

    * This will likely take an additional hour

6. Once SendGrid signals that they have committed the changeover to SSL we can now turn on proxying via CloudFront.

Remember that any changes you make to CloudFront take time to take effect, so change them once and change them right.

### SendGrid

I'm going to make the assumption that the SendGrid portion of this is quite self explanatory. Their CNAME records will start working immediately once you implement them so you should not incur more than a minute of downtime during this process. This is also a good point to stop and test to make sure everything is going smoothly.

### CloudFront

Here's your CloudFront settings.

* You want a Web Distribution.
* Origin Domain Name: In the case of the diagram this is link.whatever.com
* Path = None (This would be if we were assuming any URI, but we just want to mirror whatever the existing GET parameters are)
* Viewer Protocol Policy: HTTP & HTTPS (These are the links we allow incoming)
* Allowed HTTP Methods: We're only concerned about **GET & HEAD**
* Query String Forwarding: Forward All, cache based on whitelist (We're never caching these)
* SSL Certificate: Custom SSL Certificate (You should've setup a SSL Certificate at the beginning of this tutorial. This is where you use it)

This'll take a bit to propagate but everything will work from here on out. Once SendGrid gives you the thumbs up, turn on HTTPS redirection for the Viewer Policy.

I hope this helps anyone that's been looking for a viable solution to implement SendGrid whitelisting with AWS!
