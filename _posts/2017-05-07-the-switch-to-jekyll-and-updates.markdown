---
title: "The switch to Jekyll and updates"
layout: post
tag:
- Updates
- Jekyll
category: [Blog]
---

2017 has been really good to me and I've had the opportunity to grow a lot both
as as person and in my career. I'm currently writing this post sitting on my
back porch watching my dogs play.

That said, earlier versions of my website were always based on Wordpress. While
Wordpress is a great functional FOSS blogging platform, it was really a bit
bloated for my needs. As a developer, and someone who is trying to make more
efficient use of their time, I was looking for something that got down to the
nuts and bolts of what I wanted without sacrificing too much functionality.

In that I found Jekyll and GitHub pages. I kind of wrote this off as a poor
man's solution to a common problem, however, I've discovered the elegance in
what Jekyll provides. Jekyll is categorized as a Static Site Generator written in
Ruby and extended through plugins and API driven services such as Disqus. You
can checkout Jekyll [here](https://jekyllrb.com/).

I use a Mac and haven't done a lot of Ruby focused projects so I was really
unaware of how to start. What I've figured out though is that between RVM and
HomeBrew you can get a sustainable environment going. I could probably also have
written a Dockerfile to do a lot of the dependency management for me, but I'm
only using one gem and that's Jekyll itself.

As for deployment, I already pay for GitHub. I was a little disappointed to
discover GitHub pages doesn't support a CDN proxy to run SSL on custom domains
but I looped in CloudFlare for that. All in all, a blog post is just a commit
away and I can use Atom as a post editor.

While I've been writing a lot of specific 'getting started' tutorials lately
I think it's time that I start writing more Site Reliaibility Engineering focused
work. I currently work in an AWS environment but dumping my virtual machine
provider frees me up to do some AWS and GCP tutorials. Who knows, maybe I'll get
froggy and do some Azure too.
