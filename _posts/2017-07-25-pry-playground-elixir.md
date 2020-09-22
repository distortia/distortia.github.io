---
title:  "Pry-like Playground in Elixir"
date:   2017-07-25
---

Hey everyone! It's been a while, but I am back with a quick post for helping us Elixir devs out there.

Coming from the Ruby world, many of us have come to depend on the awesome gem by the name of [Pry](https://github.com/pry/pry). `Pry`'s name to fame is allowing you to drop into your running code at a given breakpoint. This is extremely helpful when triaging tests and inspecting code on the fly. One of the neat things about `Pry`, is that is allows access to your entire environment. 

# What does that have to do with Elixir?

Great question! The Elixir team has already incorporated `Pry` like capabilities into [IEx](https://hexdocs.pm/iex/IEx.Pry.html#content) and you can read more about it from this [Plataformatec blog post](http://blog.plataformatec.com.br/2016/04/debugging-techniques-in-elixir-lang/). I am not really here to help you use `Pry`, but I am here to help you get to a pry like state when running `IEx`.

I have been using `IEx` a lot during the development of a project at work. Coming in and out of `IEx` sessions has led to me to retyping a bunch of my aliases for all my models/controllers/modules/etc. This can become really tedious! After doing some digging I came across the `.iex.exs` file. This is a config file that alows us to configure our `iex` experience. Which means we can tailor it to our needs!

# My elevated developer experience

When we run `iex -S mix`, it will read from our `.iex.exs` file and load the code. Which means we can do our aliasing inside this file. We are opting for a local file, because each one of our repos will probably need different configurations.

So for our case, we can create a `.iex.exs` file in the root of our project:

`touch .iex.exs`

Then proceed to alias the things we want. I found a neat way to do multiple imports on one line from the [Elixir Getting Started Docs](https://elixir-lang.org/getting-started/alias-require-and-import.html#multi-aliasimportrequireuse)

`alias MyApp.{Module1, Module2, Controller1, MyModel}`

When we load up `iex -S mix`, we are dropped into our environment like normal, but now the things we care about are aliased!

# Local vs Global Config Files

In this huge blob of text [in the documentation](https://hexdocs.pm/iex/master/IEx.html#module-exiting-the-shell), you can see what I am talking about:

> When starting, IEx looks for a local .iex.exs file (located in the current working directory), then a global one (located at ~/.iex.exs) and loads the first one it finds (if any).

So it's important to remember that you can cause havoc if you have a local and a global `.iex.exs` file. I would probably stick with only local config files.

# Word of Warning

In Elixir, we like our namespacing. But we also like our development experience. There's a small compromise if we have modules that share the same name as an already aliased module. I've personally ran into naming collisions around `Elixir.Node`, which has a different context than `MyApp.Node` but calling `Node.connect` is referrring to `MyApp.Node.connect`. So just be sure to please use responsibly!

As always! Happy Hacking!

Nick