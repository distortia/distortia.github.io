---
title:  "Overcommit Your Feature Files!"
date:   2017-08-11
---

You ever have the issue where someone checks in some bad Gherkin?

Maybe some things like a missing `|` on a scenario outline table or the like?

Well how do you stop those changes from getting into your repo? [Overcommit!](https://github.com/brigade/overcommit)

# What is Overcommit?

From the [Overcommit repo](https://github.com/brigade/overcommit): 

> A fully configurable and extendable Git hook manager

With Overcommit, we can validate our code before it ever gets into the repo. But there's a catch, it only checks code that you are changing now! This means that you can effectively stop the bleeding of your repo, and move towards a better state.

# Installation and Setup of Overcommit

Setting up Overcommit is pretty simple, as taken from their [readme](https://github.com/brigade/overcommit#installation):

`gem install overcommit`

`cd my-repo/`

`overcommit --install`

From this, overcommit will install git hooks into your `.git` folder and there will be a file called `.overcommit.yml`

_NOTE_: If you already have git hooks, you should know that Overcommit will move your hooks into a directory at `.git/hooks/old-hooks/`. You can simply copy them back manually or have a script to do it for you. This helps when you want to roll this out to the rest of the team.

# Creating a Custom Overcommit Hook

Inside the `.overcommit.yml` file we can replace the contents of the file with thie following:

```yaml
PreCommit:
  GherkinValidator:
      enabled: true
      description: 'Makes sure that gherkin isnt broken'
      include: '**/*.feature'
```

Next we sign the overcommit changes, this allows for our changes to be picked up.

`overcommit --sign` 

What we are doing is creating a PreCommit hook called `GherkinValidator`. The `include: '**/*.feature'` option is telling Overcommit to only run this hook when feature files get updated.

Now that Overcommit knows we have a hook, we need create the hook. Let's do that now.

`mkdir .git-hooks`

`mkdir .git-hooks/pre_commit`

`touch .git-hooks/pre_commit/gherkin_validator.rb`

Open up the `gherkin_validator.rb` now. 

Place the following inside:

```ruby
module Overcommit
  module Hook
    module PreCommit
      # Gherkin Validator that makes sure all the gherkin is parsable and doesnt cause issues
      class GherkinValidator < Base
        def run
          puts 'Starting Gherking Validation'
          status = system('bundle', 'exec', 'cucumber', '-t', '@gherkin_validation')
          puts 'Gherkin Validated' if status
          status ? :pass : :fail
        end
      end
    end
  end
end
```

Our hook resides inside the `Overcommit::Hook:PreCommit` module. What we are doing is using `Cucumber` to parse our feature files and blow up if theres a problem. We accomplish this by trying to run a test that shouldn't exist with that tag. If you are using `@gherkin_validation` where you work, let me know, cause thats kinda interesting.

Here's what it looks like when it fails

```
λ git add features\.
λ git commit -m "undo me"
features/my_test.feature: Parser errors:
(193:7): inconsistent cell count within the table (Cucumber::Core::Gherkin::ParseError)
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core/gherkin/parser.rb:30:in `rescue in document'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core/gherkin/parser.rb:25:in `document'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core.rb:27:in `block in parse'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core.rb:26:in `each'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core.rb:26:in `parse'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-core-1.5.0/lib/cucumber/core.rb:18:in `compile'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-2.4.0/lib/cucumber/runtime.rb:67:in `run!'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-2.4.0/lib/cucumber/cli/main.rb:32:in `execute!'
/bin/Ruby23/lib/ruby/gems/2.3.0/gems/cucumber-2.4.0/bin/cucumber:8:in `<top (required)>'
/bin/Ruby23/bin/cucumber:22:in `load'
/bin/Ruby23/bin/cucumber:22:in `<main>'
Running pre-commit hooks
Starting Gherking Validation
Makes sure that gherkin isnt broken................[GherkinValidator] FAILED
````

And when you fix that `|` that someone left off of a outline:

```
λ git add features\.
λ git commit -m "Fixed broken gherkin"
Using the default and chrome profiles...
0 scenarios
0 steps
0m0.000s
Running pre-commit hooks
Starting Gherking Validation
Gherkin Validated
Makes sure that gherkin isnt broken................[GherkinValidator] OK
```

Sweet! Now we can stop people from checking in bad Gherkin.

# Conclusion

With our new precommit hook in place, we can elevate ourselves and repos. Spread the word to your friends and colleagues. The possibilities with Overcommit are pretty powerful and you can rest easier at night knowing that someone isn't pushing broken code.

My Gitlab Repo for the post - [https://gitlab.com/distortia/overcommit_setup](https://gitlab.com/distortia/overcommit_setup)

Happy Hacking!

Nick