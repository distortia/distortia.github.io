---
title:  "Exploring Page Objects in Elixir"
date:   2017-05-20
---

Page Objects are a well understood paradigm in the testing world. In Ruby, i use a well known gem which happens to be called [Page Object](https://github.com/cheezy/page-object). This gem abstracts out html elements and provides helper methods for interacting with elements on the pages and the browser. Exploring the concepts of page objects in the Elixir world is my focus for this post. It may get a little weird.

# What is a Page Object?

A great explanation comes from a post from [Martin Fowler](https://martinfowler.com/bliki/PageObject.html). In the post he describes page objects as a wrapper for the html. You abstract your pages into objects and interact with them as a normal human would. This abstraction allows for reusable logic and flexibility to describe your application as a series of pages to flow through. These interactions are why testing is so valuable to businesses. It's far more beneficial to have a test suite that emulates user behavior, so you can spend more time shipping a quality product.

# So what can we do in Elixir?

I said it may get a little weird, and here's my reasoning behind it. Elixir is a functional programming language, meaning there are no objects. But, there are Elixir goodies that allow us to tap into some of the benefits of objects. These goodies are structs, processes, macros and module attributes. For a better breakdown of the "object oriented" nature of Elixir, check out this post from [noredink](http://tech.noredink.com/post/142689001488/the-most-object-oriented-language). Let's explore some of these.

# Setup
The packages I am using in this post are [Hound](https://github.com/HashNuke/hound) and [White Bread](https://github.com/meadsteve/white-bread). For more info on these check out my [last post](/elixir-cucumber-browser-testing.html) and the repo used on my [Gitlab repo](https://gitlab.com/distortia/elixir-cucumber-browser-automation-demo).

# Plain Methods

Using plain old methods to define elements is a pretty common paradigm. Essentially, every element is a method and is resolved to the locator strategy and locator. We are using these methods to hold our contstants. For the list of locator strategies, check out [Hound's documentation](https://hexdocs.pm/hound/Hound.Helpers.Page.html#find_element/3).

Here is how we can define our login page:

```elixir
defmodule LoginPage do
  def username_field, do: {:id, "username"}
end
```

In our test we can do `fill_field(LoginPage.username_field, "testuser")`

# Module Attributes

Module attributes are better used as constants.

Here is an example:

```elixir
defmodule LoginPage do
  @username_field {:id, "username"}
end
```

Module attributes are only accessible within the module they were defined in, unless we add a method which returns the attribute.

Here's the updated code:

```elixir
defmodule LoginPage do
  @username_field {:id, "username"}
  def username_field, do: @username_field
end
```

This works the same way as the plain method example. But the issue here is that we have duplication of code. Which is less than ideal.

# Macros and metaprogramming

The above samples explore a primitive type of page object. The real strength behind the Page Object gem is its use of meta programming. With Page Object we can define our element as `text_field(:username_field, :id => 'username')`. Page objects takes this and turns it into a method called `username_field`, using the locator strategy of `id` and locator of `username`. the `text_field` has built in methods which can be found [here](http://www.rubydoc.info/gems/page-object/PageObject/Elements/TextField). The one we care about is the `value=(new_value)`. In our case calling `username_field='testuser'` will set the username_field to testuser. Meaning we don't have to call `username_field.set 'testuser'`

We are going to try recreating a page object-like macro for our username field above. Metaprogramming in Elixir is based around [macros](https://hexdocs.pm/elixir/Macro.html). 




