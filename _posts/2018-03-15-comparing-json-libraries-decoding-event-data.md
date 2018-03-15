---
layout: post
title: 'Comparing Elixir JSON Libraries for Decoding Event Data'
date: 2018-03-15 11:38:34 -0400
comments: true
categories:
---

I was recently working on building an application which ingests JSON-encoded event data
and was running into some bottlenecks. My first bottleneck stemmed from a `Logger.debug`
statement that wasn't being compiled out. After tackling that I wanted to compare various libraries
for performing JSON decoding. We've used `poison` for most of our apps and it has worked pretty
well however there's been some discussion about other libraries and I wanted to run tests with
real data.

I compared every library for both Erlang and Elixir I could find on Github that did JSON decoding
and was hosted on hex. Here is my wrapper application:

```
defmodule JsonBm do
  def run do
    stream = IO.stream(:stdio, :line)
    {uptime, _} = :erlang.statistics(:wall_clock)
    state = {0, uptime + 10_000}
    Enum.reduce(stream, state, fn line, {seen_in_period, next_flush} ->
      decode(json)
      {uptime, _} = :erlang.statistics(:wall_clock)
      if uptime >= next_flush do
        IO.puts "throughput = #{seen_in_period / 10} lines/sec"
        {1, uptime + 10_000}
      else
        {seen_in_period + 1, next_flush}
      end
    end)
  end

  defp decode(json) do
    Poison.decode(json)
  end
end
```

To test the code I'm running `cat test.log | mix run --no-halt -e 'JsonBm.run'`. `test.log` is a file of
line-separated JSON-encoded events. Events are fairly small with around 10 keys with values of integers, strings,
and short arrays of other small JSON objects (who each have ~3 keys and have primitive data types for values)

And here are the results. I wanted to let each approach run for about a minute to take advantage of any caching
the library might provide. I also included a run without any JSON decoding as a base benchmark. Lastly I
compiled the best performing libraries with HiPE support to see what impact that had (note: you can compile
libraries with HiPE support using `ERL_COMPILER_OPTIONS="native" mix deps.compile`):

```
...no json decoding...
throughput = 38170.9 lines/sec
throughput = 49968.4 lines/sec
throughput = 49164.3 lines/sec
throughput = 48691.4 lines/sec
throughput = 49093.1 lines/sec
throughput = 49840.3 lines/sec
throughput = 50581.5 lines/sec
throughput = 50858.5 lines/sec
throughput = 49499.6 lines/sec

{:json, "1.0.2"}
throughput = 9396.7 lines/sec
throughput = 9152.2 lines/sec
throughput = 10033.3 lines/sec
throughput = 10735.7 lines/sec
throughput = 10333.1 lines/sec
throughput = 10684.1 lines/sec
throughput = 10590.0 lines/sec
throughput = 10528.5 lines/sec
throughput = 10572.7 lines/sec
throughput = 10637.0 lines/sec

{:poison, "3.1.0"}
throughput = 12127.0 lines/sec
throughput = 16455.0 lines/sec
throughput = 17176.3 lines/sec
throughput = 16944.2 lines/sec
throughput = 16561.1 lines/sec
throughput = 17004.2 lines/sec
throughput = 16941.9 lines/sec
throughput = 16903.5 lines/sec
throughput = 16464.6 lines/sec
throughput = 16914.6 lines/sec
throughput = 17393.8 lines/sec

{:jason, "1.0.0"}
throughput = 18331.0 lines/sec
throughput = 24303.5 lines/sec
throughput = 25544.2 lines/sec
throughput = 25518.5 lines/sec
throughput = 26096.4 lines/sec
throughput = 25829.8 lines/sec
throughput = 25727.8 lines/sec
throughput = 25852.5 lines/sec
throughput = 25818.5 lines/sec
throughput = 24682.9 lines/sec
throughput = 24414.6 lines/sec

{:jason, "1.0.0"} + hipe
throughput = 24207.5 lines/sec
throughput = 32055.9 lines/sec
throughput = 36146.8 lines/sec
throughput = 36091.8 lines/sec
throughput = 36496.0 lines/sec
throughput = 36972.4 lines/sec
throughput = 37008.9 lines/sec
throughput = 36738.1 lines/sec

{:exjsx, "4.0.0"}
throughput = 11275.1 lines/sec
throughput = 14466.1 lines/sec
throughput = 14522.0 lines/sec
throughput = 14540.4 lines/sec
throughput = 14286.5 lines/sec
throughput = 14539.7 lines/sec
throughput = 14561.6 lines/sec
throughput = 14309.8 lines/sec
throughput = 14533.1 lines/sec
throughput = 14532.9 lines/sec
throughput = 14046.5 lines/sec
throughput = 14444.5 lines/sec

{:jiffy, "0.15.1"}
throughput = 28312.8 lines/sec
throughput = 32938.7 lines/sec
throughput = 37906.4 lines/sec
throughput = 36750.0 lines/sec
throughput = 36617.8 lines/sec
throughput = 39002.7 lines/sec
throughput = 38869.3 lines/sec
throughput = 35317.3 lines/sec
throughput = 35023.4 lines/sec
throughput = 37812.3 lines/sec
throughput = 37732.1 lines/sec

{:jiffy, "0.15.1"} + hipe
throughput = 29167.3 lines/sec
throughput = 33164.6 lines/sec
throughput = 37887.4 lines/sec
throughput = 37182.7 lines/sec
throughput = 38157.6 lines/sec
throughput = 38082.7 lines/sec
throughput = 39079.3 lines/sec
throughput = 37734.1 lines/sec
throughput = 34797.6 lines/sec
throughput = 38699.7 lines/sec
throughput = 38904.2 lines/sec
throughput = 38497.6 lines/sec
throughput = 38694.1 lines/sec
throughput = 37583.5 lines/sec
throughput = 36774.7 lines/sec

{:jsx, "2.9.0"}
throughput = 12834.2 lines/sec
throughput = 16685.8 lines/sec
throughput = 15337.4 lines/sec
throughput = 15533.9 lines/sec
throughput = 15310.9 lines/sec
throughput = 15671.6 lines/sec
throughput = 15515.1 lines/sec
throughput = 16326.9 lines/sec
throughput = 15799.3 lines/sec
throughput = 15969.3 lines/sec

{:jsone, "1.4.5"}
throughput = 16873.7 lines/sec
throughput = 20276.5 lines/sec
throughput = 19119.5 lines/sec
throughput = 18956.6 lines/sec
throughput = 20040.1 lines/sec
throughput = 20028.4 lines/sec
throughput = 19830.4 lines/sec
throughput = 18045.4 lines/sec
throughput = 19290.0 lines/sec
throughput = 20027.7 lines/sec
```

So to conclude, `:jiffy` seemed to be the fastest, but `:jason` was very close. Both performed better when compiled with HiPE.
I think for our use case we'll probably go with `:jason` because it will be easier to work with Elixir-friendly data types,
but if raw performance is what you need then it seems like `:jiffy` is hard to beat.
