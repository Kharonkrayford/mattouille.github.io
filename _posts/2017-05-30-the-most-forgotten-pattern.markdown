---
title: “The Most Forgotten Pattern”
layout: post
date: 2017-05-30 19:30
headerImage: false
tag:
- design patterns
- iterative development
star: true
category: blog
author: mattouille
description: Developers love consistent ways of solving repeating problems, but the most consistent problem of them all is never solved repeatedly.
---

Developers love consistent ways of solving repeating problems, but the most consistent problem of them all is never solved repeatedly. Every time we sit down to solve problems with software we apply design patterns to overcome situations that would be contentious if we hadn’t already solved them years ago. There’s no point in reinventing the wheel, right? So why are we sitting here planning development of software for months on end still?

### Some men just like to watch the world burn

Iterative development isn’t really a *design pattern* but it is a pattern nonetheless. It’s a pattern used to overcome the hang in project velocity. Nothing sucks more than getting excited about your next project coming up other than trying to suck your own soul out with a vacuum while you plan every little piece of it three months in advance of even writing code.

There’s a myriad of excuses for this behavior. Let’s review and refute a few:

1. I don’t want to waste my time developing something that will inevitably fail.
2. I want to make sure all the right resources are allocated to this *before we begin*.
3. I want to make sure this project stays within cost.

---

1. All software is retired eventually folks. Even better, software is also continuously refactored, which sometimes results in an almost unrecognizable codebase. Can you possibly tell me that you’re going to find the crystal ball in some docs somewhere that say, “This is not possible” and that you will just accept it?
2. It is literally impossible to plan more than two weeks in advance. I will bake fifty cookies for the man or woman that can show me they can precisely plan (yes, it should be precise for the amount of time forward planning takes) two weeks in advance. Have you ever even taken a vacation that went *exactly* as planned **and** stayed on budget that you planned more than two weeks in advance?
3. Cost is a funny thing. Cost is reflective of a well practiced team. Nothing you do the first time will come cheap. You’re still figuring out the in’s and out’s. You’re still afraid to shut things down at night. Hell, you’re probably not even 10% the way through **understanding** the documentation through the first iteration. Consider this: A well organized SRE team might be able to iteratively build on AWS or GCP very cheaply. Why? Because they automate their deployments, know pricing models, and since everything is ephemeral they aren’t afraid to tear down at night.

### One potato, two potato

Iterative Development is simple. **First**, quickly build a completely crap, fundamentally working prototype. Get the major features working, maybe it’s an API, or functions hooked up to a CLI - who cares. It may be totally unusable by the average person, but as long as someone can use it that’s what matters. **Next**, recognize what you learned and what direction you can go from there. Steadily build on features but take time to revise your existing functionality. Rinse and repeat.

**The key here is discovery.** The team members involved are able to get very intimate with how the application should be developed. They can actively discover and fix pain points, or even prioritize a rewrite of certain aspects of the codebase while maintaining a forward velocity of **major features**. This way there is no individual sprint dedicated to maintenance and refactoring, because it’s done on the fly, as needed, with the participation of the rest of the team.

The day development becomes a chore, innovation and creativity die.
