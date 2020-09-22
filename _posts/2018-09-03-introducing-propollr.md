---
title:  "Introducing ProPollr.com"
date:   2018-09-03
---

Hey everyone!

I have been hacking away on an idea for a week and finished it! 

Introducing [ProPollr.com](propollr.com)!

# What is ProPollr?

ProPollr is a realtime polling application that focuses on ease of use, realtime interactivity, and polling anonymity! 

I came up with the idea as I was walking passed a group that was holding a retro. They were writing all of their responses onto sticky notes and walking up to the whiteboard to place them. That's when it hit me. We know that anonymous feedback is honest feedback, so let's capitalize on that!

# What's the Tech Stack?

This should come as no surprise, I am using

* [Elixir](https://elixir-lang.org/)
* [Phoenix](https://phoenixframework.org/) - I'll touch on this in a minute.
* [Bulma](https://bulma.io/) - With Node-Sass for Brunch
* [Font Awesome 5 Pro](https://fontawesome.com/) - Awesome Icons!
* [Veil](https://github.com/zanderxyz/veil) - I'll touch on this in a minute as well.

### Phoenix

[Phoenix](https://phoenixframework.org/) is a wonderful framework for composing web applications and API's. Some of the amazing features that comes with Phoenix, are Channels and Presence.

[Channels](https://hexdocs.pm/phoenix/channels.html) are a means to send and receive messages from sockets. There are a myriad of functionalities built in that allow you to incorporate realtime aspects with minimal code. With Channels, you can perform controller actions and broadcast to the specfied socket, you can listen for specific messages and reply with changes to state. An awesome use of sockets is the [Drab Project](https://github.com/grych/drab).

I use sockets for realtime question and answers.

[Presence](https://hexdocs.pm/phoenix/presence.html) is a newer feature of Phoenix that revolves around, well, presences. It's biggest highlight is the ability to track and register process informatatin across the cluster. Presence has no single point of failure and no single source of truth. This is massive! 

The main reason for me to use it, is to display the current number of Pollrs in a Sesh(a question session).

### Veil

[Veil](https://github.com/zanderxyz/veil) is a wonderful passwordless authentication library. It is much easier to not have to worry about storing/salting/hashing/protecting passwords if you never check for one. This also makes the user experience that much simpler. The action for logging in is the same as creating an account. You get an email sent to your inbox and you click the link to authenticate. I find is very simple and easy to work with. Authentication is then handled to an encoded cookie that automatically gets checked and its `TTL(Time to Live)` extended.

# So What is the Plan?

Propollr is open for use now. 

My next steps, in no particular order:

* Full Launch
* Add extra features such as graphs and charts
* Extra question categorization - So you can run things like retros
* Setup CI/CD using [Distillery](https://github.com/bitwalker/distillery)

# Conclusion

For me, the biggest win is the ability to take an idea and push something out in a week. It's a wonderful feeling to release something new and already have a viable beta group to use it.

Feel free to check it out and submit some feedback!


### Links

* [ProPollr.com](propollr.com)
* [Elixir](https://elixir-lang.org/)
* [Phoenix](https://phoenixframework.org/)
* [Channel](https://hexdocs.pm/phoenix/channels.html)
* [Presence](https://hexdocs.pm/phoenix/presence.html)
* [Bulma](https://bulma.io/)
* [Drab](https://github.com/grych/drab)
* [Veil](https://github.com/zanderxyz/veil)
* [Distillery](https://github.com/bitwalker/distillery)