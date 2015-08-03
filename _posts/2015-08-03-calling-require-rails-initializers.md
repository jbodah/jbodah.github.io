---
layout: post
title: "Calling require From Rails Initializers"
date: 2015-08-03 14:43:19 -0400
comments: true
categories:
---

I recently booted up my `rails c` and was greeted with a pleasant warning message:

```
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:6: warning: already initialized constant Backupify::Redis::Server::REDIS_SHARED_ROLE
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:6: warning: previous definition of REDIS_SHARED_ROLE was here
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:9: warning: already initialized constant Backupify::Redis::Server::MULTI_REDIS_ROLE
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:9: warning: previous definition of MULTI_REDIS_ROLE was here
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:12: warning: already initialized constant Backupify::Redis::Server::DEFAULT_REDIS_CONFIGURATION
/Users/Bodah/repos/backupify/backupify/lib/backupify/redis/server.rb:12: warning: previous definition of DEFAULT_REDIS_CONFIGURATION was here
```

Well that's a fun error. This type of thing is usually benign, but it's still annoying and not really what you **want** to happen.
What the error is telling us is that we're loading the file `lib/backupify/redis/server.rb` twice.

Turns out that the culprit was one of our initializers which was calling `require 'backupify/redis/server'`.
We also have eager loading enabled in our development environment.
Rails runs the initializers first and **then** eager loads [as documented here](https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb#L37).

Rails does some monkey patching shenanigans to `require` and sets up some non-trivial global state.
I'm not entirely sure why it's so complex, but it winds up handling whether or not a piece of code should be loaded with native `load` or native `require`.
Regardless, it's pretty obvious that calling `require` on code that will eventually be eager loaded before the eager loading kicks in causes some issues.

If you follow the trail of crumbs for eager loading you'll eventually come to [Rails::Engine::eager_load!](https://github.com/rails/rails/blob/master/railties/lib/rails/engine.rb#L475)
which calls `require_dependency` under the hood.
This seems to setup the global state such that eager loading knows to not try and call `load` on the code again.

So just change those `require 'backupify/redis/server'` statements to `require_dependency 'backupify/redis/server'` and ahhh... no more warnings.
