---
layout: post
title: "spy_rb: A Secret Agent for Ruby"
date: 2015-07-10 09:29:33 -0400
comments: true
categories:
---

## A Tale of Romance

One of my favorite pieces of software in the whole world is [Sinon.JS](https://github.com/cjohansen/Sinon.JS/).
If you're not familiar with this library, it's a dead-simple
testing library for Javascript applications that lets you see when functions are called without stubbing them.
That is, by default it will always call through methods, but it will still record all sorts of interesting information about the calls.

I tend to prefer spying to the traditional approach of stubbing methods. One of the things that bothers me about stubs is that your
implementation of your stubbed code could change, but your tests could still pass because they rely on your stub and not the actual
implementation. There's certainly a place for both, but calling through allows your code to still run, so I prefer it when given the option.

## Introducing [spy_rb](https://github.com/jbodah/spy_rb)

About a year ago I started creating a similar library for Ruby which I call [spy_rb](https://github.com/jbodah/spy_rb). `Spy` lets you do several cool things.
I think it's easiest to just give some examples.

Spying starts declaring a method you want to spy on. There are several ways to do this depending on you use case. (NOTE: In short, [spy_rb](https://github.com/jbodah/spy_rb) works
by either wrapping a method under spy or intercept it by dynamically inserting a method on an object's singleton so it is called first in the lookup chain)

```rb
Spy.on(MyClass, :new)                 # Spy on a MyClass's singleton method :new
Spy.on(my_obj, :hello)                # Spy on my_obj's bound method :hello
Spy.on_any_instance(MyClass, :hello)  # Spy on MyClass's unbound instance method :hello
```

Once you have a `Spy::Instance` then you just simply call your code like normal and [spy_rb](https://github.com/jbodah/spy_rb) will record things you might interested.

```rb
class MyClass
  def initialize(msg)
    @msg = msg
  end
end
spy = Spy.on(MyClass, :new)
MyClass.new('hello')
puts spy.call_count
#=> 1
puts spy.call_history.first.args
#=> ["hello"]
puts spy.call_history.first.result
#=> #<MyClass:0x007f8a8d1d57a8 @msg="hello">
```

When you're finished, just call `Spy.restore` and your original implementations will be restored.

```rb
puts MyClass.method(:do_stuff).source_location
#=> ["/Users/Bodah/repos/test_app/models/my_class.rb", 68]
spy = Spy.on(MyClass, :do_stuff)
puts MyClass.method(:do_stuff).source_location
#=> ["/Users/Bodah/.rbenv/versions/2.1.3/lib/ruby/gems/2.1.0/gems/spy_rb-0.4.1/lib/spy/instance/api/internal.rb", 14]
Spy.restore(:all)
puts MyClass.method(:do_stuff).source_location
#=> ["/Users/Bodah/repos/test_app/models/my_class.rb", 68]
```

`Spy::Instance` provides you with callbacks and filters to do all sorts of intricate things. Check out the full list in the [README](https://github.com/jbodah/spy_rb/blob/master/README.md).

```rb
spy = Spy.on(MyClass, :new)
spy.before { |*args| puts args.join(', ') }
MyClass.new('hello')
#=> "hello"

Spy.restore(MyClass, :new)

other_spy = Spy.on(MyClass, :new)
other_spy.when { |msg| msg != 'hello' }
MyClass.new('hello')
puts other_spy.call_count
#=> 0
```

There are even some experimental features like replaying your call history with `Spy::Instance#replay_all`

```rb
spy = Spy.on(self, :puts)
puts 'hello'
#=> "hello"
puts 'goodbye'
#=> "goodbye"
spy.replay_all
#=> "hello"
#=> "goodbye"
```

## But Wait, There's More

While [spy_rb](https://github.com/jbodah/spy_rb) is an excellent testing tool, its real power comes through in debugging and analysis. Below I list several instances
where I've used [spy_rb](https://github.com/jbodah/spy_rb) to do powerful things outside of tests.

### Debugging Nefarious Method Calls

We had a record in our Rails app that was being saved automatically. When our workers ran at scale, this caused contention in the database and
wound up slowing jobs to a crawl. Digging through thousands of lines of source code is not really fun. Luckily, I could simply throw a spy on my model,
run the job, and inspect the backtrace.

```rb
spy = Spy.on_any_instance(MyModel, :save)
spy.before { binding.pry }
@job.perform

# in pry
pry-backtrace
```

### Profiling and Benchmarking

I've used [spy_rb](https://github.com/jbodah/spy_rb) for quick benchmarking and profiling too. Our test suite can be painfully slow at times. Some initial testing led me to believe that
we were spending a lot of time with [factory_girl](https://github.com/thoughtbot/factory_girl) calling `FactoryGirl.create` instead of the more performant `::build` or `::build_stubbed`. I could
save the original method and wrap it, but `Spy::Instance#wrap` gives me a much cleaner interface for doing so. (NOTE: The example below is a little simpler
than the actual implementation was as this example doesn't handle recursion properly).

```rb
@time_spent = 0
spy = Spy.on(FactoryGirl, :create)
spy.wrap do |*args, &block|
  # Wrap the original call in a benchmark
  @time_spent += Benchmark.measure { block.call }.real
end

# run test suite

puts @time_spent
```

### Runtime Analysis

Continuing off of the `FactoryGirl` example above, I found that `FactoryGirl.create` made up a significant portion of our test duration. But what can I do about that?
A global substitution would certainly break a ton of things, but going through every example by hand sounds pretty painful too. Maybe you're detecting a pattern here,
but [spy_rb](https://github.com/jbodah/spy_rb) provides us tools to do this in a smarter way!

Using `Spy::Instance#wrap` I put together a small library that would:

  1. Wrap each test run

  2. Detect how many calls are made to `FactoryGirl.create`

  3. For each call made:

      a. Stub the nth call to `FactoryGirl.create` with `FactoryGirl.build` or `FactoryGirl.build_stubbed`

      b. Rerun the test

      c. Log whether or not the test passed (e.g. we can substitute `::create` for a more efficient method and the test still passes)
  4. Report all of the valid substitutions we can make

Overall this wound up giving us a 20% speed boost. [The code](https://gist.github.com/jbodah/e1a9bbe1ab321b479b41) is pretty hairy and I'm still working on some refactoring, but this is a great example of the power that spying
can give you.

## Conclusion

If you have never used test spies before, then I highly recommend you play with them. You can do all sorts of really cool things with them, and it shows the incredible
power of working with a dynamic language. For Javascript applications I really love [Sinon.JS](https://github.com/cjohansen/Sinon.JS/), and now with
[spy_rb](https://github.com/jbodah/spy_rb) we now have many of those great features in Ruby.
