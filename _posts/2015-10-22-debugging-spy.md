---
layout: post
title: "Debugging With Spy"
date: 2015-10-22 15:15:11 -0400
comments: true
categories:
---

Today I had a piece of code that looked like this:

```rb
assert_equal 0, my_model.size

my_model.save

assert_equal 128, my_model.size
```

I wanted to see where `my_model.size` was being updated in the save call.
Sincde I was already in my console, my first approach was to use [Pry](https://github.com/pry/pry) and call `show-source my_model.save`:

```rb
def save
  run_callbacks :save do
    database.insert(self)
    @updated_at = Time.now
  end
  self
end
```

Great, so it's probably done in some callback.
Luckily with [Spy](https://github.com/jbodah/spy_rb) I can just intercept the call to `#save=`:

```rb
assert_equal 0, my_model.size

Spy.on(my_model, :size=).after { |*args| require 'rubygems'; require 'pry'; binding.pry }
my_model.save

assert_equal 128, my_model.size
```

Then run my test again

```rb
> pry-backtrace
/Users/Bodah/repos/my_repo/test/unit/concerns/my_model/content_test.rb:262:in `
/Users/Bodah/repos/my_repo/vendor/bundle/ruby/2.1.0/gems/spy_rb-0.5.0/lib/spy/i
/Users/Bodah/repos/my_repo/vendor/bundle/ruby/2.1.0/gems/spy_rb-0.5.0/lib/spy/i
/Users/Bodah/repos/my_repo/vendor/bundle/ruby/2.1.0/gems/spy_rb-0.5.0/lib/spy/i
/Users/Bodah/repos/my_repo/vendor/bundle/ruby/2.1.0/gems/spy_rb-0.5.0/lib/spy/i
/Users/Bodah/repos/my_repo/vendor/bundle/ruby/2.1.0/gems/spy_rb-0.5.0/lib/spy/i
/Users/Bodah/repos/my_repo/app/models/concerns/my_model/storage_common.rb:162:i
```

And there we have it in `storage_common.rb` line 162.

I tend to prefer Spy since it lets me do things like [conditional spying](https://github.com/jbodah/spy_rb/blob/master/lib/spy/instance.rb#L48),
but you can also do this with plain ol' Ruby:

```rb
assert_equal 0, my_model.size

my_model.define_singleton_method(:size=) { |*args| require 'rubygems'; require 'pry'; binding.pry }
my_model.save

assert_equal 128, my_model.size
```
