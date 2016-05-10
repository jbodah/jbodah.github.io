---
layout: post
title: 'Enumerator Goodness Even With Legacy Code'
date: 2016-05-09 15:26:56 -0400
comments: true
categories:
---

Our codebase has a lot of fairly old Ruby code that isn't quite up to snuff on the current best practices.
One area in which this is particularly true is in how we deal with blocks. It's not uncommon to see some code
like this in our codebase:

```rb
def find_each(row_id, start: nil)
  current = start
  loop do
    res = client.get_chunk(row_id, start: current)
    break if res.empty?
    yield res
    current = res.last
  end
end
```

This is a pretty common thing to see in Ruby. This code could be doing something like walking a Cassandra row.
Yielding records this way is really important so we don't blow up either our databases or our workers as it gives Ruby time to GC the results between yields.

Unforuntately however, this still isn't great.  Our jobs are more or less generic, so it makes more sense to instead
pass around different streams. One approach you could take is to duck-type on some method (say `:find_each`), but that
can still be pretty limiting and difficult to maintain as our application rapidly grows.
Also, what if we only want the first item of a stream or something? With the above approach we'd have to do something along the lines of:

```rb
def find_first(row_id)
  find_each(row_id) do |slice|
    return slice.first
  end
end
```

This is a border-line `GOTO` statement if you ask me. A better approach would be to use an `Enumerator`. In case you're not familiar with
using enumerators, I think it's easiest to just walk through an example:

```rb
def find_each(row_id, start: nil)
  Enumerator.new do |yielder|
    current = start
    loop do
      res = client.get_chunk(row_id, start: current)
      break if res.empty?
      yielder.yield res
      current = res.last
    end
  end
end
```

`Enumerator.new` takes a block that yields a `yielder`. This is just an object that you can call `yield` on (if you're curious, Ruby
uses `Fibers` under the hood to switch execution contexts). Just like above, you can simply pass it the thing you want to `yield`.

Why do we do this? This makes our stream much more flexible and allows us to pass it around as an object and tack things on (and
make it `lazy` which I won't cover here but you should play around with). Here's what our `find_first` looks like now:

```rb
def find_first(row_id)
  find_each(row_id).first
end
```

Can't get much simpler than that. However we still have one more slight problem. Our API which worked with blocks before is now broken:

```rb
find_each(row_id) { |slice| puts slice }
#=> Doesn't work! :(
```

Digging through the Rails codebase I found a few examples of the use of `to_enum` (or `enum_for`, they are aliased). This method is
cool because it lets you write your methods with just `yield` but then allows you to wrap them in enumerators very easily.
Implementing with it looks like:

```rb
def find_each(row_id, start: nil)
  return to_enum(:find_each, row_id, start: start) unless block_given?

  current = start
  loop do
    res = client.get_chunk(row_id, start: current)
    break if res.empty?
    yield res
    current = res.last
  end
end
```

This first checks if a block is given to the method. If so, continue and use it. If not, create an `Enumerator` using this method.
This is equivalent to wrapping the rest of the body in the `Enumerator.new {...}` syntax we used previously. The first argument to `to_enum`
is the method name, and the rest are arguments that will get passed to that method. The end result of this approach is a solution that has
all of the conveniences of both the `Enumerator` and the direct `yield` approach:

```rb
def find_first(row_id)
  find_each(row_id).first
end

find_each(row_id) { |slice| puts slice }
#=> will print each slice
```

`to_enum` can also be used to help me with our legacy code; I don't have to go around touching a bunch of old code to have enumerators if
I don't want to. I can simply use `to_enum` with the first implementation:

```rb
to_enum(:find_each, row_id).first
```

Enumerators are one of the most powerful Ruby tools out there.
They can help you write code that is not only elegant and flexible but also highly performant (e.g. batching requests to reduce bandwidth).
Moreover, you can do all of this transparently without the consumer of your Enumerator having to know the details of how the records are fetched.
Ultimately it's well worth the investment to take the time to master them.
