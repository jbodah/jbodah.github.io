---
layout: post
title: 'How Supervisors Work'
date: 2016-11-18 14:19:01 -0500
comments: true
categories:
---

In Erlang (and Elixir) supervisors are processes which manage child processes and
restart them when they crash. In this post we're going to take a look at the details
of how supervisors are implemented. I had a rough idea of how these they worked, but I
didn't understand the specifics. I felt like learning some stuff and figured I'd share it
with you <3.

For this dive it would be helpful if you understand how to use both the `gen_server` and `supervisor` modules before.
If you've used the Elixir equivalent then those are fine too as they just delegate down to the Erlang
modules and don't really change behaviorally.

Let's start with an example in Elixir, straight from the docs:

```ex
defmodule MyApp.Supervisor do
  use Supervisor

  def start_link do
    Supervisor.start_link(__MODULE__, [])
  end

  def init([]) do
    children = [
      worker(Stack, [[:hello]])
    ]

    # supervise/2 is imported from Supervisor.Spec
    supervise(children, strategy: :one_for_one)
  end
end
```

In this example, `start_link` is spawning the supervisor and then `init` is a callback
used by the `Supervisor` behaviour. Let's dig into `Supervisor`. As a quick recap of behaviours,
the `use Supervisor` call will expand at compile time to whatever is in that behaviour's `__using__` macro.
So, let's look at `__using__` in the `Supervisor` module in the Elixir source (tangent: [the Elixir source](https://github.com/elixir-lang/elixir)
is laid out very conventionally and I recommend you take a little time to get comfortable navigating it).

Here's the `__using__` macro at the time of writing this:

```ex
defmacro __using__(_) do
  quote location: :keep do
    @behaviour Supervisor
    import Supervisor.Spec
  end
end
```

`@behaviour` is doing some checks to make sure we implement the necessary callbacks for our supervisor and
the `import` statement is pulling in some extra methods into `MyApp.Supervisor` from `Supervisor.Spec`.
This is where the `worker` and `supervise` methods are both defined.

That's the gist of what the `use Supervisor` statement is doing; mixing in some functions and making sure we
implement the right callbacks.

The way we'd start our supervisor is by calling `MyApp.Supervisor.start_link`,
so let's dig into that. Obviously, this delegates to `Supervisor.start_link` passing a reference to itself (via `__MODULE__`).

Checking out the source for `Supervisor.start_link`:

```ex
def start_link(module, arg, options \\ []) when is_list(options) do
  case Keyword.get(options, :name) do
    nil ->
      :supervisor.start_link(module, arg)
    atom when is_atom(atom) ->
      :supervisor.start_link({:local, atom}, module, arg)
    {:global, _term} = tuple ->
      :supervisor.start_link(tuple, module, arg)
    {:via, via_module, _term} = tuple when is_atom(via_module) ->
      :supervisor.start_link(tuple, module, arg)
    other ->
      raise ArgumentError, """
      expected :name option to be one of:
        * nil
        * atom
        * {:global, term}
        * {:via, module, term}
      Got: #{inspect(other)}
      """
  end
end
```

The main thing we see here is that the Elixir module is delegating down to the Erlang `:supervisor` module. Wait, don't run
screaming! You can follow [the Erlang source](https://github.com/erlang/otp), trust me :)

Grepping for `start_link` you'll find an `export` statement which is just exposing it outside the module, a `spec` which is
just telling you what types the function expects, and the actual implementation:

```erlang
start_link(Mod, Args) ->
    gen_server:start_link(supervisor, {self, Mod, Args}, []).
```

See? Already something familiar. We're just starting a `gen_server`. We won't dig into [how gen_server works](http://erlang.org/doc/design_principles/gen_server_concepts.html).
The main takeaway is that supervisors are built on-top of `gen_server`. `gen_server` expects us to implement a bunch of callbacks.
The one we're most concerned with right now is `init`. Note that even though `MyApp.Supervisor` implements `init` it is *not* the callback that will be called next.
If you look back at `start_link` in the Erlang `:supervisor` module you'll see that it passes the `self` reference meaning that `:supervisor.init` is the function we're looking for next.

Here's the source for that:

```erlang
init({SupName, Mod, Args}) ->
    process_flag(trap_exit, true),
    case Mod:init(Args) of
        {ok, {SupFlags, StartSpec}} ->
            case init_state(SupName, SupFlags, Mod, Args) of
                {ok, State} when ?is_simple(State) ->
                    init_dynamic(State, StartSpec);
                {ok, State} ->
                    init_children(State, StartSpec);
                Error ->
                    {stop, {supervisor_data, Error}}
                  end;
        ignore ->
            ignore;
        Error ->
            {stop, {bad_return, {Mod, init, Error}}}
end.
```

This is doing a few things. First, it's calling `Mod:init(Args)` which is just calling `MyApp.Supervisor.init`. Let's look at that again real quickly:

```ex
def init([]) do
  children = [
    worker(Stack, [[:hello]])
  ]

  # supervise/2 is imported from Supervisor.Spec
  supervise(children, strategy: :one_for_one)
end
```

Remember, `worker` and `supervise` are helpers coming from `Supervisor.Spec`. Without digging through the plumbing, I'll cut to the chase so we can focus more
on the Erlang side of things. `worker` outputs a [child_spec](http://erlang.org/doc/man/supervisor.html#type-child_spec) and `supervise` outputs a tuple looking
like `{:ok, { {strategy, max_retries, max_seconds}, child_specs} }`.

Back to `:supervisor.init`. Next, very importantly, it calls `process_flag` which will [trap exits](http://erlang.org/doc/reference_manual/processes.html#errors).
This is very important and central to how the supervisor knows when to restart a process. The TL;DR is that when a process terminates it sends an
exit signal to all of its linked processes. Calling `process_flag` will trap that signal and instead send the `{'EXIT', from_pid, reason}` message
to that process instead. As we'll see later, the supervisor process will use that `from_pid` value to know which process died and how to
restart it.

Okay, we left off in `:supervisor,init` and just started trapping exit signals. Next we initialize our state and our children. I won't go into `init_state`.
Let's go into `init_children` since we're not dealing with `:simple_one_for_one` supervisors.

```erlang
init_children(State, StartSpec) ->
    SupName = State#state.name,
    case check_startspec(StartSpec) of
        {ok, Children} ->
            case start_children(Children, SupName) of
                {ok, NChildren} ->
                    {ok, State#state{children = NChildren}};
                {error, NChildren, Reason} ->
                    _ = terminate_children(NChildren, SupName),
                    {stop, {shutdown, Reason}}
            end;
        Error ->
            {stop, {start_spec, Error}}
end.
```

Oh boy, more symbols. Another quick tangent: `State#state.name` is accessing the variable `State` as a `state` record and plucking off the `name` field.
[Records](http://erlang.org/doc/reference_manual/records.html) are more-or-less structs that are stored as ordered tuples like `{:state, "josh", [1, 2, 3]}` (kind of like an enum type in other languages).
Records are just a way to decouple the position of a field from its meaning. Here's the source for the `state` record defined at the top of the file:

```erlang
-record(state, {name,
                strategy               :: strategy() | 'undefined',
                children = []          :: [child_rec()],
                dynamics               :: {'dict', ?DICT(pid(), list())}
                                        | {'set', ?SET(pid())}
                                        | 'undefined',
                intensity              :: non_neg_integer() | 'undefined',
                period                 :: pos_integer() | 'undefined',
                restarts = [],
                dynamic_restarts = 0   :: non_neg_integer(),
                module,
                args}).
```

As you can see, `state` records have a `name` field, so `SupName = State#state.name` is just treating the `State` tuple as a `state` record, plucking out whatever
field corresponds to `name`, and saving that in `SupName`.

Glancing over `check_startspec`; this is doing some validation as well as casting the spec we received from `MyApp.Supervisor.init` to a record ([source](https://github.com/erlang/otp/blob/2a56d0ed91c1c5e18008d1cf37406f36b46b4e62/lib/stdlib/src/supervisor.erl#L1346))

The real meat of `:supervisor.init_children` is `start_children` though:

```erlang
start_children(Children, SupName) -> start_children(Children, [], SupName).

start_children([Child|Chs], NChildren, SupName) ->
    case do_start_child(SupName, Child) of
        {ok, undefined} when Child#child.restart_type =:= temporary ->
            start_children(Chs, NChildren, SupName);
        {ok, Pid} ->
            start_children(Chs, [Child#child{pid = Pid}|NChildren], SupName);
        {ok, Pid, _Extra} ->
            start_children(Chs, [Child#child{pid = Pid}|NChildren], SupName);
        {error, Reason} ->
            report_error(start_error, Reason, Child, SupName),
            {error, lists:reverse(Chs) ++ [Child | NChildren],
            {failed_to_start_child,Child#child.name,Reason}}
    end;

start_children([], NChildren, _SupName) ->
    {ok, NChildren}.
```

Here we have a little recursion, plucking off each child and calling `do_start_child`:

```erlang
do_start_child(SupName, Child) ->
    #child{mfargs = {M, F, Args}} = Child,
    case catch apply(M, F, Args) of
        {ok, Pid} when is_pid(Pid) ->
            NChild = Child#child{pid = Pid},
            report_progress(NChild, SupName),
            {ok, Pid};
        {ok, Pid, Extra} when is_pid(Pid) ->
            NChild = Child#child{pid = Pid},
            report_progress(NChild, SupName),
            {ok, Pid, Extra};
        ignore ->
            {ok, undefined};
        {error, What} -> {error, What};
        What -> {error, What}
end.
```

`apply` is Erlang's dynamic function invocation method (similar to `send` in Ruby or `apply`/`call` in Javascript). This winds up dynamically calling
the callback defined in `Supervisor.Spec.worker`. [Conventionally, it calls start_link on your worker module](https://github.com/elixir-lang/elixir/blob/f8f4731f5ea32b6ee2896fcda5ed901b28fbaca7/lib/elixir/lib/supervisor/spec.ex#L240).
Finally, it calls `report_progress` which uses Erlang's unfortunately named [error_logger](http://erlang.org/doc/man/error_logger.html) to publish an info event
that the new process was started by the supervisor.

First breathe. Okay, let's continue.

Zooming way out, this is how a supervisor starts, traps exits, and starts children. We still haven't gotten to the real beef of how supervisors
restart their children, but we're really close!
Remember how `process_flag` traps exit signals and turns them into `{'EXIT', from_pid, reason}` tuples?
Also remember that supervisors are built on top of `gen_server`?
Well, `gen_server` handles all non-call/cast messages using `handle_info` ([further reading on handle_info](http://erlang.org/doc/man/gen_server.html#Module:handle_info-2)).
This is how supervisors handle exits from a child!

```erlang
handle_info({'EXIT', Pid, Reason}, State) ->
    case restart_child(Pid, Reason, State) of
        {ok, State1} ->
            {noreply, State1};
        {shutdown, State1} ->
            {stop, shutdown, State1}
    end;
```

Here we see that it handles the exit message which contains the pid of the child process that exited and the reason and it feeds that into `restart_child`!
Victory! We've already seen how `do_start_child` works, and `restart_child` is fairly similar, so I'll leave that for the curious to look up on their own.
If you're interested in how supervisors implement their shutdown strategies, take a peek at [:gen_server.stop](http://erlang.org/doc/man/gen_server.html#stop-1)
which delegates to the [supervisor's terminate callback](https://github.com/erlang/otp/blob/2a56d0ed91c1c5e18008d1cf37406f36b46b4e62/lib/stdlib/src/supervisor.erl#L644).

So, let's sum it all up!
Elixir implements a conventional framework for writing supervisors.
That framework interfaces with Erlang's `:supervisor` module which is built on top of `:gen_server`.
Elixir passes configs down to the `:supervisor` module which it uses to start child processes.
It manages the child processes by trapping exit signals from its children to convert the signal
into a message.
It implements the `handle_info` callback which can handle the exit messages and restart the correct worker.
Finally, it's worth reiterating that it publishes reports to `:error_logger`, Erlang's event manager.

Anyways, I learned a ton from this dig, and I hope you did too.
If you found this post useful or have ways you think it could be clearer, then please let me know!
Thanks for reading!
