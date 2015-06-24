---
layout: post
title: "Vim Tricks: Ctrlp and Gems (Take 2)"
date: 2015-03-16 11:26:43 -0400
comments: true
categories:
---

A little while ago I wrote a post regarding how to work with Ctrlp and your Gem
directory effectively. I've since found a way that I like a lot more. The one
issue with the following approach is that it assumes that you are using a Gemfile.

To start, toss this in your vimrc:

```vim
" open gem dir in ctrlp
function! GemCtrlP(gem_name)
  let path = system('bundle list ' . a:gem_name . ' | tr -d "\n"')
  execute 'CtrlP ' . path
endfunction
command! -bang -nargs=* -complete=file Gem call GemCtrlP(<q-args>)
```

Usage works like so: `:Gem rails`

The `function!` call defines the `GemCtrlP` function. The bang will overwrite
the function if it is already defined (e.g. if you source your .vimrc).
The `command!` call allows you to call `:Gem my_gem` in command mode.
The function works by making a syscall to bundler to get the gem path.
It then opens Ctrlp in that path.
