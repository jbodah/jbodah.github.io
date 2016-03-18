---
layout: post
title: "Pro-Tip: Using Executables From Local node_modules"
date: 2016-03-17 23:25:33 -0400
comments: true
categories:
---

One thing that always bothers me about working in node is the number of examples
with `npm install -g`. Global dependencies are a huge pain in the ass especially
when you need to work with multiple projects that use different versions of the same
package.

Tonight I found out that node stores your executables in your local `node_modules`
under `node_modules/.bin`. So if I wanted to do `ember new <MY_PROJ>` I could do
something like `node_modules/.bin/ember new <MY_PROJ>`. If you're doing that a lot
then you can quickly alias it in your current shell:

```sh
$ alias ember=`pwd`/node_modules/.bin/ember
$ ember new <MY_PROJ>
```

Alternatively, if you have a bunch of stuff in your `node_modules/.bin` then just
add that to your current shell's `PATH`:

```sh
$ export PATH=`pwd`/node_modules/.bin:$PATH
$ which ember
#=> /Users/jbodah/repos/ember_noob/node_modules/.bin/ember
```
