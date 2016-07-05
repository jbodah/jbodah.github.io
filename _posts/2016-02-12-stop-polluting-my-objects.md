---
layout: post
title: 'Stop Polluting My Objects'
date: 2016-02-12 10:52:45 -0500
comments: true
categories:
---

Stop adding inheritable helper methods to mixins. Take the following example:

```rb
class MyVehicle
  def has_wheels?

  end
end

module WheelCounter
  def print_status
    pp status
  end
end
```

Mixins. I'm sure many of us have a love-hate relationship with them.
In fact, I should just call out inheritance in general, but mixins are the particular culprits I want to focus on.
