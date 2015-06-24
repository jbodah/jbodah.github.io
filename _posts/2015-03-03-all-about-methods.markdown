---
layout: post
title: "All About Methods"
date: 2015-03-03 19:11:45 -0500
comments: true
categories:
---

One of the things I find most confusing about Ruby is how methods are defined and bound to instances.
Even after 4 years of working with the language, I still struggle to understand how exactly classes and method definitions in Ruby interact.
I learned a lot about the Ruby object model while working on [spy_rb](https://github.com/jbodah/spy_rb).
To help clear up my thinking (and hopefully yours), I've decided to do a write up on how methods work in Ruby.

The way I see it, Ruby really has three primitive classes that you must know: `Object`, `Module`, and `Class`.
`Class` is a subclass of `Module`, and `Module` is a subclass of `Object`.

```rb
Class < Module
#=> true
Module < Object
#=> true
```

Thus, every instance of `Class` or `Module` is also in Ruby is also an instance of `Object`.
To be more concrete, when we type `class MyClass` we are really just creating a new constant `MyClass`
and assigning it the value of `Class.new`. Alternatively, we could be more explicit: `MyClass = Class.new`.
`MyClass` is still an `Object` at some level.

One other important thing about Ruby is the idea of the singleton class.
Every instance of `Object` also has it's own class special to that instance (hence the name "singleton").
There are a few ways to access the singleton class:

```rb
# a subset of the ways to access the singleton class
Object.new.singleton_class
#=> #<Class:#<Object:0x007fbd0c0efda0>>

class << Object.new; self; end
#=> #<Class:#<Object:0x007fbd09147c10>>
```

This class is dynamically generated and, while it doesn't show up in the ancestor chain, it is first in line
in Ruby's method lookup chain (note: it's beyond the scope of this post, but the lookup chain is roughly first check the singleton class, then class, mixins, ancestors).

```rb
obj = 'hello'
obj.to_s
#=> "hello"

# NOTE: The "-> {}" syntax is equivalent to Proc.new and similar
# to defining a block in case you aren't familiar with it
obj.define_singleton_method :to_s, -> { 'world' }
#=> :to_s

obj.to_s
#=> "world"

obj.method(:to_s).owner == obj.singleton_class
#=> true
```

Methods in Ruby have several interesting properties. One is `owner`.
I like to think of this as where to method lives (as on which object it lives on,
not to be confused with the very useful `source_location`).
In our example above, the owner is the singleton class of `obj`.

Another neat property is `receiver`. When Ruby does it's lookup, it's essentially
looking at all methods where the called object is the receiver. This is similar to
how languages like Smalltalk and Io work as well. You'll sometimes hear this referred
to as "message passing". In the example above, `obj` is the receiver because it
responds to the message.

```rb
obj.method(:to_s).receiver
#=> "hello"
```

This isn't the best example as that just prints the value of the string, but you can
try it with something like an `Object` or `ActiveRecord` instance and check the id.

There are a few other properties you can explore (see ruby-doc for `Object`) like `name`, but those are the two most important ones.

In addition to its properties, each method also has a block of code associated with it.
Ruby evaluates that block by applying it to a context (or binding) at run time.

Now to talk about method ownership. A vanilla instance of `Object` cannot own methods.
If you look at the ruby-doc for `Object` you'll see that `define_method` isn't there.
`define_singleton_method` is there, but that just calls `define_method` in that object's
singleton class. Comparing it with our above example:

```rb
obj2 = 'john'
obj2.singleton_class.instance_eval { define_method :to_s, -> { 'dough' }}
#=> :to_s

obj2.to_s
#=> "dough"
```

In Ruby only instances of `Module` (and by inheritance, `Class`) can own methods.
Looking at the ruby-doc for `Module` you should see that it has the method `define_method`.
When we call `define_method` we are creating a new method whos `owner` is the target class/module.
If you're still with me, then you might think of that last statement as: `define_method` creates
a method such that the method's `owner` is the `receiver` of the `define_method` call
(don't worry too much if you didn't catch that).

Class methods are something that you have in languages like Java and C#, but they don't really exist
in Ruby. In fact, I recommend you toss the idea of class methods in Ruby altogether.
Instead you can simulate class methods by defining methods on the class's singleton class.
These are all equivalent:

```rb
class MyClass
  def self.hello
    'world'
  end
end

def MyClass.hello
  'world'
end

MyClass.define_singleton_method :hello, -> { 'world' }

class << MyClass
  def hello
    'world'
  end
end

class MyClass
  class << self
    def hello
      'world'
    end
  end
end

MyClass.singleton_class.instance_eval { define_method :hello, -> { 'world' }}

# And the call:
MyClass.hello
#=> "world"
```

The important thing is to think more in how each object has a singleton.
When you want class-level responsibilities, one way to do that is to just define them
on the class dedicated to the class instance in mind.
In fact, all method are essentially instance methods.
That is, when you call `define_singleton_method`, you're really just defining an instance method in the singleton class.
Ruby is really centered around this idea (ex. when you mix in a `Module` via `include` you are "copying" the instance methods
of the module into your current class/module).
You can make some impressively flexible and modular code by desiging around this principle.

How methods are bound is also important. When you reference a method in the context
of an instance, then you'll generally get a `Method` object. However, if you reference
the method in the context of a class/module then you'll get an `UnboundMethod`.
The key difference between the two is that `Method` has a `call` method while `UnboundMethod`
needs to be bound to an instance via `bind` first. That will convert the method into a `Method` which is callable.

(note: In case you're playing around with this on your own,
it's worth mentioning that `define_method` can accept a `Proc`, a `Method`, or an `UnboundMethod`.
Also remember that `define_method` provides a closure over its local variables.
Instance variables are not closed on)

```rb
class MyClass
  def hello
    'world'
  end
end

obj = MyClass.new
#=> #<MyClass:0x007fbd0f3068e8>

m = MyClass.instance_method(:hello)
#=> #<UnboundMethod: MyClass#hello>
m.owner
#=> MyClass
m.receiver
#=> NoMethodError: undefined method 'receiver'
m.call
#=> NoMethodError: undefined method 'call'

m = obj.method(:hello)
#=> #<Method: MyClass#hello>
m.owner
#=> MyClass
m.receiver
#=> #<MyClass:0x007fbd0f3068e8>
m.call
#=> "world"
```

What about mixins?

```rb
module Greeter
  def hello
    'world'
  end
end

class MyClass
  include Greeter
end

m = Greeter.instance_method(:hello)
#=> #<UnboundMethod: Greeter#hello>
m.owner
#=> Greeter

m = MyClass.instance_method(:hello)
#=> #<UnboundMethod: MyClass(Greeter)#hello>
m.owner
#=> Greeter
```

Did you catch that?
`Greeter.instance_method(:hello)` returns `#<UnboundMethod: Greeter#hello>`
but `MyClass.instance_method(:hello)` returns `#<UnboundMethod: MyClass(Greeter)#hello>`.
Although the code block is the same for each of these methods, they are not equivalent:

```rb
m1 = Greeter.instance_method(:hello)
#=> #<UnboundMethod: Greeter#hello>

m2 = MyClass.instance_method(:hello)
#=> #<UnboundMethod: MyClass(Greeter)#hello>

m1 == m2
#=> false
```

This brings up the last important topic that's really the crux of why I'm writing this post
as it led to a lot of headache for me: transplanting/rebinding methods.

Time for a case study:

The idea behind [spy_rb](https://github.com/jbodah/spy_rb) is that you want to transparently listen when a method is called on some target object.
There are two major groups that fall into this category:

  1. Spying on a method that isn't owned by the spy target or its singleton in which case we abuse the method
     lookup chain to spy on the method. When we are finished spying we can just remove the method.

  2. Spying on a method that is owned by either the spy target or its singleton in which case we need to wrap
     the original method. When we are finished spying in this case we need to rebind the original method.

The first case was pretty straightforward. It's the second case that brought the pain.
The reason for this is that methods can only be bound to objects who have the method owner in their ancestor chain.

For example:

```rb
module Greeter
  def hello
    'world'
  end
end

class MyClass
  include Greeter
end

# Save references
mod_meth = Greeter.instance_method(:hello)
klass_meth = MyClass.instance_method(:hello)

# Delete the method
Greeter.instance_eval { remove_method :hello }

MyClass.new.respond_to?(:hello)
#=> false

# Defines just fine
Greeter.instance_eval { define_method :hello, mod_meth }
#=> :hello

# Breaks
Greeter.instance_eval { define_method :hello, klass_meth }
#=> TypeError: bind argument must be a subclass of MyClass
```

To bend your mind a little further:

```rb
str = 'test'

mod_meth.bind(str).call
#=> :hello

klass_meth.bind(str).call
#=> TypeError: bind argument must be an instance of MyClass

str.define_singleton_method :hello, mod_meth
#=> :hello

str.define_singleton_method :hello, klass_meth
#=> TypeError: bind argument must be a subclass of MyClass
```

Personally I'm still trying to grok what this means for my designs
(not that I recommend you go around transplanting methods everywhere, but it certainly has a use).

So that's Ruby methods in a nutshell. There's still a good amount of things I didn't talk about
like method visibility and Ruby's extremely powerful method reflection capabilites, but I think
if you've made it this far then you shouldn't have any problem figuring out the rest
as you go. I hope that you found this useful, learned something, and maybe even had
some light bulb click for some problem you've been working on.

If you're looking for cool examples of what you can do with methods, then I recommend
checking out the useful debugging tool [pry](https://github.com/pry/pry) as well as the testing gem [spy_rb](https://github.com/jbodah/spy_rb).
