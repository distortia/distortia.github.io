---
layout: post
title: "Phoenix LiveView: Offline-ish list with localStorage and a follow-up event"
date: 2026-02-15
categories: [engineering, elixir, phoenix]
tags: [phoenix, liveview, elixir, localStorage, reconnect, mobile]
author: Nick Stalter
excerpt: "Stop your LiveView from doing a full refresh every time the user unlocks their phone: cache list data in localStorage and restore it via a follow-up event (not connect params) while refetching in the background."
---

On mobile, locking the screen often drops the WebSocket or suspends the tab. When the user unlocks and returns to your LiveView, the process may have been torn down and **mount runs again**. If mount always shows a loading spinner and refetches from the server, the user sees a full refresh every time they check their list—annoying when they're in the store and opening the app every few minutes.

This post shows a small pattern to make a list (or any data-heavy view) feel "offline-ish": **cache a serializable snapshot in localStorage**, have the client **push it to the server in a follow-up event** (not in connect params—that can blow the URI length limit), and **show it immediately** while still refetching in the background. If decode or staleness check fails, you fall back to a normal load.

I used this for the shopping list in [OnRotation](https://onrotation.app); the same idea applies to any LiveView that remounts often on mobile.

## 1. Cache shape

Store a JSON object with whatever the template needs to render, plus a timestamp for staleness:

- `entries` – array of items (e.g. `id`, `display_name`, `quantity`, `category`, `checked`, …)
- Any other assigns the template uses (e.g. `selected_week_start`, `week_meal_names`, `item_key_to_meal_names`)
- `cached_at` – ISO8601 timestamp so the server can ignore cache older than your TTL (e.g. 15 minutes)

Use a single key per origin (e.g. `myapp_shopping_list`) if you have one list per user.

## 2. Server: push cache after loading or updating

Whenever you assign the list (or related) data, push a serializable payload to the client so the hook can write it to localStorage. No structs or `DateTime` in the payload—plain maps and strings.

```elixir
defp build_cache_payload(assigns) do
  %{
    "entries" => Enum.map(assigns.entries || [], &entry_to_cache/1),
    "selected_week_start" => assigns.selected_week_start && Date.to_iso8601(assigns.selected_week_start),
    "cached_at" => DateTime.utc_now() |> DateTime.to_iso8601()
    # ... other keys the template needs
  }
end

defp push_cache(socket) do
  push_event(socket, "cache_shopping_list", build_cache_payload(socket.assigns))
end
```

Call `push_cache(socket)` at the end of `handle_info(:load_plan, socket)` and after every `handle_event` that updates the list (toggle, add, delete, etc.).

## 3. Client: hook to write cache

A hook that listens for the event and stores the payload:

```javascript
const CACHE_KEY = "myapp_shopping_list"

const ShoppingListCacheHook = {
  mounted() {
    this.handleEvent("cache_shopping_list", (payload) => {
      try {
        localStorage.setItem(CACHE_KEY, JSON.stringify(payload))
      } catch (_) {}
    })
  }
}
```

Mount the hook on an element in your LiveView (e.g. a hidden div so it’s present when the list is rendered).

## 4. Client: send cache in a follow-up event (not connect params)

**Do not** put the cache in LiveSocket’s `params`. Those end up in the connection URL and can hit **URI length limits** (often 4–8KB), causing “request URI too long” errors and even infinite redirect/error loops when the list is large.

Instead, have the same hook that writes the cache **push an event** to the server when it mounts. The payload goes in the WebSocket message body, so size is not an issue:

```javascript
const ShoppingListCacheHook = {
  mounted() {
    this.handleEvent("cache_shopping_list", (payload) => {
      try {
        localStorage.setItem(CACHE_KEY, JSON.stringify(payload))
      } catch (_) {}
    })
    // Send cache to server in a follow-up event to avoid URI length limit
    try {
      const cached = localStorage.getItem(CACHE_KEY)
      if (cached) this.pushEvent("restore_cached_list", { cache: cached })
    } catch (_) {}
  }
}
```

Keep `params: { _csrf_token: csrfToken }` (or a simple static object); no cache there.

## 5. Server: apply cache when the event arrives

Handle the event: if you’re still loading and the cache is valid, apply it and set your loading flag to `false`. You still run the refetch (e.g. `send(self(), :load_plan)` in mount); when it completes, assign fresh data and call `push_cache` again.

```elixir
@cache_ttl_seconds 900  # 15 minutes

def mount(_params, _session, socket) do
  socket = assign_defaults(socket)
  send(self(), :load_plan)
  {:ok, socket}
end

def handle_event("restore_cached_list", %{"cache" => cache_str}, socket) when is_binary(cache_str) do
  if socket.assigns.plan_loading? do
    case try_apply_decoded_cache(socket, cache_str) do
      {:ok, updated} -> {:noreply, assign(updated, :plan_loading?, false)}
      :skip -> {:noreply, socket}
    end
  else
    {:noreply, socket}
  end
rescue
  _ -> {:noreply, socket}
end

def handle_event("restore_cached_list", _params, socket), do: {:noreply, socket}

defp try_apply_decoded_cache(socket, cache_str) do
  case Jason.decode(cache_str) do
    {:ok, decoded} when is_map(decoded) ->
      if cache_fresh?(decoded), do: {:ok, apply_cached_assigns(socket, decoded)}, else: :skip
    _ -> :skip
  end
end

defp cache_fresh?(decoded) do
  case decoded["cached_at"] do
    nil -> false
    iso when is_binary(iso) ->
      case DateTime.from_iso8601(iso) do
        {:ok, dt, _} -> DateTime.diff(DateTime.utc_now(), dt, :second) <= @cache_ttl_seconds
        _ -> false
      end
    _ -> false
  end
end
```

Decode and apply: build `entries` (and any derived assigns like `entries_by_section`) from the decoded map. If anything fails (decode, missing keys, bad types), leave the socket as-is and let the refetch show the real data. No need for elaborate validation—fail fast and fall back.

## Edge cases

- **No cache** – Behave as today: loading spinner, then load.
- **Stale cache** – Ignore if `cached_at` is older than your TTL; load as normal.
- **Decode / structure error** – Ignore cache and refetch. Defensive decode keeps bad or tampered data from breaking the view; the server remains the source of truth.

## Security note

The cache is only for UX: show something immediately. The server always re-fetches and scopes data by the authenticated session. Don’t trust the cache for authorization. For a shopping list or similar, storing it in localStorage is fine; for more sensitive data, consider whether you want it on the client at all.

## Summary

1. After each load or mutation, `push_event(..., "cache_...", payload)` with a serializable snapshot.
2. A JS hook writes that payload to localStorage and, on mount, reads it and `pushEvent("restore_cached_list", { cache })` so the server gets it in a **WebSocket message** (not connect params—avoids URI length limits).
3. Server handles `"restore_cached_list"`: if still loading and cache is valid (decode + TTL), apply it and set loading to false.
4. Mount always triggers a refetch; when it completes, assign fresh data and push the new cache.

With a short TTL (e.g. 15 minutes), users get an instant list when they reopen the app after a brief lock, and the list still refreshes in the background so it stays accurate.

For more on hooks and storage, see the earlier post [Phoenix LiveView with localStorage or sessionStorage](https://distortia.github.io/2022/04/14/phoenix-liveview-with-localstorage-or-sessionstorage.html); this one focuses on **reconnect/mount** and sending the cache in a **follow-up pushEvent** (not connect params) to avoid both the full refetch and URI length limits.
