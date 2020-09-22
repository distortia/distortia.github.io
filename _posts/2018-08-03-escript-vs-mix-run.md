---
title:  "Elixir EScripts vs Mix Run"
date:   2018-08-03
---

Continuing on my last post as a write about a new service I am working on at work, I'd like to talk about [EScripts](https://hexdocs.pm/mix/master/Mix.Tasks.Escript.Build.html).

What are EScripts? They are Elixir scripts in the form of an executable that you compile and run.

# What are EScripts used for?

This is a great question! EScripts are compiled with all the Elixir code needed to run. Which means it'll run on any machine that has Erlang installed. This means you can make an portable executable as a CLI(Command Line Interface).

If you are making a CLI application, such as a custom scaffolding based on inputs from the CLI, then EScripts are wonderful.

# What are EScripts not used for?

While having a CLI can be a great thing, it does not make sense to port service code into EScripts. Service code, such as database seeding, migrating data, etc.

Phoenix does a great job at demonstrating this with it's [seed files](https://phoenixframework.org/blog/seeding-data).

EScripts are best suited for creating CLIs as stated in their docs. I have seen many use EScripts for what they should be using `mix run my_script.exs` for. Maybe the docs of Elixir is too great and they came across EScripts. Either way, I wanted to shed some light and help educate those that may be tempted to use an EScript.

# My experience in using an EScript instead of `mix run`

In an attempt to get into the weeds of the situation, I created an EScript to read data from a file, parse/transform, and load it into my database.

Normally, I would have this as an `.exs` file or Elixir Script inside a `bin` folder. I would invoke the file as `MIX_ENV=my_env mix run bin/my_script.exs`. I have different databases for `dev` and `prod`. I know that all my prod/dev environments all have Elixir on them, so I do not have to worry about shipping an all-inclusive erlang executable.

With each EScript it needs to be compiled for the environment that it needs to run in. This is kind of tedious. Running `MIX_ENV=prod mix escript.build` and not remembering that last time you compile your script can bite you sometimes. I accidentally ran, what I thought was a `prod` EScript, in `dev`. I was confused for a little bit as to why my data was not in my `prod` database. Then it hit me, I compiled my EScript for `dev`, not `prod`

With `mix run`, I have to explicitly set my `MIX_ENV` to `prod` in order to run the script on my `prod` instance. There is a bit more verbosity with this setup and I feel that the process keeps you from making the mistake of running the right script in the wrong environment.

# Conclusion

For most of your needs you should always pick `mix run` over EScripts. Both can be covered under your code coverage tool of choice. Both can be tested in the same manor. `mix run` is more verbose in it's usage and for me, that's a huge plus.