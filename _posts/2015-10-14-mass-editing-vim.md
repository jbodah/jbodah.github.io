---
layout: post
title: "Mass Editing in Vim"
date: 2015-10-14 16:00:53 -0400
comments: true
categories:
---

One of the things that I've found really tricky about working with Vim has been mass search and replace.
Luckily I've been reading the excellent [Practical Vim](http://www.amazon.com/Practical-Vim-Thought-Pragmatic-Programmers/dp/1934356980) by Drew Neil.
In it Drew explains how to use `:argdo` to edit multiple buffers at once.
Here's how I've been using it lately and it's been working really nicely.

First, I have three aliases for working with git. These each give me pipeable portions of my git status:

```sh
# Pipe untracked files
function untracked {
  git status --porcelain | grep ?\? | awk '{ print $2 }'
}

# Pipe modified but unstaged files
function unstaged {
  git status --porcelain | grep '^.[^\s]' | awk '{ print $2 }'
}

# Pipe files staged for commit
function staged {
  git status --porcelain | grep '^[^\s]' | awk '{ print $2 }'
}
```

Then to edit files I can do the following (note I'm also using [ag](https://github.com/ggreer/the_silver_searcher) instead of grep):

```sh
staged | ag *_test | xargs -o vim
```

```vim
:argdo %s/ActiveSupport::TestCase/Backupify::TestCase/g
:wall
:q
```
