---
layout: post
title: "Ember: First Impressions"
date: 2016-03-17 22:51:24 -0400
comments: true
categories:
---

I've always heard that Ember was a great fit for Rails devs, and tonight I took some
time to finally play with it. So far my "JS life" has gone from working with vanilla
JS to straight jQuery to Backbone and now most recently some Angular.

My personal preference has always been towards Backbone as opposed to Angular. Backbone takes
more of an MVC approach with a major focus being on how the data on the client
syncs with the data on the server. Angular, on the other hand, takes more of an MVVM
approach that I find very similar to .NET. I think this explains why so much of the
.NET mindshare has gravitated towards Angular.

So my personal preference is in Backbone (which is also in part because it doesn't perform [wild magics](https://www.ng-book.com/p/The-Digest-Loop-and-apply/)
on my code and I can read the [entire source code](https://github.com/jashkenas/backbone/blob/master/backbone.js) in an hour or two). That being said,
I don't think Backbone is full-featured enough for a fresh project and particularly
not for a single-page app. Backbone is really more of a library on top of jQuery for data syncing
than a framework. You'll have lots of bootstrapping to do. Honestly, I enjoy that kind of work,
and I tend to think I'm pretty good at coming up with custom framework-y code, but that's
still no reason to choose it over something which is much more established like Angular.

Angular is a great choice for a single-page app in particular because it uses MVVM.
In Angular your views are blessed with rich functionality. Additionally, the type of organization
and structure that Angular provides is perfect for a SPA where you want the real
focus to be on user interaction with widgets and the like.

Backbone is a better choice for something like an activity feed; you may never interact
with the feed, so you don't need the heavy SPA-first feature set of Angular.
But if you want something like a rich dashboard with filtering and graphs, then Angular
is a prime choice.

But this post isn't really about either of those :) Today we want to talk about [Ember](http://emberjs.com/).
So far I've just dabbled with `ember-cli` a bit and gone through the tutorials.
Right off the bat I have to say that the documentation is incredibly good.
I was able to get up and be productive in literally seconds.

Right now Ember feels really similar to Backbone in it's construction; it uses
prototypal inheritance heavily and renders views with handlebars (I guess Backbone
technically uses underscore, but I've always used handlebars). There is more structure
out-of-the-box than with Backbone and, while I hate the Rails and Phoenix generators,
I really like the Ember generator.

Some other great things include first-class component abstractions,
a built-in build pipeline (which clearly resembles [rails/sprockets](https://github.com/rails/sprockets)), and
a web server to boot. Actually, I'm not sure how I feel about having Ember provide me with a server, but
we'll reserve judgement for now.

A last thing I'll mention is that `ember new` seems to generate very little crappy
bootstrapping code (yay!). Off the bat you just have an unstyled `Welcome to Ember`
message which means you don't have to dig through a dozen files removing boiler plate.

So far I'm a big fan of Ember. It looks like it's a happy medium between the Backbone
and Angular. I get the not-too-far-from-vanilla-JS from Backbone without the wtf-this-magic-is-worse-than-rails
from Angular. I also get great application structure like in Angular.

In the end though, the thing I ***really*** like about Ember is the fact that it is
written by a developer who really understands server-side frameworks. It's clear that
Yehuda took a lot of what makes Rails great and boiled it into Ember (thankfully in a much
simpler way in many places). The `ember-cli` is pretty legit. I wish I could say the
same for something like Angular (yes, I realize Angular is not trying to give me a
build system and prescribe a testing framework, etc.)

While I don't think we'll be seeing Ember much at my current job (we're pretty in
love with Angular I guess), I see huge potential for Ember in my near future.
I still need to see how it handles for SPA-style applications, and I'll need to play
with it to see how it stacks up with my favorite web frameworks. For now though,
I'm pretty confident that I'll be pushing hard to use it the next time we're looking
for a great front-end framework.
