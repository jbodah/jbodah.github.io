---
layout: post
title: 'Implementing Graceful Exits in Elixir'
date: 2017-05-18 14:49:04 -0400
comments: true
categories:
---

I work on our video encoding pipeline which is built as a collection of Elixir services. Unlike a traditional
Erlang distributed system which might connect multiple BEAM instances (i.e. nodes) together and have processes
communicate over Erlang's custom protocols, we've instead chosen to use HTTP to communicate between our services.
This decision fits better with our infrastructure, allows us to transfer knowledge from other applications (such
as nginx), and ensures our services won't become too coupled should we ever need to write a replacement.

At Wistia we run our Elixir services in Kubernetes. Kubernetes is a container orchestration
system which spins containers up and down based on traffic and in certain cases such as when we're deploying.
Kubernetes tears down containers (more properly "pods") by [sending them OS signals](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods).
Kubernetes will send a `SIGTERM`, wait for some grace period for the container to shutdown, and then finally send a `SIGKIll` if the container is still up.

By default the BEAM will exit immediately whenever it receives a `SIGTERM`. This might not be a problem in
many applications, but in our platform we want to give running jobs a chance to complete before the system
shuts down. [Andrew Djoga wrote a nifty gist](https://gist.github.com/Djo/bfa9fa75928ce432ec51) which essentially
wraps the BEAM processes, traps signals, and handles them by instead spinning up another BEAM node which connects to our
main node and calls `:init.stop`. `:init` is the origin process in the Erlang runtime (similar to UNIX `init`) and stopping
it will cause the entire node to shutdown.

For the rest of this post I'll make use of [a small demo app](https://github.com/jbodah/shutdown_app). I put the contents of the
gist in `beam_wrapper.sh`. You can start the app using `./beam_wrapper.sh`. For reference for those of you on mobile or the like,
here's the source for the demo app:

```elixir
defmodule ShutdownApp do
  use Application
  import Supervisor.Spec

  def start(_type, _args) do
    IO.inspect "#{__MODULE__}.start"

    children = [
      worker(ShutdownApp.Worker, [])
    ]

    Supervisor.start_link(children, strategy: :one_for_one)
  end

  def stop(_) do
    IO.inspect "#{__MODULE__}.stop"
  end
end

defmodule ShutdownApp.Worker do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, [])
  end

  def init(_) do
    IO.inspect "#{__MODULE__}.init"
    Process.send_after(self(), :ping, 2000)
    {:ok, {}}
  end

  def handle_info(:ping, state) do
    IO.inspect "#{__MODULE__}.ping"
    Process.send_after(self(), :ping, 2000)
    {:noreply, state}
  end

  def terminate(_, _) do
    IO.inspect :terminating
    Process.sleep(1_000_000)
  end
end
```

In this app we want `ShutdownApp.Worker` to exit gracefully. Conventionally the `terminate` callback is used by OTP-compliant
modules (like `GenServer`) to specify teardown logic. [The docs](https://hexdocs.pm/elixir/GenServer.html#c:terminate/2)
tell us that `terminate` will only be called if the process is trapping exit signals (which it will *not* do be default).
We'll flag the `ShutdownApp.Worker` process in its `init` callback to trap exits:

```elixir
def init(_) do
  Process.flag(:trap_exit, true)
  {:ok, {}}
end
```

This will turn the exit signal into a message which we can act on. Note that the `EXIT` message will get enqueued in our mailbox just like
any other message and will be handled only after we process (or ignore) the other messages in our mailbox that are before it.
`GenServer` already implements a `receive` block which will delegate to the `terminate` callback. In the demo app we simply print a message and sleep.

```elixir
def terminate(_, _) do
  IO.inspect :terminating
  Process.sleep(1_000_000)
end
```

Now we can test our app by starting it with `./beam_wrapper.sh` and then sending it a signal with the UNIX command `kill`:

```sh
$ ./beam_wrapper.sh
+ trap term_handler TERM INT
+ PID=77484
+ echo 'Started the server process with PID 77484'
Started the server process with PID 77484
+ wait 77484
+ elixir --name app@127.0.0.1 -S mix run --no-halt
Compiling 1 file (.ex)
Generated shutdown_app app
"Elixir.ShutdownApp.start"
"Elixir.ShutdownApp.Worker.init"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"

$ kill -s TERM 77484

"Elixir.ShutdownApp.Worker.terminate"
"Elixir.ShutdownApp.stop"
+ trap - TERM INT
+ wait 77484
+ EXIT_STATUS=0
+ exit 0
```

Now if you've been paying close attention you'll notice `ShutdownApp.Worker` was killed before the `Process.sleep` finished. This is because supervisors start
with a default shutdown time of 5000ms. We can override this by specifying a different shutdown time. In our environment we plan to let Kubernetes hard-kill our
app, so any long timeout should work. I like to follow the advice of using very large numbers instead of `:infinity`, so I arbitrarily picked `123_456` instead:

```elixir
defmodule ShutdownApp do
  # ...

  def start(_type, _args) do
    # ...

    children = [
      worker(ShutdownApp.Worker, [], shutdown: 123_456)
    ]

    Supervisor.start_link(children, strategy: :one_for_one)
  end
```

And finally when we re-run our verification test we see that the app waits longer for the Worker to exit cleanly:

```sh
$ ./beam_wrapper.sh
+ trap term_handler TERM INT
+ PID=44942
+ echo 'Started the server process with PID 44942'
Started the server process with PID 44942
+ wait 44942
+ elixir --name app@127.0.0.1 -S mix run --no-halt
"Elixir.ShutdownApp.start"
"Elixir.ShutdownApp.Worker.init"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"
"Elixir.ShutdownApp.Worker.ping"

$ kill -s TERM 44942
"Elixir.ShutdownApp.Worker.terminate"
```

So there we have it. Now we are handling OS signals, and we can bide our system more time to let expensive jobs finish before we tear the system down.
