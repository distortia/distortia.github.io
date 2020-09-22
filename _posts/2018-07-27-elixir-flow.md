---
title:  "Using Flow in Elixir to Easily Parallelize your Data Parsing"
date:   2018-07-27
---

I know its been a long time since I wrote. I had a few posts in the pipeline but got caught up with work.

Today's post will be quick and focused around [Flow](https://github.com/elixir-lang/flow), Elixir's Computational parallel flows on top of GenStage. 

Flow was written by Elixir's creator, José Valim. José announced and demoed Flow/GenStage as part of his keynote at [Elixir Conf 2016](https://youtu.be/srtMWzyqdp8?t=244).

I have been slowly introducing my company to the power of Elixir through various applications and services that I built. One of these applications is my user agent parser, Suavé. Suavé takes in a weeks worth of user agents from a text file, parses and transforms them before inserting them into my database. These files contain anywhere from 300k - 450k+ rows. My Database is currently sitting on around 6.5 million records. 

I built Suavé to inform our test lead which operating systems and browsers pose the highest risks. This leads to less duplicated efforts in testing and a more risk-based approach. So we can save time, energy, and money while having a higher confidence.

### The Problem

As working through my Suavé MVP, I had to work with speed and get the product out into the hands of those who need the tool. As many of you know, this can lead to some unoptimized code. This is okay, as long as we tackle it in the future. We know what the bottlenecks are and how we can best resolve them.

My bottleneck was parsing this gigantic text file and loading all the data into the database. From early on, we chose to run the importer as an `escript`. Escripts are elixir scripts that are compiled and live as an executable in your project. For more info on Escripts check out the [Escript Docs](https://hexdocs.pm/mix/master/Mix.Tasks.Escript.Build.html)

The main parser looks similar to this:

```elixir
  def insert_user_agents(file_name) do
    File.stream!(file_name, [], :line)
    |> Stream.chunk(1)
    |> Enum.each(fn chunk ->
      chunk
      |> List.first()
      |> UserAgentParser.parse()
      |> DateParser.parse()
      |> Repo.insert!()
    end)
  end
```

We take in a `file_name` and stream the file, so we don't keep all the file in memory, take single chunks from the file, parse the `user_agent`, `date` and `insert` into the database. All really straightforward.

The problem is this took upwards to 30 mins on my virtual machine

```
Operating System: Linux"
CPU Information: Intel Core Processor (Haswell)
Number of Available Cores: 4
Available memory: 23.55 GB
Elixir 1.6.5
Erlang 20.3
```

### The Solution

I toyed around with doing my own parallelization of parsing and inserting, but I did not want to reinvent the wheel. That's where `Flow` came in.

With `Flow`, I can split up my chunks into parallal computations and do everything I was doing, but faster!

Here is my `Flow` code:

```elixir
 def insert_user_agents(file_name) do
    file_name
    |> File.stream!()
    |> Flow.from_enumerable()
    |> Flow.partition()
    |> Flow.map(fn item -> 
      item
      |> UserAgentParser.parse()
      |> DateParser.parse()
      |> Repo.insert()
    end)
    |> Flow.run()
  end
```

We are calling the same methods for parsing the `user_agent` and `date`, but now you use methods like`Flow.map()` in place of the normal `Enum.map()`. Flow has the concept of `partitions`, `windows`, and `triggers`. Right now we only care about the `partition`. 

With `Flow's` `paritions` it automatically chunks my data into a default of _500_ items per `partition`. We no longer have to implement our own parallel computations. 

This new `Flow` implementation takes 3-4 minutes, down from 30! This is a huge performance bump for little refactoring.

### Conclusion

If you are doing processing/parsing/transformation of data, give `Flow` a shot. As always, benchmark the before and after, you can find some surprising differences using options such as `:max_demand`, and `:min_demand`.

### Links

* [Flow](https://github.com/elixir-lang/flow)
* [José Valim's Keynote from ElixirConf 2016](https://youtu.be/srtMWzyqdp8?t=244)
* [Escripts](https://hexdocs.pm/mix/master/Mix.Tasks.Escript.Build.html)