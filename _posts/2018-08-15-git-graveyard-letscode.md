---
title:  "Git Graveyard: Letscode - 2014"
date:   2018-08-18
---

Welcome back to another post about my Git Graveyard. Today I want to look at a CLI(Command-line Interface) utility I worked on.


# Let's Code - CLI Utility for Opening Up Project Specific Programs - 2014

Let's Code was intended to be a CLI that would open up specific programs that you need to get working on your project.

You would use it like `letscode myproject`. Let's Code would then open your editor with your project, Trello/Jira, and can be configured to open up another other programs you need, like Slack.

# Motivation

I was motivated to make getting into work easier. I was constantly switching between projects and various programs in order to do my work. I wanted to help my context switching out while trying to remain in the terminal as best I could. 

The usage was pretty straightforward:

```ruby
require 'thor'

module Letscode
    class CLI < Thor
        include Thor::Actions
        include Letscode

        #main method
        desc "start [PROJECT]", "To start a project, type 'letscode start [PROJECT]'"
        def start(project)
          get_project(project)
          process_keys
        end

        desc "config", "To config a project, please type 'letscode config'"
        def config
            config = get_config
            write_file config
        end

        desc "list", "Shows all projects in the config file."
        def list
           puts  "#{list_projects.to_yaml}"
        end

        desc "delete [PROJECT]", " To delete a file type 'letscode delete [PROJECT].'"
        def delete(project)
            config = read_file
            config.delete project
            write_file config
        end
    end
end

```

The various methods went out and did what you expect. Just follow up with a `letscode myproject` and you were ready to roll!

# Reason for Abandoning

I abandoned this project as I switched jobs and had less context switching over all. As I reread the code I wrote and the direction it was going, I feel this project would have been a fun little utility.

# Tech Stack

* Ruby - I was getting more into rails at the time and wanted something familiar to work in.

* [Thor](https://github.com/erikhuda/thor) - toolkit for building powerful command-line interfaces in Ruby

* [Watir](http://watir.com/) - A gem typically used for browser automation(mostly for testing)

* [ChromeDriver](http://chromedriver.chromium.org/) - chromium's webdriver implementation for browser automation

# Conclusion

All in all, I may want to revisit this utility. I have quite a bit of use for it as of late. The project I am working on has different repos for it's umbrella apps and I have to switch between projects fairly regularly. As far as looking at my code, I am pleasantly surpised with past me. I managed to keep things coherent and readible. 

I may end up porting this over to Elixir, it's one of my favorite languages and has strong built in CLI Utilities with [OptionParser](https://hexdocs.pm/elixir/OptionParser.html)
