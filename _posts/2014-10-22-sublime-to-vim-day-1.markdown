---
layout: post
title: "Sublime to Vim: Day 1"
date: 2014-10-22 21:02
comments: true
categories: vim
---

Today I made my switch over to Vim from Sublime. I thought that things we be a lot more painful, but they really weren't too bad. I feel pretty comfortable and only had to pop into Sublime once or twice to do some cross-file searches.

First thing I did was disable the arrow keys. I wound up turning these back on for insert mode later

My .vimrc

```vim
" pathogen setup
execute pathogen#infect()
filetype plugin indent on

" colorscheme
colorscheme facebook

" enable syntax processing
syntax enable

" line numbers
set number

" font
set guifont=Menlo\ Regular:h13

" disable arrow keys
map <up> <nop>
map <down> <nop>
map <left> <nop>
map <right> <nop>
imap <up> <up>
imap <down> <down>
imap <left> <left>
imap <right> <right>

" tabs to spaces; tabs=2 spaces
set expandtab
set tabstop=2
set shiftwidth=2

" don't wrap
set nowrap
```

I had some trouble getting OSX Terminal to play nice with my color scheme, so I switched over to macvim. macvim is great because it still gives you the OSX conveniences as you make the transition (like using alt to skip words or command to go to end of line). I used Pathogen as a bundle manager (although I'll probably switch to Vundle to have everything in my .vimrc). I also used Ctrl-P (go to anything), NERDTree (explorer sidebar), Supertab (autocomplete), and Fugitive (Git integration). I plan to install Airline and Snipmate.

For commands I learned (C = Ctrl):

```
h = left
j = down
k = up
l = right

w = forward word
b = backward word
G = EOF
1G = BOF
:42 = Goto line 42 (also 42gg, 42G)

/ = find
n = find next
N = find previous

u = undo
C-R = redo

cw = change word

dd = delete line
dw = delete word

> = indent

i = insert
a = insert after
I = insert at beginning of line
A = insert at end of line
```

I also learned how to call executables from the buffer which is really sweet:

e.g. run a Ruby test from the test file
```
! ruby -Itest %:p
```

TODO:

- Figure out how to get snippets
- Check for refactoring commands
