---
layout: post
title: 'Using `bundler --standalone`'
date: 2017-11-22 11:09:41 -0500
comments: true
categories:
---

[Bundler](http://bundler.io/) is a staple in the Ruby community for managing dependencies, and it is so good that
[it is now a part of Ruby core](https://bugs.ruby-lang.org/issues/12733).
The typical use case of Bundler is pretty simple:

```
# Get bundler
gem install bundler

# Create a Gemfile/Gemfile.lock
bundle init

# Edit your Gemfile

# Install your deps
bundle install

# Run Ruby scripts which lock your gems to those managed by bundler
bundle exec ruby my_script.rb
```

This flow is great for development and local scripts, but it kind of sucks if you need to use the bundle outside of
that directory. One way to get around this is to `require "bundler/setup"` in your scripts. This will make it so
you don't need to use the `bundle exec` prefix (as your script will now "activate" bundler).

There are more problems still though. In particular, what happens if you work with multiple versions of Ruby? If you
do then your current Ruby's Bundler will be loaded (assuming you have it installed) and then you'll need to make sure
you have all your dependencies for that Ruby version, make sure you don't have potential version conflicts... it's not fun.

Luckily Bundler has a `--standalone` option that will install of the gems into the project's `bundle` directory. This
changes the lookup from a global lookup to a local lookup. The project will have (almost) everything it needs to run
right there allowing you to have consistent behavior despite what your global gem paths look like.

Doing `bundle install --standalone` is enough to make the installation a standalone installation. Then you will need to switch
your scripts to `require "bundle/bundler/setup"` so that they know to activate your standalone bundle instead of bubbling to
your global bundler instance.

With that you should be able to rest assured that no matter where your project is called from (for example, I have some aliases to
some executables in some of my checked out projects) that you will load a consistent set of dependencies when your code is called.
