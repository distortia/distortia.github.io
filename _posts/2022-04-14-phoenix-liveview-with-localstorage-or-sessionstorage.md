---
title:  "Phoenix Liveview using LocalStorage or SessionStorage"
date:   2022-04-14
---

# Phoenix Liveview using LocalStorage or SessionStorage

Sometimes we want to fetch or store data in `localStorage` or `sessionStorage`, we can fetch/store this data using [LiveView hooks](https://hexdocs.pm/phoenix_live_view/js-interop.html#client-hooks-via-phx-hook).
These hooks allow for interoperability with JavaScript and grant us the ability to interact with a liveview during its lifecycle.

For storing the data, we will call a JavaScript hook inside a liveview function to store the value of a counter.

For fetching the data, we will be using the `reconnected` hook to fetch data from `localStorage` or `sessionStorage`. In this post we will be using `sessionStorage` as the data is wiped when we close the tab. If you wanted to persist the data, `localStorage` is what you are looking for. Both storages use the same function `getItem(key)` where we provide a key which _hopefully_ has the values we are looking for.

For more information regarding `sessionStorage` and `localStorage` see the [Mozilla Docs](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)

## Cloning the Demo Project

Let's clone the demo Phoenix LiveView app. The main branch will only contain the minimal amount of setup without implementing any hooks:

`git pull git@github.com:distortia/counter_example.git`

For the complete solution check out the `complete` branch.

## Setting up the Hooks

__NOTE__: All hooks need to be defined above the following line `let liveSocket = new LiveSocket("/live", Socket, {params: {_csrf_token: csrfToken}})`

Initializing our empty `hooks` object to be populated later:

```
let hooks = {}
```

We need to add our hooks to the `liveSocket`:

```
let liveSocket = new LiveSocket("/live", Socket, {
  params: {_csrf_token: csrfToken},
  hooks: hooks,
})
```

## Saving State on Button Click

To save the state of our counter on button click we need to write a custom hook.

In `app.js`, we are going to create a hook called `saveCount`:

```

hooks.saveCount = {
  mounted() {
    this.handleEvent("saveCount", ({ count }) =>
      sessionStorage.setItem("count", count)
    )
  }
}
```

`saveCount` uses the `mounted` hook so it gets attached as the component is mounted.

For the `handle_event` function we use a new function called `push_event` which will invoke the JavaScript hook we called `saveCount`

```
def handle_event("inc", _session, socket) do
  count = socket.assigns.count + 1
  socket = socket |> assign(count: count)
  {:noreply, push_event(socket, "saveCount", %{count: count})}
end
```

To wire up our button we need to add a unique ID as well as the hook:

`<button id="inc-button" phx-click="inc" phx-hook="saveCount">Increment</button>`

We should now see the value of count being incremented in `sessionStorage` as well as on the page

## Restoring state on Reconnection

To restore the state from `sessionStorage` we need to create a new hook that I will call `restore`:

```
hooks.restore = {
  mounted() {
    count = sessionStorage.getItem("count")
    this.pushEvent("restore", {count: count})
  }
}
```

This hook will call our `restore` event in the liveview which looks like:

```
def handle_event("restore", %{"count" => count}, socket) do
  count = String.to_integer(count)
  socket = socket |> assign(count: count)
  {:noreply, socket}
end
```

__NOTE__: Values in sessionStorage are strings and we need to properly convert to the correct type

We do need to handle cases where we have no value in the `sessionStorage`, we can do that by explcitly handling `nil` values above the original `handle_event` call:

```
def handle_event("restore", %{"count" => nil}, socket), do: {:noreply, socket}

def handle_event("restore", %{"count" => count}, socket) do
  count = String.to_integer(count)
  socket = socket |> assign(count: count)
  {:noreply, socket}
end
```

In order to actually call the `restore` hook we need to add a unique ID to the element and attach our hook to it:

`<section class="phx-hero" id="page-live" phx-hook="restore" >`

We now have a way to restore data from `sessionStorage` or `localStorage` into our LiveView!

To test this out, we can go into the developer settings and manually add key-value pair to `sessionStorage`

example: `count: 100`

When we refresh the page, the counter should reflect the value in the `sessionStorage`

## Conclusion

This is how to store and restore data in a LiveView using Hooks and `sessionStorage`. If you found it helpful, reach out on twitter or share with your friends!

Thanks!

### Links
- [Repo](https://github.com/distortia/counter_example)
- [LiveView Hooks](https://hexdocs.pm/phoenix_live_view/js-interop.html#client-hooks-via-phx-hook)
- [MDN SessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
