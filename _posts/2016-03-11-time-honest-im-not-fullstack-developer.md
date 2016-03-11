---
layout: post
title: "Time To Be Honest: I'm Not A Full-Stack Developer"
date: 2016-03-10 23:45:07 -0500
comments: true
categories:
---

I started playing around with [Phoenix](http://www.phoenixframework.org/) the other night
and really enjoyed it. It's a lot like Rails in many ways, but it's very clean and the source
is crazy readable. I'd say it's a step up in complexity from Rails since it has things
like first class websockets support.

Websockets is one of those buzzwords everyone seems to be talking about but not actually
doing anything with. I figured I'd play around and try to create a basic chat client. I started with
some primitive socket code, but I needed some markup to get started.

I started off with just some basic jQuery, HTML, and CSS. jQuery is supported out
of the box with Phoenix, so this was an easy choice. After a little hacking I came to
the conclusion that my design skills are pretty poor. Despite having used HTML and CSS
quite a bit, neither is something that I've ever felt 100% comfortable with. Everything
I write tends to end up being divs, spans, and lists, and all my CSS winds up becoming
a jumbled mess of global scope.

Given this, I turned to Bootstrap. Bootstrap is a pretty simple framework that has
some components built in. This got me going a little faster. I was able to throw
together a couple of pre-built components and styles pretty fast, and everything
was going swimmingly. Then I needed to add my own custom tweaks to Bootstrap.
Now I could just throw whatever I wanted in my `app.css` file, but that seems pretty
terrible. Really what I want is something more comprehensive than Bootstrap.

I remembered reading about Foundation recently, so I gave that a quick googling.
Foundation seems to me like a more complete alternative to Bootstrap. Moreover,
it looks like it's trying to unify all of the front-end technologies into one
comprehensive library for the rest of us. It does use Angular which I don't
particularly care for, but I can see why it's a fit for a library like this.

So I merrily downloaded Foundation and started going through the documentation
when suddenly something clicked. I hate front-end development. I hate working with
CSS and HTML and designing web pages. Programming the actual interactions in Javascript
is all well and good, the more visual and semantic parts are literally hell.

The sad thing though is that I actually enjoy working on the UI. The thing I dislike is the
ecosystem. HTML and CSS used to be simple things that anybody could understand.
Over the past few years we've added all sorts of ridiculousness on top: now you need
to understand npm, bower, SASS/SCSS/LESS, ES6, Gulp/Grunt/Brunch, all sorts of random libraries; all this on top of understanding the
core principles and best practices and adapting ourselves to updates in HTML5, CSS3,
and ES6. I mean, the Foundation docs even have you install Ruby; why in the world do you need
Ruby for a front-end framework?

In the end it really makes me question: why not just go back to vanilla JS? Is getting
a few "free" components from these frameworks really worth it if I have to wrap my head
around 6 other dependencies and slog through documentation. In the meantime I could
just be mastering the fundamentals without all the indirection of a framework.

I think the real answer for me however is that I just don't like front-end development.
It's boring and excruciatingly tedious all for something to look nice. Sure, that has
a time and place, but I'm not sure I'm the guy for it. I'll help out with the Javascript
interactions and asynchronous request lifecycles. Other than that, let me help you
scale your apps because that's what I love.
