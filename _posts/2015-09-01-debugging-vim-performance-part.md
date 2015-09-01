---
layout: post
title: "Debugging Vim Performance (Part 2)"
date: 2015-09-01 16:55:20 -0400
comments: true
categories:
---

In my last post I talked about how you could use binary search to narrow down components in your `.vimrc` that are bogging you down.
I just ran into a case where `syntax on` was the bottleneck.
This is basically constantly running regex matchers on your text.
If a match is found then the text is tagged with a group.
That group can then be targetted by things like your color scheme.
This provides a nice separation of concerns.

Given that, I obviously didn't want to just disable the syntax feature.
A quick stroll down Google lane brought me to [this great question](http://stackoverflow.com/questions/19030290/syntax-highlighting-causes-terrible-lag-in-vim).
One of the answers mentions using `:syntime on`, typing your slow bits, then `:syntime report`.
This will record the time it takes to run each of your syntax commands and reports the slowest one.

```
  TOTAL      COUNT  MATCH   SLOWEST     AVERAGE   NAME               PATTERN
  0.793146   560    0       0.008129    0.001416  rubyPredefinedConstant \%(\%(\.\@<!\.\)\@<!\|::\)\_s*\zs\%(STDERR\|STDIN\|STDOUT\|TOPLEVEL_BINDING\|TRUE\)\>\%(\s*(\)\@!
  0.667850   560    0       0.006749    0.001193  rubyPredefinedConstant \%(\%(\.\@<!\.\)\@<!\|::\)\_s*\zs\%(MatchingData\|ARGF\|ARGV\|ENV\)\>\%(\s*(\)\@!
  0.502142   560    0       0.005166    0.000897  rubyPredefinedConstant \%(\%(\.\@<!\.\)\@<!\|::\)\_s*\zs\%(DATA\|FALSE\|NIL\)\>\%(\s*(\)\@!
  0.321655   560    0       0.003583    0.000574  rubyPredefinedConstant \%(\%(\.\@<!\.\)\@<!\|::\)\_s*\zs\%(RUBY_\%(VERSION\|RELEASE_DATE\|PLATFORM\|PATCHLEVEL\|REVISION\|DESCRI
PTI
  0.075974   560    0       0.000646    0.000136  rubySymbol         []})\"':]\@<!\%(\h\|[^\x00-\x7F]\)\%(\w\|[^\x00-\x7F]\)*[!?]\=:\s\@=
  0.034503   560    0       0.000256    0.000062  rubySymbol         \%([{(,]\_s*\)\@<=\l\w*[!?]\=::\@!
  0.009783   740    180     0.000056    0.000013  rubyConstant       \%(\%([.@$]\@<!\.\)\@<!\<\|::\)\_s*\zs\u\w*\%(\>\|::\)\@=\%(\s*(\)\@!
  0.007708   560    0       0.000040    0.000014  rubyKeywordAsMethod \%(\%(\.\@<!\.\)\|::\)\_s*\%(public\|require\|require_relative\|raise\|throw\|trap\)\>
```

Low and behold `rubyPredefinedConstant` is also the problematic matcher in my case. We can disable it in `/share/vim` directory or, if you're using Vim 7.4+, you
can just enable to old regex engine `set re=1`. Alternatively, `vim-ruby` is also said to have syntax that works better.
