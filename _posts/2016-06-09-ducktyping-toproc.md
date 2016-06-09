---
layout: post
title: 'Duck-typing with #to_proc'
date: 2016-06-09 09:08:28 -0400
comments: true
categories:
---

One of the greatest Ruby mysteries for me has always been the idiomatic `Symbol#to_proc` which is usually used to cast
a symbol to a block usually passed to some enumerator. More concretely, it's common to see code like this:

```rb
class Integer
  def increment
    self + 1
  end
end

[1, 2, 3].map(&:increment)
#=> [2, 3, 4]
```

This winds up just being short-hand for something like:

```rb
[1, 2, 3].map { |n| n.increment }
```

The `&:increment` syntax is doing two things. First, it is calling `Symbol#to_proc` which turns the symbol into a proc, or an
anonymous function (more on this in a second). Next it treats the resulting `Proc` as a "block". In the above code that block
is then passed to `Array#map` where it is executed.

Let's dig into `Symbol#to_proc` a bit to figure out what's actually going on under the hood. In the following code we take the
`:size` symbol and will procify it. Then we'll apply it to two objects to see what happens:

```rb
fn = :size.to_proc
#=> #<Proc:...>

fn.call([1, 2, 3])
#=> 3

fn.call({ name: 'josh', food: 'pizza' })
#=> 2
```

So from this example we can see that calling `Symbol#to_proc` winds up resulting in a something similar to

```rb
Proc.new { |receiver| receiver.send(:size) }
```

What if our method can take arguments? For example, let's say we have a `name` method that can take the `:formal` option.

```rb
class Person
  def name(&block)
    block_given? ? 'Mr. Josh' : 'Josh'
  end
end

person = Person.new

fn = :name.to_proc
#=> #<Proc:...>

fn.call person
#=> 'Josh'

fn.call person, formal: true
#=> 'Mr. Josh'
```

Nice, so the proc that is generated can take arguments (blocks too!).

Let's talk about a practical application now. I was writing some code today where
I wanted to create a list of symbols representing methods which I would then
enumerate over using `Object#public_send` to make dynamic calls. Explicitly:

```rb
class Person
  def name
    'josh'
  end

  def favorite_food
    'pizza'
  end

  def dog_name
    'penny'
  end
end

person = Person.new

[:name, :favorite_food, :dog_name].map do |sym|
  person.public_send(sym)
end
#=> ['josh', 'pizza', 'penny']
```

That was working great for a long time, however then things started to get more complex
and I needed to pass arguments to my methods. For example, I wanted to have `Person#favorite_food`
take a `:vegetable` option. That throws a wrench in things because I can't really pass arguments
on these dynamic calls very easily without doing a branch (e.g. `if Symbol... elsif Proc`).

```rb
class Person
  def name
    'josh'
  end

  def favorite_food(vegetable: false)
    vegetable ? 'broccoli' : 'pizza'
  end

  def dog_name
    'penny'
  end
end

person = Person.new

[
  :name,
  :favorite_food,
  :dog_name,
  -> { person.favorite_food(vegetable: true) }
].map do |sym_or_proc|
  case sym_or_proc
  when Symbol
    person.public_send(sym_or_proc)
  when Proc
    sym_or_proc.call
  end
end
#=> ['josh', 'pizza', 'penny', 'broccoli']
```

Or so I thought. As we learned above, `Symbol#to_proc` creates a proc that takes as an argument the receiver that
we want to pass the symbol to as a message.
Well, what if we just replace the `:favorite_food` symbol in our array with a proc that will take the receiver
we want to use and pass arguments along.
`Proc#to_proc` just returns the same proc so this should allow us to still use duck-typing to keep things elegant:


```rb
class Person
  def name
    'josh'
  end

  def favorite_food(vegetable: false)
    vegetable ? 'broccoli' : 'pizza'
  end

  def dog_name
    'penny'
  end
end

person = Person.new

[
  :name,
  :favorite_food,
  :dog_name,
  -> (person) { person.favorite_food(vegetable: true) }
].map do |procable|
  fn = procable.to_proc
  fn.call(person)
end
#=> ['josh', 'pizza', 'penny', 'broccoli']
```

In the above example we use the "stabby-lambda" syntax to define our proc (TL;DR lambdas are a type of `Proc`).
Since both `Proc` and `Symbol` respond to `:to_proc` then we just call that to normalize the
argument into something we can handle generically. If you're not familiar with this
approach of building interfaces, this is called "duck-typing", and here we've proven
just how effective this technique can be in simplifying our code. If you're not familiar with
this technique then I highly recommend you also take a look at the interfaces for `Enumerable`, `Comparable`, and `IO`
which allow duck-typing on `each`, `<=>`, and `read`/`write` respectively.
