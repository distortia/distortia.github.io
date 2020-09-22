---
title:  "Embed Screenshots into Cucumber reports using Calabash 2.0"
date:   2016-01-28
permalink: "embed-calabash-screenshots"
---
Hey Everyone, sorry I have been gone for so long. Been real busy with a new job and life in general. 

I'll keep this post short and simple.

Many of us would like some kind of visual indication or guidance as to why our testing fails. This can be especially hard when
automating native apps on physical devices. Fortunately, Calabash 2.0 has a real clean solution that is easy to implement: `screenshot_embed`

Here is what `screenshot_embed` does(found [in screenshot.rb, here](https://github.com/calabash/calabash/blob/develop/lib/calabash/screenshot.rb)):

{% highlight ruby %}
# Takes a screenshot and embeds it in the test report. This method is only
# available/useful when running in the context of cucumber.
# @see Screenshot#screenshot
def screenshot_embed(name=nil)
  path = screenshot(name)
  embed(path, 'image/png', name || File.basename(path))
end
{% endhighlight %}

Pretty cool, right?

All we have to do is add that method into our after hook for cucumber and that's it.

Here's how i did it. `calabash generate` generates a nice scaffold for us to get running, here is the after hook it generates:

{% highlight ruby %}
After do
  stop_app
end
{% endhighlight %}

We are going to add our `embed_screenshot` method like this:

{% highlight ruby %}
After do |scenario|
  screenshot_embed if scenario.failed?
  stop_app
end
{% endhighlight %}

We only want to show screenshots on failures, that's why we wrap it around the scenario.failed? if-block. 

They will appear under your failed scenario as such:

![Cucumber Calabash Embed Screenshot](/img/cucumber-calabash-embed-screenshot.png)

Now go get testing and check if your screenshots are working(by making them fail)!