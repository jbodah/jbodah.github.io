---
layout: post
title: "Optimize Code for the Reader"
date: 2014-05-31
comments: true
categories:
---

I'm currently going through the Exercism exercises in Coffeescript ([exercism](http://exercism.io) and [my solutions](https://github.com/jbodah/exercism-solutions)). If you're not familiar with Exercism, it's basically a collection of exercises written in 15 or so languages in which users can submit their code then get feedback from other Exercism users on how their code is. If there is one thing this process is telling me, it's that people write code in vastly different ways. Some of the submission were a complete mess with external libraries being pulled in for things like regex while others were extremely well manicured to the point where it didn't look anything like your average Javascript. Others had gone out of their way to optimize their code by caching every query. Some people instead chose to give up some performance, favoring the route of clarity and conciseness. The point is that there's a million ways to skin a cat (well, so they say at least. I'd probably just go with the knife).

I remember when I first started grad school I had an image processing class. Students would get up in front of the class and have to explain their code. We were doing Fourier transforms, and my code was a big mess of for-loops which -- at the time -- I was actually pretty proud of, yet I distinctly remember when an older, more experienced classmate showed their code. The call sites were extremely clean and easy to understand. The algorithms were neatly encapsulated in their own functions that were responsible for nothing more than running the algorithm. The rest of the code was just clean business logic.

What I'm trying to get at is that **code should be written for the human. It should be readable and concise.** This means small, encapsulated methods. It means using ternary operators instead of if-else blocks. It means using short but full words for variable names. Code that is written in this manner is easier to understand, and even if it might be a little bit more difficult to understand (say in the case of a ternary vs. an if-else) having short, concise logic makes the reader more confident in their ability to grok what's going on.

This isn't to say that you shouldn't optimize. You most certainly should. But take Knuth's word:

> Premature optimization is the root of all evil.

The idea is that optimized code is generally more difficult to maintain and understand. Small performance gains are generally negligible, and you will almost always be better off optimizing you code for the reader. The correct way to optimize an application is by measurement and profiling. Build it, measure it, guess where the bottleneck is, make a change, go back to measuring. Do that until you're happy. This is what we scientists and engineers are trained to do, yet so often we trust our gut. Our gut wants us to make the best technical decisions and write the fastest code. The reality of the matter is that this mindset usually will bite us by making our code more costly to maintain and more prone to being misunderstood. A fellow programmer might look at our code and decide that it looks too much like debt and just work around it rather than try to understand it, and that's when things get bad.

The most important thing we can do as programmers is to write beautiful code. The meaning of that statement is dependent on context sometimes, but the focus on the developer's ability to understand what you've done should always be the same.
