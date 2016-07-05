---
layout: post
title: 'Dealing with Undocumented Hashes'
date: 2016-01-21 20:28:18 -0500
comments: true
categories:
---

If there is one thing Ruby has taught me it is that simply
putting my head to the keyboard and trying bang out painful refactors never solves my issues in
the long run.
It's like every line of code I simplify, ten more lines of terrible code get checked in somewhere else.

There are a lot of Ruby patterns that cause a lot of pain (deep inheritance, mix-in hell) but I think
hashes are the worst offenders.
There are exceptions (Rack abides by HTTP, dynamic code gets a freebie, documenting your methods helps), 
but in general taking a hash as an argument and doing something like
binding it to an instance variable is like saying "fuck you" to anyone who wants try and wrap their head around your code.

Before we go on, let's just see an example of how much of a pain in the ass hashes can be:

```rb
class Car
  attr_reader :attrs

  def initialize(attrs = {})
    @attrs = attrs
  end
end
```

This seemingly innocuous class is really simple on the surface, but in truth what you've done is broken the lock
on the deepest gate of dependency hell. Yes you've let me pass in whatever I want. That sucks, but at least it's
scoped to `Car.new`. What's worse however is that you've let me **reference** these dynamic attributes.

Undocumented code sucks, but undocumented dynamic code is the absolute worst. To give you an idea of what I mean,
take a look at this snippet of code:

```rb
class TestClass
  include MixinOne
  include MixinTwo
  include MixinThree

  def initialize(options = {})
    @options = options
  end

  def some_method_one
    # ...
  end

  def some_method_one
    # ...
  end

  def some_method_one
    # ...
  end
end
```
