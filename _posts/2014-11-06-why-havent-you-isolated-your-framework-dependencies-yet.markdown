---
layout: post
title: "Why Haven't You Isolated Your Framework Dependencies Yet?"
date: 2014-11-06
comments: true
categories:
---

Seriously. This has been an issue In every production project I've worked on. The story goes something like this:

> Company ABC creates an app using language XYZ. After some research they decide to start developing using framework FGH as it sounds like the latest and greatest. And it is, for a couple years at least. Now we see all the problems with the framework though whether it be that the framework has performance issues for our problems or that we want to switch to something else or that there's a new major revision that we want to upgrade to. The problem is we have a million lines of code that are directly dependent on the framework and doing any of these changes is a massive undertaking!


If you've never seen the above situation then you've either been extremely fortunate or probably just haven't had enough experience working with companies that have been around awhile.

Fortunately the solution to all this is pretty easy, but refactoring to it later on can be a death sentence. In aggregate, engineers tend to deal with short term pain instead of fix it. The immediate cost is usually too great to justify a refactoring. But when you let that build you get a monster. So if you've waited until the last minute then I'm not sure what I can say at this point. It will be painful.

The solution is to build internal framework abstractions rather than depending on a framework directly. This way you can freeze the API that your application relies on and isolate all of the logic for connecting the external framework to your application to one place. Upgrading a few classes is usually much easier than upgrading your whole application, especially if you have tests around it.
