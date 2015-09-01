---
layout: post
title: "Debugging Vim Performance"
date: 2015-09-01 15:55:42 -0400
comments: true
categories:
---

Recently I've been noticing Vim slow down every now and then.
At first I thought it was just that my dev environment had too much running, but I was watching `htop` today and noticed that Vim's CPU usage would
spike to 100% when I would scroll.

My first thought was that it was `CtrlP` since I've noticed that slow down in several other cases. I tried commenting that out, restarting Vim, and doing
`:PluginClean`. Nope, not the issue. I did the same for the rest of my plugins and still no dice.

As a sanity check I opened Vim without my `.vimrc` with `vim -u NONE`. This starts Vim as it would out of the box. I popped open a large file, scrolled,
and saw no performance issues. This meant that the issue had to lie somewhere in my custom `.vimrc` commands.

From there on it was just binary search. My `.vimrc` is reasonably well organzied into three to four main section, so I just took turns commenting each section
until I found the one that had the issue. I narrowed it down to `set cursorline`. I'm guessing that Vim is doing a ton of redraws when I scroll. To make matters
worse, I dropped the repeat rate of my keys significantly, so it reallys kills the CPU if I hold down `j` or `k`.

A little research shows it's a known problem which is unfortunate, but at least we're back to being nice and fast!
