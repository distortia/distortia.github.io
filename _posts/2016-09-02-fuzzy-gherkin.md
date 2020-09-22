---
title:  "Fuzzy Gherkin Gem"
date:   2016-09-02
---

Hey Everyone, I know it's been a while since I wrote about something in the QA space. But I return with some news on a gem I recently fleshed out. I call it Fuzzy Gherkin.

#### What is Fuzzy Gherkin?

Fuzzy Gherkin is a command line tool I wrote to cut down on the amount of similar steps I encounter in Gherkin. The name is derived from [fuzzy matching](https://en.wikipedia.org/wiki/Approximate_string_matching) strings and [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin)

For those that don't know what Gherkin is, check out the docs I just linked above. But it is simply a tool that allows us to write tests based on user interactions within our application, in plain English. The plain English part is the caveat here. This means that tech and non-tech people can collectively collaborate using a common medium. Engineers take the Gherkin and translate that into code which executes to what the Gherkin says. Pretty neat, right? 

Here is an example: 

```gherkin
    Feature: Serve coffee

      Background: 
        Given I want some coffee
        And the office has some in stock

      Scenario: Making espresso  
        When I press the espresso button
        Then espresso should be brewing
```

The engineers doing the conversion can abstract out specific terms inside the steps making our more generic on the back end, with little impact on the front end.

An example of the abstracted step would be something like this: 

`When /^I press the (.*) button$/`

The front end remains the same, we can still write "When I press the espresso button" in our Gherkin but it gets treated differently on the back end. We can expand the functionality to be able to push any button. FYI, the `(.*)` is a [regular expression](https://en.wikipedia.org/wiki/Regular_expression) wildcard capture group. Which means I'm asking for anything I put into this block, in our case its the word "espresso", or "coffee", or any other button we want to push.

Now we can write any steps that resemble the "I press the espresso button" we can do things like "I press the coffee button". While it may look different on the front end, its handled the exact same way as the espresso button is. 

We are making our steps more dynamic and verbose, allowing for greater reusability and maintainability in our code base. Besides, who wants to remember a gazillion unique steps? Not me.

#### Why Fuzzy Gherkin?
So we are writing all these awesome tests and user scenarios that are getting automated. Everyone is collaborating and all is good. Until, you realize that you may have some steps that can be combined into each other. Or, in my most recent case, you join a project that's pretty far in the development cycle and you have Test Analysts who write all the Gherkin and pass the automation for you to do. This is where communication and experience can get everyone back onto track with the scenarios.

#### How does Fuzzy Gherkin work?
The core is built around this [fuzzy string match gem]( https://github.com/kiyoka/fuzzy-string-match) which implements the [Jaro-Winkler distance algorithm](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance). The algorithm measures the similarity between two strings, which is exactly what we are doing.

All we do is pass in our base step, the one we want to compare all the other steps to and a threshold, from 0.00 to 1.00. The threshold is essentially a % similarity match, where the higher the % the greater the match, and a 1.00 match means the strings are equal.

Let's check out an example.

#### Getting started + Example

1. Install the fuzzy_gherkin gem - `gem install fuzzy_gherkin`
2. In your terminal/command prompt navigate to your testing directory.
3. `fuzzy_gherkin compare "I clicked the button"` - if we wanted to specify a threshold we can. The default is 0.80.
4. The program with do its thing and spit out the results in a `results.json` file which will look similar to this: 

```json
{
    "I clicked the button": [
        "90.06% - I click the back button",
        "85.69% - I click the tooltip",
        "85.09% - I click the get started button",
        "84.66% - I click the sign in button",
        "83.81% - I click the done active button",
        "82.64% - I click the <family_type> button",
        "82.63% - I click the next active button",
        "82.05% - I click off the field",
        "81.34% - I click the cat videos link",
        "81.27% - I click the next inactive button",
        "80.57% - I click the send link active button"
    ]
}
```


Here we pass in the base step "I clicked the button" and in the feature files I had it scan, it returned the following steps in descending order by % match. The program will remove all duplicates and perfect matches.

But that's it. As engineers, we can take this info and do further inspection and see if refactoring can/should happen.

#### What's next?

As we can see, the gem has a solid foundation, but lacks some of the higher features I would like.

Some of these include: 

* More verbose with how the data is formatted, i.e. giving a line# + feature file

* Cleaning of data - strip out words with little value/meaning - i.e. "and, the, etc" - This should boost our threshold %'s too

* Dynamically compare all the steps to each other without me having to specify a base step. Baby steps

That's all I have for today, feel free to submit a PR, fork it, open an issue, reach out to me, etc. 

------
**Links**

* [Gitlab Repo](https://gitlab.com/distortia/fuzzy_gherkin)

* [Rubygems.org Repo](https://rubygems.org/gems/fuzzy_gherkin)
