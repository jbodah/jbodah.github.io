---
layout: post
title: 'DSL's Explained'
date: 2015-12-07 11:08:49 -0500
comments: true
categories:
---

DSL's (or [domain-specific languages](https://www.leighhalliday.com/creating-ruby-dsl) for the uninitiated) are a bit of a controversial topic.
On one hand they can be extremely expressive and help you write code that is easier to read and faster to write.
On the other they can become convoluted, confusing, and difficult to debug.
A coworker of mine recently asked me a question about a DSL, so I figured I'd write a bit about them.
In this post I hope to explain the lessons I've learned writing DSL's and explain the way that I like to do it.
There are several ways to write them, but I'm going to focus on the way I do it, and I'll step you through my thought process.

The first thing you should do when you want to write a DSL is to ask yourself why you want to do it.
DSL's add a good amount of complexity to your API, so the overhead should feel worth it.
The most common reason I like to use them for is encapsulating a series of steps, configuration, or some general wrapper logic.
I'll use one for encapsulation in this example.

Let's say we're making some app that will go around and do some shopping for us.
We might imagine that this interface might look like the following in a classical example:

```rb
class Shopper
  def initialize
    @location = nil
    @cart = []
    @posessions = []
  end

  def visit(location)
    @location = location
    @cart = []
  end

  def add_to_cart(item)
    @cart << item
  end

  def checkout
    @location = nil
    @posessions += @cart
    @cart = []
  end
end

shopper = Shopper.new

shopper.visit 'Target'
shopper.add_to_cart 'T-shirt'
shopper.add_to_cart 'Onesie'
shopper.checkout

shopper.visit 'Kohls'
shopper.add_to_cart 'Underoos'
shopper.checkout
```

This is okay, but there are some problems I see with it.
First, there's state being maintained when we `visit` different stores.
Second, we have to call `checkout` at the end of every transaction.
Lastly, it might be difficult to see the hierarchy of what's going on and which code belongs to which transaction if we had a lot of code.

Let's try again using a block interface:

```rb
class Shopper
  def initialize
    @posessions = []
    @location = nil
  end

  def shop(location)
    @location = location
    @cart = []
    yield self
    @posessions += @cart
  ensure
    @cart = []
    @location = nil
  end

  def add_to_cart(item)
    @cart << item
  end
end

shopper = Shopper.new

shopper.shop 'Target' do |s|
  s.add_to_cart 'T-shirt'
  s.add_to_cat 'Onesie
end

shopper.shop 'Kohls' do |s|
  s.add_to_cart 'Underoos'
end
```

Here we've done a couple of things.
The API now has a nice blocky look to it too.
The reader can easily differentiate transactions when looking at the calls.
We've also streamlined the transaction by encapuslating `checkout` into the `shop` method after the `yield`.
This helped centralize some of the logic around resetting the state.
Having to maintain this state still kind of sucks, and we could get around that with a helper object instance.

```rb
class ShoppingTrip
  def initialize(location)
    @location = location
    @cart = []
  end

  def add_to_cart(item)
    
  end
end

class Shopper
  def initialize
    @posessions
  end

  def shop(location)
    trip = ShoppingTrip.new(location)
    # Proc.new captures the given block to a Proc,
    # and & converts it back to a block as we call ShoppingTrip#shop
    items = trip.shop(&Proc.new)
    @posessions += items
  end
end
```
