---
layout: post
title: 'Using bash expansions'
date: 2016-11-17 13:30:21 -0500
comments: true
categories:
---

If you're using the command line all day then it's critical that you become an expert with it.
That means learning the basic commands, reading man pages, mastering IO redirection, learning
how job control works, etc. I highly recommend [Learning the Shell](http://linuxcommand.org/lc3_learning_the_shell.php).

Let's look at a real-life example of how we can use bash expansion to make life easier.
You've probably used bash expansion if you've ever done anything like `rm test/*`. In
this example I have a bunch of docker networks that I want to clean up:

```sh
$ docker network ls
NETWORK ID          NAME                       DRIVER              SCOPE
06f6f6d67193        bridge                     bridge              local
1a88ec257b8a        host                       host                local
9af9672816f2        none                       null                local
0b8a0c32514e        testtaskrunner0_default    bridge              local
affc61d7834c        testtaskrunner10_default   bridge              local
8f974066b122        testtaskrunner1_default    bridge              local
e66a6a5252a0        testtaskrunner2_default    bridge              local
5e4002eac566        testtaskrunner3_default    bridge              local
b8ea7337cbc3        testtaskrunner4_default    bridge              local
e72c903cd99b        testtaskrunner5_default    bridge              local
1d098664e8b2        testtaskrunner6_default    bridge              local
06b7d2081c7c        testtaskrunner7_default    bridge              local
f387a7ac86fa        testtaskrunner8_default    bridge              local
f94bb5a23dee        testtaskrunner9_default    bridge              local
```

The command for removing a network is `docker network rm NAME` (or in my case `docker-compose --project NAME down`).
I could copy and paste a million times, but lets be smart instead. First let's use brace expansion to create a range then
replace all of the spaces with new lines:

```
$ echo testtaskrunner{0..10} | sed $'s/ /\\\n/g'
testtaskrunner0
testtaskrunner1
testtaskrunner2
testtaskrunner3
testtaskrunner4
testtaskrunner5
testtaskrunner6
testtaskrunner7
testtaskrunner8
testtaskrunner9
testtaskrunner10
```

Now we can pipe those into `xargs` to shutdown each network:

```
$ echo testtaskrunner{0..10} | sed $'s/ /\\\n/g' | xargs -I {} -n 1 docker-compose --project {} down
```

Here `-n 1` means "run this command for each line received in STDIN (as opposed to using multiple lines in the command)".
`-I {}` means "replace all `{}` with whatever got passed into STDIN".
