---
layout: post
title: "Want Guard for Non-Ruby Projects? Try Watch"
date: 2014-12-02 22:07:19 -0500
comments: true
categories: 
---

I've gushed a lot about Guard. It's really transformed my workflow, and I love using it for
TDD. I've been reading Seven Languages in Seven Weeks and going through exercises in Io, and
I wanted to get some TDD-style feedback:

```sh
watch -n1 io two_dim_list.io
```

This will run the command `io two_dim_list.io` every second (in this case my test file).

If you're using OSX, you can install `watch` with Homebrew:

```sh
brew install watch
```
