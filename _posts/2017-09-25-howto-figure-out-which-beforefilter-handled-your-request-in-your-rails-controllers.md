---
layout: post
title: 'How to figure out which before_filter handled your request in your Rails controllers'
date: 2017-09-25 09:53:22 -0400
comments: true
categories:
---

`before_filter` is a common way to do things like authentication and authorization in Rails controllers.
It's a great way to separate and reuse code throughout your controllers.
For example, you might want to make sure that the current user is an admin before showing them
some page or else you may want to redirect them somewhere.

While structuring your controller validations like this can lead a cleaner separation of concerns,
there's no doubt that it makes things a lot more confusing when you hit some route on your localhost
just to find that your controller action isn't actually being called like you thought.
You look at the top of the file and see ten `before_filter` calls and your eyes start to glaze
over as you realize that the next 45 minutes of your day will involve mind-numbingly stepping through
each of these before filters trying to figure out which one handled the request.

Well, those days are (thankfully) over. `before_filter` is implemented using `ActiveSupport::Callbacks`,
and the fine folks who developed this library were sweet enough to already think about our app developer pains.

Let's dig in a little to make sure we're all on the same page before I just give you the answer though.
When you define a callback, `ActiveSupport::Callbacks` more or less stashes the callback somewhere and
compiles your callbacks into a "chain" of callbacks when necessary. So concretely

```rb
before_filter :make_sure_logged_in
before_filter :make_sure_admin
```

winds up becoming something like:

```rb
if make_sure_logged_in && make_sure_admin
  process_action
end
```

`ActiveSupport::Callbacks` is, of course, a lot more complicated, generic, and feature rich than this, but that is the gist
of what it does.

One neat feature of `ActiveSupport::Callbacks` is the ability to "halt" a callback chain.
That is, the ability to stop processing the callback chain mid-way through the processing.
You've probably seen this mentioned before if you've been doing Rails for awhile by returning `false` in one of your callbacks
(DISCLAIMER: I am not up to date on edge Rails practices, but I don't believe Rails 5 works exactly this way anymore so check your facts if
you want to use the callback halting feature)

Anyways, I told you the ActiveSupport team already anticipated this pain for us? Here's the money:
You can implement the instance method `halted_callback_hook(filter)` in your controller class (or any other class that includes `ActiveSupport::Callbacks`)
and it will be called with the name of the filter that halted the chain. No more littering your code with `binding.pry`!

Here's an example from my own debugging session. I simply implemented `halted_callback_filter` on my `SessionsController`, threw a pry and there, and presto - the
filter that was redirecting me:

```rb
Frame number: 0/111

From: /Users/jbodah/repos/my_app/app/controllers/sessions_controller.rb @ line 64 SessionsController#halted_callback_hook:

    62: def halted_callback_hook(filter)
    63:   require 'rubygems'; require 'pry'; binding.pry
 => 64: end

[1] pry(#<SessionsController>)> filter
=> :send_them_on_their_way_if_logged_in
```

If you found this post helpful, have any questions, or know of any other tips that people might find useful, feel free to shoot them in the comments.
Thanks for reading!
