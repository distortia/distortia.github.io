---
title:  "Adding Bower Components to Phoenix(Elixir)"
date:   2016-05-24
---

Recently I've been digging into [Elixir](http://elixir-lang.org/), [Phoenix](http://www.phoenixframework.org/), and [Polymer](https://www.polymer-project.org/1.0/). When I thought of bringing Polymer into Phoenix, my first thought was, "This should be easy. Phoenix comes with [Brunch](http://brunch.io/) and that works with bower out of the box!". I was *sort* of right.

After some quick trial and error and digging on Stack Overflow, I created a combination that worked for me.

### bower.json

`bower.json` is just the place where we define the Bower components we want to use. Let's create a `bower.json` file in the root of the project. Inside that file, we only need the following(please fill in your project name):

```
{
  "name": "project-name",
  "dependencies": {
  }
}
```

### .bowerrc

`.bowerrc` tells bower specific details about our bower usage, such as where to install components, proxies, and more. In this file we want to specify the location where we want our `bower_components` folder. Create the following `.bowerrc` file in the root of your project.

This is what mine looks like:

```
{
  "directory": "web/static/assets/bower_components"
}
```

Phoenix likes to keep all assets inside the `web/static/assets` folder. This is where Brunch takes all the files(css, js), concats/minifies, and more importantly, moves them to a public facing folder for the app to use!

### Expose The bower_components Folder

Inside `lib/<Project>/endpoint.ex`, add `bower_components` to

{% highlight elixir %}
  plug Plug.Static,
    at: "/", from: :discovery, gzip: false,
    only: ~w(css fonts images js favicon.ico robots.txt)
{% endhighlight %}

so it looks like this:

{% highlight elixir %}
  plug Plug.Static,
    at: "/", from: :discovery, gzip: false,
    only: ~w(css fonts images js favicon.ico robots.txt bower_components)
{% endhighlight %}


The snippet above is whitelisting the following files/folders for public access. We now have access to our `bower_components` folder!

### Install Components

From here on out its smooth sailing. We can install Bower components using the `--save` switch and watch them show up in your `bower_components` folder. That's it!


