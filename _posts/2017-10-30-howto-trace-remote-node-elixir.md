---
layout: post
title: 'How to Trace a Remote Node in Elixir'
date: 2017-10-30 19:51:15 -0400
comments: true
categories:
---

At Wistia our video encoding service runs on Elixir. We run Elixir inside
of Kubernetes and use a wrapper script (which we've adapted from @[bitwalker](https://github.com/bitwalker)) to name the BEAM node:

```bash
# beam_wrapper.sh
#!/usr/bin/env bash
set -x

term_handler() {
  echo "Stopping the server process with PID $PID"
  erl -noshell -hidden -sname "term" -eval "rpc:call('app@localhost', init, stop, [])" -s init stop
  echo "Stopped"
}

trap 'term_handler' TERM INT

elixir --name app@localhost -S mix phoenix.server --no-halt &
PID=$!

echo "Started the server process with PID $PID"
wait $PID

# remove the trap if the first signal received or 'mix run' stopped for some reason
trap - TERM INT

# return the exit status of the 'mix phoenix server'
wait $PID
EXIT_STATUS=$?

exit $EXIT_STATUS
```

Naming the node allows us to connect to it via another node. Typically we will "SSH" into the Kubernetes pod we're interested in
with `kubectl exec` and then use the following script to start an `IEx` session inside the pod:

```sh
# remsh
#! /usr/bin/env sh
iex --hidden --sname console --remsh app@localhost
```

This lets us interact with any named processes and inspect running ones. It lets us change configuration on the fly and send
messages to our live processes. We've also been able to shutdown certain processes to ease maintenance and get to the bottom of
tricky issues.

We don't maintain a lot of state in our app, but the places that we do are critical for us and can be impossible to create outside
of trail and error. I was recently working on an issue where one particular video was having some issues. I was able to reproduce
the issue in our production environment with each try and wanted to dig-in to the problems the video was having.

Normally we rely on some form of logging to debug issues. Whether that be printing the state between each `GenServer` call, perusing
our log output in Scalyr, or simple `IO.puts` to figure out which blocks are being called. These are all pretty generic ways of doing
"puts" debugging and gold standards, but they all require the app to be recompiled. Unfortunately that would be redeploying our app
and losing our precious state.

Cue `dbg`, Erlang's gold standard for debugging. Tracing - in short - it lets you specify a module, function, and arity that
you're interested in, a set of processes that you're interested in, and a "match spec" (which does things like let you specify
whether you want to log all of the calls or if you want to log all of the return values) and logs all the matches (usually calls) to
that function.

Here's an example:

```ex
:dbg.tracer
```
