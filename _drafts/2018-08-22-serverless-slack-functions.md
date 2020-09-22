---
title:  "Serverless Slack Functions - Google Cloud"
date:   2018-08-22
---

I've been meaning to write this post for a while now. As an enhancement to one of my previous posts [Elixir Slackbot on Heroku](https://nickstalter.com/elixir-slackbot-on-heroku.html), I wanted to see what [Google Cloud Functions](https://cloud.google.com/functions/) meant for my slack bots and commands.

As of this time of writing, Google only allows a small number of languages to be deployed as cloud functions. That list is:

* JavaScript

* Python

[Source:](https://cloud.google.com/functions/docs/writing/)

This means that we cannot use Elixir as our function language. Pretty disappointed.

We have a few options with this though, we can use a library like [ElixirScript](https://github.com/elixirscript/elixirscript) or just use plain JavaScript.

How about we try both?!

# Getting Started

To get started, we will set up a Slack App. 

Head to [https://api.slack.com/apps?new_app=1](Slack's API Website). Fill in your information and select your organization.

From here, we want do add some features and functionality. Go ahead and select "slash commands". We want to `Create new Command`. 

For our command, we are going to make a dance command. It will look like the following.