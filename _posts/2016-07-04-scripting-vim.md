---
layout: post
title: 'Scripting With Vim'
date: 2016-07-04 09:01:20 -0400
comments: true
categories:
---

vim is a tool that continues to amaze more the more I use it. It's highly extensible
and more importantly, it's extremely easy to chain primitives together to create very
powerful editing capabilites. In my eyes, the vim philosophies are "focus on primitives"
and "expose everything".

Recently I learned that you can open files in vim to particular lines:

```
vim my_file +72
```

This is a super useful part of my workflow that allows me to jump directly to where tests
are failing.

I've also learned how to pipe files into vim:

```
unstaged | xargs -o vim
```

This has become my go-to method for editing multiple files since I can `grep` for the files
I want to edit and then batch process them (technically I can use `argdo` but I tend to use
macros instead).

I was reviewing my dotfiles setup recently and I realized that I was using another cool feature
of vim which allows you to use it as more of an ad-hoc tool. My dotfiles setup has something like
the following:

```
vim :PluginInstall :qall
```

This runs Vundle's `:PluginInstall` which install all of my dependencies then follows up with exiting
vim via `:qall`. This is really useful as it pops into vim, does a few commands, then exits and returns
to my script. The moral of the story is that we can tell vim what to do via the command line.

Now I just need to figure out how to run arbitrary primitives like `Iworld<Esc>bIhello <Esc>:wq helloworld.txt<CR>`
and then we'll *really* have something powerful on our hands.
