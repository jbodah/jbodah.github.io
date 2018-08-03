---
layout: post
title: 'Writing Simpler Programs with GNU Parallel'
date: 2018-05-18 13:54:34 -0400
comments: true
categories:
---

[parallel](GNU://www.gnu.org/software/parallel/) is an excellent little program, and I wanted to highlight some practical examples of how useful it is.
For this example I'll explain some real problems I was facing at work as well as some how and why.

I've been experimenting using MongoDB as the primary datastore in replatforming our analytics platform.
MongoDB has a bunch of features that would greatly simplify our existing analytics code and under the hood works similar
to how our existing platform works (e.g. shards + leader-follower replication) while also providing very nice tools for introducing nodes.
While the reasons for choosing Mongo over other solutions are beyond the scope of this post, this will act as the backdrop for our problem.
Specifically we want to bootstrap Mongo with a lot of test data. The main point is that we're doing *something* that is going to be bottlenecked by IO.

In this particular case I started with a pipeline that looked like the following:

```
cat *.log | ./preprocess_events | ./import
```

Logs are line-separated JSON events. I pulled these from our archives and gunzip them in batches of around 120M events.
`preprocess_events` is a simple Ruby program that reads lines from STDIN, normalizes the fields, and renames the event session key to `_id` which we'll use with Mongo.
These events are JSON encoded again and written to STDOUT.
Finally `import` is another small Ruby program which reads JSON encoded events from STDIN, batches them up into batches of 10000 insert statements and uses Mongo's bulk write
feature to persist them in batches.
(NOTE: I was previously using `mongoimport` and storing raw events but it became clear pretty quickly that that data model wasn't going to scale so we now have a data model that relies
on upserting and thus can't use `mongoimport`)

This initial pipeline worked just fine however `import` was clearly a bottleneck doing something like the following:

```
$stdin.each_line.each_slice(10000) do |lines|
  records = lines.map { |line| JSON.parse(line) }
  upserts = records.map { |record| to_upsert(record) }
  collection.bulk_write(upserts)
end
```

This code pulls in the 10k lines from STDIN, forms the batch, and pushes them to Mongo; rinse-repeat until EOF.
Since IO was the problem (you could just watch STDOUT and figure it out) my first thought was to use threads.
Ruby implements green threading which basically means you can run code concurrently however the Ruby process will not run threads in parallel.
Since any properly built IO library will be implemented asynchronously you can generally have multiple parallel requests in-flight
and just use threads to manage them (kind of similar to how an event loop would work).

The threaded attempt looked something like the following:

```
queue = Queue.new

workers =
  10.times.map do
    Thread.new do
      connection = ...new mongo connection...
      loop do
        work = queue.deq
        break if work == :stop
        connection.bulk_write(work)
      end
    end
  end

$stdin.each_line.each_slice(10000) do |lines|
  records = lines.map { |line| JSON.parse(line) }
  upserts = records.map { |record| to_upsert(record) }
  puts "warning! high queue depth #{queue.size}" if queue.size > 500000
  queue.enq(upserts)
end

10.times { queue.push(:stop) }
workers.map(&:join)
```

Here we're creating a `Queue` which is a thread-safe queue (NOTE: while Ruby only executes one thread at a time the point at which the VM's scheduler context switches provides no guarantees to our code
and depends heavily on how the instruction set gets parsed from our code).
`Queue` uses locking internally to provide us with atomic operations which we *can* rely on.
After the queue declaration we spin up a pool of ten workers that just read from the queue until they are told to shutdown.
We read everything out of STDIN like before but instead push all the upserts to the queue which one of our workers will pick up
when they are ready.
Note that we don't have any backpressure mechanisms in place so we need to be careful not to blow up memory by enqueuing too much data.
Here just printing a warning seems sufficient (we can bump our worker count if necessary).
After reading all of the messages we push a `:stop` message for each worker (the worker will break out of its read loop when it picks up the `:stop` message
so we can rely on ten messages stopping all the workers).
Finally we wait for the workers to clean up by waiting for the threads to stop.

Our output was a bit more complicated after this but one thing was for sure - we were definitely inserting a lot more stuff (the logs for importing to Mongo were interspersed in STDOUT by each thread).

Finding the next bottleneck was a bit trickier.
First I used `strace` to see what the programs were doing.
When you build a pipeline like `a | b | c` what you're really doing is spawning three processes `a`, `b`, and `c` and connecting them with "pipes".
Pipes are an OS abstraction that let our processes pass data between each other.
One interesting property of pipes is that they buffer data.
When these buffers are full then calls to `write` will be blocked to prevent buffer overflows which would result in data loss and potentially crashing.
We can inspect this with strace.
I grabbed the pids using `ps aux` and then did something along the lines of `strace -p 123` for each process.
`cat` isn't doing nearly as much work as our other processes and that is evident by it pausing with messages like:

```
write(
```

This means that
I grabbed the pids using `ps aux` and then did something along the lines of `strace -p 123` for each process.
`cat` isn't doing nearly as much work as our other processes and that is evident by it pausing with messages like:

```
write(
```
