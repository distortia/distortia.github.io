---
title:  "Elixir Cucumber Browser Automation Testing"
date:   2016-10-18
---

Hey Everyone, moving forward with my Elixir and Test Automation experience, I've been working on some kind of new POC. I've been following two Elixir projects: [White-Bread](https://github.com/meadsteve/white-bread) and [Hound](https://github.com/HashNuke/hound). White-Bread is a Cucumber parser built in Elixir. While Hound is a Browser Automation framework, also written in Elixir. You can see where I am going now.

### The Problem

My problem doesn't really stem from a frustration or lack of interest in the typical Ruby+Cucumber+Selenium/Watir/Page Object automation stack; but from all the extra effort and resources spent trying to make Ruby do things in parallel. We have more than one core our computers now a day, we should be able to use those out of the box.

### The Idea

My idea is to combine White-Bread and Hound into a Cucumber driven Browser automation suite. Those that are familiar with ruby gems/frameworks as [Watir](https://github.com/watir/watir), its similar to that. 

### Motivation

My main motivation behind doing this is to leverage the power of the Elixir language. I want to be able to run all my tests concurrently and without having to worry about scale. This means we can write as many tests as we want and not have to wait for a super long feedback loop. With every test as it's own process, we don't have to worry about a single test failing and causing subsequent crashes. We can even have a supervisor restart that process and attempt to self-heal. 

### So how does this thing work?

The premise is to have White-Bread's Cucumber parsing drive Hound's browser automation capabilities. Think of it as dropping RSpec, Capybara, etc. into any Cucumber step definition. 

As of writing this, I am not too sure what the final result can look like. But hope it can act as a logical base to start and branch out as I discover more of its capabilities and effects.

### Getting Started with White Bread

Let's create a new mix project: `mix new elixirtest`

This will generate the skeleton for your project.

Let's define some of our dependencies, open up `mix.exs` and `:white_bread` and `:hound`:

```elixir
defp deps do
    [
        {:white_bread, "~> 2.5", only: [:dev, :test]},
        {:hound, "~> 1.0"}
    ]
end
```

Save your changes and run `mix deps.get` in your terminal. 

Next, let's create a `features` directory in the root of your project

Inside that folder create file called `test.feature` - `.feature` files denote that it is a [Cucumber feature file](https://github.com/cucumber/cucumber/wiki/Feature-Introduction).

Let's paste the following inside `test.feature`:

```gherkin
Feature: Test thing

    Scenario: Test 200 status code page
        Given I navigate to "http://the-internet.herokuapp.com/status_codes/200"
        Then the page contains the header "Status Codes"
```

This is going to be our test scenario, we are going to [http://the-internet.herokuapp.com/status_codes/200](http://the-internet.herokuapp.com/status_codes/200) and checking the header that we are on the right page. Pretty straightforward.

Awesome, now that we have a scenario, we need step definitions, these step definitions drive our automation.

Let's run White-Bread now: `mix white_bread.run`. This will compile your app and give you a prompt that looks like this:

```
Default context module not found in features/contexts/default_context.exs.
Create one [Y/n]?
```
Go ahead and type `y`. This will create our step definition file. 

The test will fail and will look like the following:

```elixir
Test 200 status code page ---> failed
1 scenario failed for Test thing
  - Test 200 status code page --> undefined step: I navigate to "http://the-internet.herokuapp.com/status_codes/200" implement with

given_ ~r/^I navigate to "(?<argument_one>[^"]+)"$/,
fn state, %{argument_one: _argument_one} ->
  {:ok, state}
end
```

As you can see from the output, it's telling us what we need to order to get it working! This is a nice feature! Go ahead and use the code generated and place it in your `default_context.exs` file. The whole file should look like this:

```elixir
defmodule WhiteBread.DefaultContext do
  use WhiteBread.Context

    given_ ~r/^I navigate to "(?<argument_one>[^"]+)"$/, fn state, %{argument_one: _argument_one} ->
        {:ok, state}
    end
end
```

Let's look at what's going on here. `given_` is a Cucumber keyword, and we use Regular Expression(regex) to parse the step. Coming from Ruby, you notice we actually call out the item we want to capture from the step. In our case, it's the url of the webpage we want to go to: "http://the-internet.herokuapp.com/status_codes/200". After we define our step, we pass in a function with state, which is supposed to keep track state between steps. After that, we have a map, or key/value pairs, which we use to store the url from the step. 

After each step, we want to return a tuple of the atom :ok and the state, so the next step can use state.

At this point we should run the test again using `mix white_bread.run` and add that new step to our `default_context.exs`. 

Your `deafault_context.exs` should look like this:

```elixir
defmodule WhiteBread.DefaultContext do
  use WhiteBread.Context

    given_ ~r/^I navigate to "(?<argument_one>[^"]+)"$/, fn state, %{argument_one: _argument_one} ->
        {:ok, state}
    end

    then_ ~r/^the page contains the header "(?<argument_one>[^"]+)"$/, fn state, %{argument_one: _argument_one} ->
        {:ok, state}
    end

end
```

Run the `mix white_bread.run` again and we should have similar output:

```elixir
Test 200 status code page ---> ok
All features passed.
```

Awesome! Let's wire up Hound next

### Wiring up Hound

Hound is used to drive our browser automation, but does not come with a way to run your WebDriver server. So, we have to do that ourselves. With that in mind, Hound currently supports a few options including: [PhantomJs](http://phantomjs.org/), [Selenium](http://www.seleniumhq.org/), and some others. We are going to use PhantomJs. The steps to set up Selenium are pretty identical and are shown in the [docs for Hound](https://github.com/HashNuke/hound). 

First install PhantomJs: `npm install -g phantomjs`

Config Hound by adding the following line in our `config.exs` file:

`config :hound, driver: "phantomjs"`

Since we are not using ExUnit to manage Hound for us, we need to manage the session for each scenario. White-Bread comes with something similar to a Hooks file in Ruby. 

We can add the following above our steps in `default_context.exs`:

```elixir
use Hound.Helpers

scenario_starting_state fn state ->
    Application.ensure_all_started(:hound)
    Hound.start_session
end

scenario_finalize fn state ->
    Hound.end_session
end
```

Here, we say before each scenario ensure Hound is started and start a session. Afterwards, we tear down the session. Pretty straightforward.

If we were to run our setup now, it will fail since we have no started our PhantomJs server. Open up another terminal and run PhantomJs in WebDriver mode: `phantomjs -w` and feel free to run your suite. It should return: 

```elixir
Test 200 status code page ---> ok
All features passed.
```

Hey! Look at that, we are running automation. Now we need to make our steps do stuff! Here is where the fun starts.

### Making our tests do work

Let's take a look at the first step: `Given I navigate to "http://the-internet.herokuapp.com/status_codes/200"`

Right now, our step is telling us to navigate to a url. Let's make it do just that.

Hound has a method called `navigate_to/1`. The only parameter the method requires is a URL. Which we are passing through our step. We should change the step to let other people know exactly what we are looking for. 

It should look like the following:

```elixir
given_ ~r/^I navigate to "(?<url>[^"]+)"$/, fn state, %{url: url} ->
    navigate_to(url)
    {:ok, state}
end
```

Run it and you should seen anything different. This is expected, its a headless browser, meaning we don't actually see it move around and click things. If you want that, you can use the Selenium configuration.

Moving onto the next step: `Then the page contains the header "Status Codes"`. In this step we are asserting that a header element on the page contains the text "Status Codes". 

First let's update our step to keep up with what we changed for the first step, updating our capture group name:

```elixir
then_ ~r/^the page contains the header "(?<expected_header>[^"]+)"$/, fn state, %{expected_header: expected_header} ->
    {:ok, state}
end
```

We can break the rest down into the following steps:

1) Identify the *actual* header element
2) Grab text from said header
3) Compare to the expected header passed in from our step

Hound comes with some methods to identify and find elements on the page. The main method is called `find_element/2`. This method takes in an html classification(class, id, tag, etc) and its value. In our case we have an element that we can only identify by using an h3 tag. Here's how that looks: `find_element(:tag, "h3")`.

To get the text of an element, we use the method: `visible_text/1`, which takes a selector. We can combine these two methods to use the strength of elixir:

```elixir
actual_header = find_element(:tag, "h3")
|> visible_text
```

The `|>`(pipe operator) feeds in the result from the thing(could be a method or variable) before as the first parameter of the next method.

This handles numbers 1 and 2. For number 3, we use an assert to, well assert, that the two things are equal. The assert looks like this `assert actual_header == expected_header`. The entire step definition should look like this:

```elixir
then_ ~r/^the page contains the header "(?<expected_header>[^"]+)"$/, fn state, %{expected_header: expected_header} ->
    actual_header = find_element(:tag, "h3")
    |> visible_text

    assert actual_header == expected_header
    {:ok, state}
end
```

Running the suite again should result in a success. If you are curious, as you should be, of what failures look like, change the expected header in the step from "Status Codes" to something like "Traffic Cones". Your error should look like this: 

```elixir
Test 200 status code page ---> failed


Assertion with == failed
code: actual_header == expected_header
lhs:  "Status Codes"
rhs:  "Traffic Cones"
```

Elixir gave us a real strong failure message. The message informed us of where the assertion failed, what it was using, and what each side of the assertion was.


### What's next?

This is a very primative tutorial. I am still exploring the options and capabilities. My goal is to raise the fidelity of this concept to its Ruby counterpart. It should be called out that the Ruby+Cucumber community has been around for a very long time. But, we have a lot of foundation to build upon, using a language that has many great benefits to reap. 

I am very interested in collaboration around this and would more than welcome some assistance. I think it's a step in the right direction. 

**Links**

* [Gitlab Repo](https://gitlab.com/distortia/elixir-cucumber-browser-automation-demo)


