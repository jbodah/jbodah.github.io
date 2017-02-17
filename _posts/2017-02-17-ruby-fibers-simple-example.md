---
layout: post
title: 'Ruby Fibers: A Simple Example'
date: 2017-02-16 22:27:27 -0500
comments: true
categories:
---

[Fiber](https://ruby-doc.org/core-2.4.0/Fiber.html) is one of those classes in Ruby that has always been mysterious to me.
The RubyDocs describe Fibers as being "primitives for implementing light weight cooperative concurrency".
Yep, still mysterious.
The example makes things a little clearer, but it still not entirely clear why anyone would want to use a Fiber:

```rb
fiber = Fiber.new do
  Fiber.yield 1
  2
end

puts fiber.resume
#=> 1
puts fiber.resume
#=> 2
puts fiber.resume
#=> FiberError: dead fiber called
```

If you're already familiar with Fibers then you probably know that they can be used to implement `Enumerator`, an extremely
useful and versatile class that deserves a post of its own.

Amidst a late night coding session I finally found a reason to use them - something not too complicated, but real enough to hit home.
In my adventures I was trying to create a `Counter` class. Long story, short - I needed to generate increasing primary ID's and
wanted `counter.next` to just give me the next available ID.

Now, I could just store the counter in an instance variable in another class, but that solution wouldn't be great - I'd have to do something
like `@counter += 1` all over the place, which feels a little primitive and it feels like I'm breaking encapsulation. I could just wrap that logic
up into its own class too, but I tend to find that that breaks up the code a bit more than I'd like it to.

In the past I've seen Enumerators used to loop forever; could I do the same thing with Fibers? Here's what I came up with:

```rb
class Counter
  def initialize(init)
    @fiber = Fiber.new do
      n = init
      loop do
        Fiber.yield n
        n += 1
      end
    end
  end

  def next
    @fiber.resume
  end
end

counter = Counter.new(0)
counter.next
#=> 0
counter.next
#=> 1
counter.next
#=> 2
```

If you can't follow: basically every call to `@fiber.resume` switches our execution context back into the Fiber. When `Fiber.yield` is called,
the Fiber is yielding execution back to our main thread.
This is why we don't get stuck in an infinite loop.
Is this the best use of Fibers? Would this kind of thing hold up in a code review for production code?
Probably not, but I do think this leads to an interesting solution to the counter problem and gives a taste of how powerful Fibers can be.
I'd love to hear about how you've seen Fibers used in the wild in the comments
