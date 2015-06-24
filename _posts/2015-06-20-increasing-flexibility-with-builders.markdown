---
layout: post
title: "Increasing Flexibility with Builders"
date: 2015-06-20 10:17:05 -0400
comments: true
categories:
---

This is one of the patterns we use a lot in our code base:

```rb
class Service < ActiveRecord::Base
  belongs_to :customer

  def self.search_class
    Search
  end
end

class GoogleMailService < Service
  def self.search_class
    ImapSearch
  end

  # ...
end

class GoogleDocsService < Service
  # ...
end

class Search
  def initialize(search_args = {})
    # ...
  end

  def perform
    # ...
  end
end

class ImapSearch
  def initialize(search_args = {})
    # ...
  end

  def perform
    # ...
  end
end

class Jobs::DeepSearch
  def initialize(service_id, search_args = {})
    @service_id = service_id
    @search_args = search_args
  end

  def perform
    service = Service.find(@service_id)
    search = service.search_class.new(@search_args)
    search.perform
  end
end
```

There are several issues with this paradigm.
For one, `Service` is tightly coupled with the `Search` class.
This might not seem like a big deal here, but in our codebase `Service` has
a lot of other responsibilities too. For example, it also declares things like
`::restore_class` and `::backup_class` in addition to all of the traditional
things we'd expect for an `ActiveRecord` model.

Another downside is that we have to override `::search_class` for specific subclasses
which is a code smell. This mean each new `Service` we create needs to override each
of these classes for their particular use case.
In practice, this has led us to leaking a lot of domain concepts into our abstractions
(e.g. we might have `GoogleMailService` and `GoogleMailBackup`).
This style of design is not generic and doesn't scale well to a large number of use cases since
we need to have a separate class for each one.

A third issue lies in how we're instantiating `service.search_class`.
This implementation assumes two things.
it assumes we want to instantiate an object every time we want to perform a search,
and it assumes that the constructors for each of those takes a `Hash`.
This means we wouldn't be able to take a more functional approach if we wanted to e.g.:

```rb
module ImapSearch
  def self.perform(service, search_args = {})
    # ...
  end
end
```

It also means we have to deal with all of the issues that come with using `Hash` parameters
such as not having a convenient way to throw `ArgumentError` when we don't receive everything
we've expected. `Hash` parameters make it a lot more difficult to spot errors when you do
something as simple as rename an argument (note: if you aren't using Ruby's keyword arguments
then you really should [check them out](https://robots.thoughtbot.com/ruby-2-keyword-arguments)).

So, in summary, these are the issues with the original approach:
  1. `Service` is overloaded with the responsibility of defining the `Search` strategy
  2. Business rules for selecting `Search` strategy are obscured by inheritance
  3. Directly instantiating the `Search` implementation doesn't provide flexibility during construction

Luckily, we can use a Builder to fix this. Rather than link to some obscure UML diagram, let's just
think of what a Builder is conceptually. A Builder is a thing that takes a bunch of input, makes decisions
on that input to constructs an object for some task, and returns that object. For example, we might use a
`SearchBuilder` which would take in a `service_id` and a bunch of `search_args` and it will return to us
something that responds to `#perform`. This returned object is abstractly our `Search` strategy and we can
simply rely on ducktyping to use this object (note: the pattern I described actually be more of a Factory
pattern, but semantics-shemantics - that's not the point. The point is using good OO design, not worrying
about using the correct labels)

Here's an example of what we might have after some refactoring:


```rb
class Service < ActiveRecord::Base
  belongs_to :customer
end

class GoogleMailService < Service
  # ...
end

class GoogleDocsService < Service
  # ...
end

class SearchBuilder
  def self.build(service_id, search_args = {})
    service = Service.find(service_id)
    case service
    when GoogleMailService
      ImapSearch.new(search_args)
    else
      Search.new(search_args)
    end
  end
end

class Jobs::DeepSearch
  def initialize(service_id, search_args = {})
    @service_id = service_id
    @search_args = search_args
  end

  def perform
    service = Service.find(@service_id)
    search = SearchBuilder.build(service_id, search_args)
    search.perform
  end
end
```

This solves our first issue; `Service` is no longer responsible for determing the `Search` strategy.
Instead we've inverted the dependency to create a new object whose responsibility is determining the
`Search` strategy.

You might argue that `SearchBuilder` now has a direct dependency on the `Service` subclasses.
This is a problem indeed as it violates [open/closed principle](https://en.wikipedia.org/wiki/Open/closed_principle);
if we wanted to add `GoogleContactsService` then we'd have to modify `SearchBuilder::build`.
One way around this is to change the construction rules to instead depend on the interfaces that `Service` provides.
We might change it to something like:

```rb
class SearchBuilder
  def self.build(service_id, search_args = {})
    service = Service.find(service_id)
    if service.data_type == :imap
      ImapSearch.new(search_args)
    else
      Search.new(search_args)
    end
  end
end
```

This also solves the second issue we had.
Instead of relying on a web of distributed logic due to inheritance, this approach does is centralizes
the creation of `Search` strategies. Everything is nicely encapsulated in one place and it's fairly easy
to reason about.

Say we wanted to use our more functional approach of using a singleton now.
`SearchBuilder::build` makes it simple for us to add because we've now abstracted the construction
of the `Search` strategy away from `Jobs::DeepSearch`. We still have one issue though: we'd need to wrap
our singleton `ImapSearch` in an adapter of some sort so that we can defer the `perform` call to whenever
`Jobs::DeepSearch` wants to make it. My recommendation is to instead rename `perform` to `call`.
`call` is ducktyped for lambda and procs allowing us to easily defer a method call whenever we want.
After this refactoring, we have something like this:

```rb
class Search
  def initialize(search_args = {})
    # ...
  end

  def perform
    # ...
  end
end

module ImapSearch
  def self.perform(service, search_args = {})
    # ...
  end
end

class SearchBuilder
  def self.build(service_id, search_args = {})
    service = Service.find(service_id)
    case service
    when GoogleMailService
      Proc.new { ImapSearch.perform(service, search_args) }
    else
      Proc.new { Search.new(search_args).perform }
    end
  end
end

class Jobs::DeepSearch
  def initialize(service_id, search_args = {})
    @service_id = service_id
    @search_args = search_args
  end

  def perform
    service = Service.find(@service_id)
    search = SearchBuilder.build(service_id, search_args)
    search.call
  end
end
```

`SearchBuilder::Build` is starting to look a little ugly, so let's clean it up with a little syntactic sugar.
We'll use `-> {...}` instead of `Proc.new` as it's a bit cleaner.
We'll also use `method()` instead of `Proc.new` as it's more clear and both `Proc` and `Method` respond to `#call`:

```rb
class SearchBuilder
  def self.build(service_id, search_args = {})
    service = Service.find(service_id)
    case service
    when GoogleMailService
      -> { ImapSearch.perform(service, search_args) }
    else
      Search.new(search_args).method(:perform)
    end
  end
end
```

This approach starts to break down a bit if we want our `Search` strategies to do more than just respond to `#call`,
but it shows how ducktyping on `#call` gives us a lot more flexibility in our design by allowing us to avoid object
instantiation entirely if we wanted.

So to recap, here are the changes we made:
  1. Move responsibility of determining `Search` strategy out of our monolithic `Service` class and into a `SearchBuilder`
  2. Encapsulate the strategy instance creation in the `SearchBuilder` to allow the `Search` strategies to be implemented
     in whatever way is most convenient for the strategy writer
  3. Consider ducktyping on built-in conventions to simplify the `SearchBuilder` logic

Hopefully you found something there helpful. I'd love to hear other opinions on how you might refactor this code.
