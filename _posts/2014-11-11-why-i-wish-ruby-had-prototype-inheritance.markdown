---
layout: post
title: "Why I Wish Ruby Had Prototype Inheritance"
date: 2014-11-11
comments: true
categories: ruby
---

I was working on converting some legacy code the other day which looked something like this:

```ruby
SomeModel.find_by_id(service_id)
```

SomeModel here includes ActiveModel. It was recently patched to support a UUID as we transition from service_id to UUID:

```ruby
SomeModel.find_by_id(service_id, uuid)

def SomeModel.find_by_id(service_id, uuid = nil)
  current_uuid = uuid || find_uuid(service_id)
  super(current_uuid || service_id)
end
```

This supports finding by either service_id or UUID in our Redis system. And it works for our transition period, but the API makes no sense. A better approach would be to have something like:

```ruby
uuid ? SomeModel.find_by_uuid(uuid) : SomeModel.find_by_id(service_id)

def SomeModel.find_by_id(service_id)
  uuid = find_uuid(service_id)
  uuid ? find_by_uuid(uuid) : super(service_id)
end

def SomeModel.find_by_uuid(uuid)
  # Call superclass find_by_id with uuid
end
```

The problem here is that I can't reference `super` from the `find_by_uuid` method. If I had some way to reference the superclass, then I could just forward the message to it, but I don't think Ruby gives that to me.

If we had prototype inheritance I could just delegate to my prototype:

```ruby
def SomeModel.find_by_uuid(uuid)
  self.proto.find_by_id(uuid)
end
```

I bet I can probably do some hacking with aliases, but it's just not worth it. Keeping the old nasty interface sadly feels like the best option to me, and it's an artifact of Ruby's classical inheritance.

EDIT:

I posted something on Stack Overflow [asking this here](http://stackoverflow.com/questions/26870232/ruby-directly-send-superclass-a-message)

EDIT:

It turns out aliasing does the trick (see Stack Overflow example)
