---
title:  "Creating an Accordion in Phoenix Liveview"
date:   2023-12-04
---

# Creating an Accordion in Phoenix Liveview
Hello friends, I wanted to spend some time and write about converting an HTML/JS accordion to Phoenix's functional components in LiveView. We will go through what functional components are, `slots`, `attrs` and how to open/close without really writing any Javascript.

### Background
I wanted to start with an HTML/JS based accordion so we can go through just how powerful LiveView is

A basic HTML/Javascript accordion can look something like this:
```html
  <div class="max-w-md mx-auto">
    <div class="border rounded overflow-hidden">
      <!-- Accordion Item 1 -->
      <div class="border-b">
        <button
          class="w-full text-left py-2 px-4 font-semibold bg-gray-200 hover:bg-gray-300 focus:outline-none"
          onclick="toggleAccordion('content1')"
        >
          Section 1
        </button>
        <div id="content1" class="hidden p-4">
          <!-- Your content for Section 1 goes here -->
          Lorem ipsum dolor sit amet, consectetur adipiscing elit.
        </div>
      </div>

      <!-- Accordion Item 2 -->
      <div class="border-b">
        <button
          class="w-full text-left py-2 px-4 font-semibold bg-gray-200 hover:bg-gray-300 focus:outline-none"
          onclick="toggleAccordion('content2')"
        >
          Section 2
        </button>
        <div id="content2" class="hidden p-4">
          <!-- Your content for Section 2 goes here -->
          Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris.
        </div>
      </div>

      <!-- Add more accordion items as needed -->

    </div>
  </div>
```

With the ability to toggle open/close:
```javascript

  <script>
    function toggleAccordion(id) {
      const content = document.getElementById(id);
      content.classList.toggle('hidden');
    }
  </script>
```
That is all fine and dandy, but what if we want to reuse the `accordion` someplace else? We can convert this into a functional component!

### Functional Components
Functonal components allow us to reuse components across various facets of our application, including LiveView and non-LiveViews(some people call them dead views).

You can learn more about the `CoreComponents` in the [Phoenix Docs](https://hexdocs.pm/phoenix/components.html#corecomponents)
We will be using `core_components.ex` for our example.

We will then expand on the `accordion` to take in `slots` and `attributes`. [Attributes](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#module-attributes) are, as their name implies, attributes to pass to our component. These attrs are [Slots](https://hexdocs.pm/phoenix/components.html#corecomponents) are markup that we want our component to render in some way.

Hopefully this all makes more sense later on.

### Getting Started

We can generate a new Phoenix app: `mix phx.new --no-ecto my_app`. Once finished, there is a new `core_components.ex` that was generated.
These core components are all functional components and LiveView provides a few already(buttons, forms, etc.)

Let's start up the app with `iex -S mix phx.server`

We need a canvas to paint our `accordion` onto, so we should create a new `liveview`

Inside `router.ex` add a new `live` route:

`live "/hello", MyAppLive`

Now create the new `LiveView` file at `./my_app/lib/my_app_web/live/my_app_live.ex`

And paste the following in `my_app_live.ex`:
```elixir
defmodule MyAppWeb.MyAppLive do
  use Phoenix.LiveView
  import MyAppWeb.CoreComponents

  def render(assigns) do
    ~H"""
      <h1>Hello there</h1>
    """
  end
end
```

If we visit `http://localhost:4000/hello` we should see our `Hello there`.

### Converting HTML/JS Accordion to Functional Component

Conversion is pretty straightforward in our case, as HEEX templates are already HTML.

First we need to define our `accordion` component inside `lib/my_app/components/core_components.ex` and paste our original `accordion` without the JavaScript and `onclick`. We will re-add the click functionality later.

```elixir
  def accordion(assigns) do
    ~H"""
      <div class="max-w-md mx-auto">
        <div class="border rounded overflow-hidden">
          <!-- Accordion Item 1 -->
          <div class="border-b">
            <button
              class="w-full text-left py-2 px-4 font-semibold bg-gray-200 hover:bg-gray-300 focus:outline-none"
              onclick="toggleAccordion('content1')"
            >
              Section 1
            </button>
            <div id="content1" class="p-4">
              <!-- Your content for Section 1 goes here -->
              Lorem ipsum dolor sit amet, consectetur adipiscing elit.
            </div>
          </div>

          <!-- Accordion Item 2 -->
          <div class="border-b">
            <button
              class="w-full text-left py-2 px-4 font-semibold bg-gray-200 hover:bg-gray-300 focus:outline-none"
              onclick="toggleAccordion('content2')"
            >
              Section 2
            </button>
            <div id="content2" class="p-4">
              <!-- Your content for Section 2 goes here -->
              Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris.
            </div>
          </div>

          <!-- Add more accordion items as needed -->

        </div>
      </div>
    """
  end
```

We can now use our `accordion` in our LiveView! All we need to do is replace our `<h1>Hello there</h1>` with `<.accordion></.accordion>`. We should now see our `accordion`.

### Adding Slots and Attributes to our Functional Component
Having hard coded markup specific to this accordion is undesired if we want to reuse it in other places. We can use `slots` and `attrs` to render mark up being passed into the component.


Slots and attrs need to be defined above our `accordion` definition and will look like the following:

```elixir
  slot :content do
    attr :title, :string
    attr :id, :string
  end

  def accordion(assigns) do
    ~H"""
    <div class="max-w-md mx-auto">
      <div class="border rounded overflow-hidden">
        <%= for content <- @content do %>
          <div class="border-b">
            <button
              class="w-full text-left py-2 px-4 font-semibold bg-gray-200 hover:bg-gray-300 focus:outline-none"
            >
              <%= content.title %>
            </button>
            <div id={content.id} class="p-4">
              <%= render_slot(content) %>
            </div>
          </div>
        <% end %>
      </div>
    </div>
    """
  end
```
Here, we are using a named `slot` called `content` that has a few attributes that will help us when we add the open/close functionality back.

We need to update our LiveView to use the new `accordion` functionality:
```elixir
    <.accordion>
      <:content title="my title" id="my-title">
        <h1>Hello there</h1>
      </:content>
      <:content title="my second title" id="my-second-title">
        <h1>Hello there x2</h1>
      </:content>
      <:content title="my third title" id="my-third-title">
        <h1>Hello there x3</h1>
      </:content>
    </.accordion>
```

Currently, we cannot open/close it, as we are not using the JavaScript that came with it. So let's do that now!


### Replacing JavaScript with LiveView JS
Okay, so, this is technically cheating as we are still using JavaScript. LiveView has a module called [`JS`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.JS.html). This module is full of utility functions that will call the JavaScript for us without us having to write a script for it.

We need to add a `phx-click` to the `button` in the accordion:
```js
phx-click={JS.toggle(to: "##{content.id}")}
```

`JS.toggle()` will
> Show or hide elements based on visibility, with optional transitions


### Condiontally Show/Hide by default
If we wanted to start with the accordion panels closed by default we can use an `attr` called `start_closed`

Add the `attr` to the `content` slot:
```elixir
  slot :content do
    attr :title, :string
    attr :id, :string
    attr :start_closed, :boolean
  end
```


And add a condtional style to the accordion panel:
```html
<div id={content.id} class="p-4" style={if content[:start_closed], do: "display: none;"}>
```

Note how we have to access the `start_closed`, currently we cannot set default values for slot attributes

Finally, add the attribute to some of the accordion's `content` slots:
```elixir
<:content title="my second title" id="my-second-title" start_closed={true}>
<:content title="my third title" id="my-third-title" start_closed={true}>
```

We have parts of the accordion that are hidden by default while retaining the ability to toggle as needed!

### Conclusion
Functional components are super powerful and allow us to reuse components across our applcation. Slots and attributes can be somewhat intimidating at first but hopefully this helped to bring you the confidence you need.
