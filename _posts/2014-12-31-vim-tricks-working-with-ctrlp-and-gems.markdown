---
layout: post
title: "Vim Tricks: CtrlP and Gems"
date: 2014-12-31 11:50:55 -0500
comments: true
categories:
---

My transition to Vim has been pretty good so far. There are still a couple
painful points though. One of those is with the fantastic [CtrlP plugin](https://github.com/ctrlpvim/ctrlp.vim)
It took me a bit of digging to figure out how to add my gems to CtrlP:

```
# Open Vim in working directory
vim

# CtrlP your gem directory
:CtrlP ~/.rbenv/versions/2.1.3/lib

# CtrlP your working directory
<c-p>
```

I also created a mapping for mine:

```vim
" open gem dir in ctrlp
nnoremap <leader>gem :CtrlP ~/.rbenv/versions/2.1.3/lib/ruby/gems/2.1.0/gems/
```

Then you can do something like:
```vim
<leader>gem<cr>
minitest<tab, complete for version needed><enter>
```

EDIT:

You may want to bump your max files in CtrlP with the following:

```vim
" unset cap of 10,000 files so we find everything
" src: https://github.com/kien/ctrlp.vim/issues/325
let g:ctrlp_max_files = 0
```
