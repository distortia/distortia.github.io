---
title:  "Introducing Calapog"
date:   2016-06-03
---

Hey everyone, after releasing my boilerplates and thinking of ways to improve it, I came up with the notion to create a generator/scaffolder for Calabash page objects. *Note:* Best results when paired with my [app automation boilerplate](https://gitlab.com/distortia/app_automation_boilerplate)

#### Introducing Calapog

[Calapog](https://rubygems.org/gems/calapog) takes the effort of creating multiple files and reduces it into a simple command, `calapog generate`.


#### Example

Add the `calapog` gem to your Gemfile and run `bundle install`

`calapog generate SharedTestPage` will generate 3 files as such:

`features/pages/shared_test_page.rb` with the contents: 

```ruby
class SharedTestPage < CommonPage

  def trait

  end

  def page_data_file
    'test_page'
  end

  private
end
```

`features/android/pages/test_page.rb` with the contents: 

```ruby
class Android::ClassPage < SharedClassPage
  include Calabash::Android

  private
end
```

`features/ios/pages/test_page.rb` with the contents:

```ruby
class IOS::ClassPage < SharedClassPage
  include Calabash::IOS

  private
end
```


These files are generated in accordance with [Calabash's Cross Platform](https://github.com/calabash/x-platform-example) and many other examples paired with Calabash 2.0 implementations.

This is a very rough and barebones release. There is more on the roadmap. Such as a setup command which will take in the various paths for each file and the ability to customize what gets generated.

#### links
* [Rubygems Repo](https://rubygems.org/gems/calapog)
* [GitLab Repo](https://gitlab.com/distortia/calapog)
