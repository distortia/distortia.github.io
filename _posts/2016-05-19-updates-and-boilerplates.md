---
title:  "Updates and Boilerplates"
date:   2016-05-19
---

Sorry for the lack of content lately, been a busy few months(switching jobs and what not)

In the meantime I have been working on creating a boilerplate for some automation that should prove to be useful.

Introducing [App Automation Boilerplate](https://gitlab.com/distortia/app_automation_boilerplate) and [Web Automation Boilerplate](https://gitlab.com/distortia/web_automation_boilerplate)!!

These two repos are a culmination of efforts and problems I encountered when working on creating new test suites. Those of us that have been in the test automation game for a while know how much code you may have to write to get things up and running in a smooth elegant manor. 

Well, I took the things that I wanted and created a boilerplate with them. Let's take a look:

#### App Automation Boilerplate

The App Boilerplate comes with [Calabash 2.0 pre](http://calaba.sh/), the cross platform(iOS & Android) testing library for Ruby.

The boilerplate is made up of:

* Ruby
* [Cucumber](https://cucumber.io/)
* [Allure Reporting](http://allure.qatools.ru/) - Beautiful reporting and metrics tool
* Custom Routing Proof of Concept - based off of [PageObject's](https://github.com/cheezy/page-object) Routes
* [RSpec](http://rspec.info/) - for better test expectations
* Custom random email generator - if you didnt want/need the [Faker gem](https://github.com/stympy/faker)
* Custom data expectations based off of stored data in YAML files based off of elements on the app
* Page objects setup for Calabash use
* Cucumber profiles and variables
* Custom common methods for interacting with the app
* IDeviceInstaller file - for installing iOS devices from the app
* And more!

The Web Boilerplate comes with 

* Ruby
* Cucumber
* [SafariDriver](https://github.com/SeleniumHQ/selenium/wiki/SafariDriver)
* [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/)
* Various devices you can use for testing different dimensions
* Headless capabilities
* Allure Reporting
* Page Object + Data Magic + Routing(to get you to where you need to go in your app)
* RSpec
* [Parallel-Cucumber](https://github.com/badoo/parallel_cucumber) - run multple scenarios in parallel
* Custom data expectations
* And more!

These projects will be actively maintained by me to support the latest changes in the testing world. There is documentation in each of the README's but I will probably miss things. Please feel free to contribute!