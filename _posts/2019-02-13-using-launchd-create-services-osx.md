---
layout: post
title: 'Using launchd to Create Services on OSX'
date: 2019-02-13 11:21:05 -0500
comments: true
categories:
---

If you're not familair with them, services are a critical part of running any non-trivial computer system.
For example, on Linux you have things like `systemd` which will ensure certain processes are running on boot,
reboot them if they crash, run them with the correct user, make sure the logs go somewhere reasonable, etc.
They are really great for going from "here's a program" to "here's something that feels more production-ready".

My dev machine - in anger - runs OSX (which, if you don't know, is based on BSD) so my process for trying to do cool
things usually ends if Homebrew doesn't have an equivalent for some Linux tool I want to use.
Recently I had built a [gitbot](https://github.com/jbodah/gitbot) to manage operations from my team's PRs.
For example, we can do things like write "@bot merge" and the bot will wait for our builds to pass and then will
merge the branch into master.
Our master branch requires an up-to-date passing build so it was becoming a very annoying chore to wait for
the build to pass only to find out you have to merge upstream back in and wait again because someone merged before
you.

Anyways, the way I solved this was by running a simple script that polls Github every minute and does its thing.
I was just running this on my local machine and starting it back up every time I opened my laptop for the day.
The tool became more popular and I started getting Slacks from people asking me to start the bot because I'd forgotten,
or it would get killed because my laptop went to sleep or whatever. Needless to say, this was annoying for everyone.

Know what would be great? If OSX would just manage the process being up for me. That's where services come in, and
the OSX way for running services is with `launchd`. Homebrew also does some `laucnhd` trickery to get things like your
MySQL server running on boot.

I started with [this tutorial](http://www.launchd.info/) which feels complete but verbose, so I figured I'd give you the lowdown of my solution.
My setup looks something like this:

```
# Directory structure
~/.env.yml                  # Contains secrets
~/bin/with_secrets          # Script that reads my secrets and injects them into ENV
~/repos/gitbot/bin/gitbot   # Starts my gitbot
```

So I run the following to start the bot: `with_secrets gitbot`

To have `launchd` manage the process for us we first need a manifest file.
Here's the one I used:


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>KeepAlive</key>
  <true/>

  <key>Label</key>
  <string>gitbot</string>

  <key>Program</key>
  <string>/Users/jbodah/bin/with_secrets</string>

  <key>ProgramArguments</key>
  <array>
    <string>/Users/jbodah/bin/with_secrets</string>
    <string>bash -lc "/Users/jbodah/repos/gitbot/bin/gitbot"</string>
  </array>

  <key>RunAtLoad</key>
  <true/>

  <key>StandardErrorPath</key>
  <string>/Users/jbodah/gitbot/stderr.log</string>

  <key>StandardOutPath</key>
  <string>/Users/jbodah/gitbot/stdout.log</string>

  <key>WorkingDirectory</key>
  <string>/Users/jbodah/repos/gitbot</string>
</dict>
</plist>
```

This file lives in `~/Library/LaunchAgents/gitbot.plist` (I symlink it in my `~/repos/gitbot` directory for convenience).
Most of this is pretty straightforward:
`KeepAlive` restarts the process if it dies.
`Label` is what we want to call the service.
`Program` is the program that will be passed to [exec(2)](https://docs.oracle.com/cd/E19455-01/806-0626/6j9vgh64s/index.html) and `ProgramArguments` is obviously the args passed.
One thing I found weird was that I had to specify the program again as `ProgramArguments[0]`, but such is life.
The spawned process will have access to these via argv.
I'm using `bash -lc "..."` to start a bash shell and login because I'm lazy and need to run some rbenv hooks that exist in my `.bashrc` to pull the correct Ruby version.
`RunAtLoad` is whether or not start the service on boot (or when the service is loaded into `launchd` which we'll get to in a second).
`StandardErrorPath` and `StandardOutPath` are for logs, and I put them somewhere that's easy for me to debug.
`WorkingDirectory` is the pwd for the process.

Now that we have the manifest file we need to load it into `launchd` using `launchctl`:

```
$ launchctl load -w ~/Library/LaunchAgents/gitbot.plist
```

And you can inspect that it's running:

```
$ launchctl list | grep gitbot
14148   1       gitbot
```

Thus, the `gitbot` service is running (hence the `1`) with pid `14148`.
If we kill the process we can see that `launchd` will reboot it:

```
$ kill 14148
$ launchctl list | grep gitbot
19757   -15     gitbot
```

Here the `-15` is the exit code of the last time the process exited (which makes sense since `-15` corresponds to `SIGTERM` which we sent with `kill`).

That's it! Happy... servicing... or something
