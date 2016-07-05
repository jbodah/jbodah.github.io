---
layout: post
title: 'Relearning C'
date: 2016-04-19 08:44:28 -0400
comments: true
categories:
---

C was the first programming language that I learned (unless you consider adding gifs to my geocities page programming).
I was a bit of a "late bloomer" with programming and didn't start learning it until junior year of college.
Once I did though, I was hooked, and as things go I have a soft spot for C.

Unfortunately I haven't had the chance to work on C in a production code base yet. Currently most of my work is done in
Ruby. I've always felt that Ruby and C are very similar in many ways. On paper they're very different: static vs. dynamic
typing, compiled vs. interpreted, procedural vs. OOP. However many C conventions feel like they transfer over to Ruby,
and I feel like once you start doing any systems programming in Ruby that you quickly see that the two aren't terribly different
syntactically.

I picked up K&R a few months ago, a book I've always meant to read. I just picked it up and started working through it and wanted
to give my thoughts on coming back to C about five years later. Let me also give a quick plug that I think the K&R book is excellent.
It's terse and to the point while still having lots of detail. I'm always looking for language books aimed at professional programmers,
and this is one of them.

## What I Like

* C has lots of great things about it. First and foremost, the language is very small. This makes it extremely easy to get up and going.
I was a bit surprised how quickly I went through all of the syntax and built-in functions.

* I miss static types. There, I said it. I usually find types very limiting (see Java and C#), but I feel like they're done right in C.
I don't have to look up what this ridiculous interface is and then have three long winded discussions with co-workers about why we have
three ever-so-slightly-different interfaces for doing the same thing. To me C is more like functional languages in this regard (note: I don't
find functional and procedural programming all that different, but that's a talk for another time).

* Function prototypes. Oh man, do I miss these. I love the fact that C requires you to include your headers up-front making it fairly clear
what your dependencies are and where they're coming from. Big plus for me coming from "where the hell was this defined" Ruby.

* Character constants. It's a little strange that Ruby doesn't have this. There is a way to convert strings to characters, but it's wonky.

## What I Dislike

* In general I find idiomatic C a bit of a pain. The various short-hands like skipping brace for one-line loops are something I find
a bit annoying. It's like the language is just begging for more expressive syntax.

* Parentheses. You really just don't need them. I'd rather see a cleaner syntax that is more restrictive for the benefit of readability
and agility. I may write my own transpiler to enable writing C more cleanly.

* For loops with multiple conditions. `break` was made for a reason. I feel like having more than a few break conditions in your `for` expression
is just an abuse.

* Prefix ***and*** postfix increment/decrement is overkill. There is no reason why a language should need both. Having both just adds to the cognitive overload
of the language.

* Lack of standard data structures. Sometimes I just want a linked list or a hash table. I don't want to implement my own. I want the standard library to provide
these things for me, and I want to be able to implement my own when things are mission-critical. I really wish generic lists, hashes, queues, stacks, trees,
graphs all just came for free with the language.

* No regular expressions. Muy sad face. Now I understand how Perl got so much traction. C kind of sucks for pretty much all text processing in my opinion,
and it's difficult for me to really think of C as a modern, general purpose language suitable for things like modern web MVC development.
