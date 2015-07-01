---
layout: post
title: "Making Life Easier with TerminalNotifier"
date: 2015-06-17 10:51:00 -0400
comments: true
categories: ruby, osx
---

Anyone who knows me knows I'm a sucker for anything that streamlines my development
process. What's better than typing 10 characters? Typing 9. What's better than doing
things manually? Not having to worry about them at all.

One of the things that has dramatically improved my development process has been
creating a CLI on top of our deploy scripts to manage things like best practices.
Our vanilla deploy process looks something like this:

```sh
# Add my deploy to our deploy queue

# Mark my deploy as started

# Update my /etc/hosts
RUBBER_ENV=production bundle exec cap rubber:setup_local_aliases

# Give sudo privilege

# Make sure I can ping the servers
# Make sure I'm connected to VPN/have correct deploy secret
RUBBER_ENV=production bundle exec cap invoke COMMAND='echo hello' FILTER_ROLES=app,worker

# Actually run the deploy
# - Split it up so I don't take down the whole system
# - Don't deploy to "no deploy" hosts
RUBBER_ENV=production bundle exec cap deploy FILTER_ROLES=worker,-no_deploy
RUBBER_ENV=production bundle exec cap deploy FILTER_ROLES=app,-no_deploy

# Check the deploy worked

# Clear my deploy to allow the next person to deploy
```

The CLI streamlines a lot of this process to the following:

```sh
# Add my deploy to our deploy queue

# Mark my deploy as started

# Actually deploy
# - No need to sync /etc/hosts, ping hosts, segment deploy, or worry about "no deploy"
# - Way shorter to type
deploy app,worker

# Give sudo access

# See that ping actually worked. Confirm y/n to continue with deploy

# Check that deploy worked

# Clear my deploy to allow the next person to deploy
```

It's been great however I'm notoriously irresponsible and often forget about my deploys.
They're so easy to kick off now that I just can do it then go do something else while the deploy runs.
Luckily I found out about the fantastic OSX gem [terminal-notifier](https://github.com/alloy/terminal-notifier).

Why is it so great? Now instead of switching over to my deploy tab every few minutes I can just
be notified when a deploy is done:

```rb
require 'terminal-notifier'

# shortened for brevity

run_deploy(roles)
TerminalNotifier.notify("Deploy Finished to #{roles}")
```

This displays a sleek message in the top of my right screen telling me that it's time to verify my deploy so
I can stop holding up the deploy queue. `#notify` also accepts some options too in case you're really zoning out:

You can add sounds:

```rb
TerminalNotifier.notify("Deploy Finished. Verify it, knucklehead!", sound: 'default')
```

Or you can change it so the notification opens up a program:

```rb
# cat /Applications/Firefox.app/Contents/Info.plist | grep BundleIdentifier -A 1
TerminalNotifier.notify("Deploy Finisihed. Verify it, knucklehead!", activate: 'org.mozilla.firefox')
```
