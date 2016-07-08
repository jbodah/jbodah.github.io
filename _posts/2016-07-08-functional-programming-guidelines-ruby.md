---
layout: post
title: 'Functional Programming Guidelines in Ruby'
date: 2016-07-08 11:29:56 -0400
comments: true
categories:
---

1. Use case equality (`#===`) for psuedo-pattern matching

2. Implement `#call`

3. Use stateless singleton methods

```rb
module Writer
  class << self
    def write(writable)
      # ...
    end
  end
end
```

4. Learn to use `#to_proc`

```rb
[1, 2, 3].map(&Writer.method(:write))
```

5. Make the critical arguments the last ones so you can use `#curry`

```rb
module Writer
  class << self
    def write(io, writable)
      # ...
    end
  end
end

[1, 2, 3].map(&Writer.method(:write).curry[$stdout])
```
