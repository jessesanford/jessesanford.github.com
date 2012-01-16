--- 
wordpress_id: 75
layout: post
title: Successful SAAS/PAAS. How to architect maintainable services without killing yourself.
date: 2011-03-09 10:10:40 -05:00
wordpress_url: http://www.jessesanford.com/?p=75
---
Even the most simple SAAS/PAAS ain't easy. As soon as you have a single client you are on the hook to make sure your software works as expected 24/7 or suffer the tongue lashing. I recently launched a SAAS to support a multi-player game title on top of ~30 bare-metal servers. I have had a lot of experience building large scaleable web-apps, but this was the first time I had to support other developers directly. This series of blog posts will cover how I was able to architect a hardware/software stack that could be developed by a team of two, iterated on quickly and then maintained with minimal downtime. For all of you nerds out there...I will try to get into the nitty gritty technical details. More importantly though I hope to also focus on the high level principles which helped me make the right decisions in the right places and ultimately led to a calm, cool and collected launch. Here is a top level view of what I believe to be the most important parts of the software lifecycle of a modern platform/service.

1) Testing. (Unit, Functional, Regression and ESPECIALLY LOAD testing)
2) Tooling. (Language, Framework, Libraries, IDE, Best practices/Recipes, Documentation)
3) Architecture/Platform. (Decoupled, PROVEN logical architecture on top of redundant, highly available hardware)
4) Logging. (Unified logging across all of your app servers with tunable logging levels vs shared nothing logging with distributed shell access our best friend GREP)
5) Ops Team/Hosting SLA. (24/7 support for hardware/Low level software related issues, Backups, Disaster Recovery)
6) Metrics. (Realtime, Historical, Query-able metrics)
7) Client facing support staff.

Other things worth mentioning:
Version control (Branching, Merging, developing features and maintaining releases in parallel), Behavior driven development (BDD), Agile/Iterative software development, Deployment management, Systems configuration management, Change control, Software/Service selection. The list goes on...

Looking forward to brain dumping all of this. I hope this helps make your project run easier.
