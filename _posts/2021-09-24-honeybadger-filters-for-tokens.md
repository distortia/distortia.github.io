---
title:  "Custom Honeybadger filters for Tokens"
date:   2021-09-24
---

Hey All, it has been quite a long time since my last post. I apologize for that. Today, I will be keeping things short and sweet.

# Custom Honeybadger filters for Tokens

[Honeybadger](https://www.honeybadger.io/) is a service for "Exception, uptime, and cron monitoring, all in one place". At [InfluxData](https://influxdata.com), we use this service to monitor our application and recieve notifications when things go wrong. Sometimes, we receive a _bit_ too much information, such as Bearer Tokens.

Leaking tokens is a serious security concern. Bad faith actors can use that token to manipulate your systems or systems you are connected to. Once a token has been exposed, you need to fix the issue and rotate your tokens.

In this case, we want to redact any token going to Honeybadger.

Fortunately, the [Honeybadger Elixir Repo](https://github.com/honeybadger-io/honeybadger-elixir) has an easy way for us to create a [custom filters](https://hexdocs.pm/honeybadger/Honeybadger.Filter.Mixin.html). Let's do that now.


```elixir
defmoudle MyApp.MyFilter do
  use Honeybadger.Filter.Mixin

  def filter_error_message(message) do
    Regex.replace(~r/(Token|Bearer) [^"]+/, message, "\\1 redacted")
  end
end

```

This will filter error messages before being sent to Honeybadger and check for `Token <token>` or `Bearer <token>` and replace them with `Token redacted`. What is nice is that this covers [Tesla](https://github.com/teamon/tesla), [HTTPoison](https://hexdocs.pm/httpoison/readme.html) and probably any other HTTP library you pick. All the error messages are just strings. That means we can expect escaped strings and can capture the values up until we hit the closing `"`.

So, let's write a few tests for these scenarios:

__NOTE__: We are looking for typical `Bearer <token>` and `Token <token>`. Error messages are truncated as they are quite large.

```elixir
defmodule MyApp.HoneybadgerFilterTest do
  use ExUnit.Case, async: true
  alias MyApp.HoneybadgerFilter

  describe "filter_error_message" do
    test "replaces general tokens" do
      token = "some.jwt.abc123"
      message = "[token: \"#{token}\"]"
      actual = HoneybadgerFilter.filter_error_message(message)
      assert actual =~ "[token: \"redacted\"]"
      refute actual =~ token
    end

    test "replaces bearer tokens" do
      token = "some.jwt.abc123"
      message = "{\"Authorization\", \"Bearer #{token}\"}"
      actual = HoneybadgerFilter.filter_error_message(message)
      assert actual =~ "{\"Authorization\", \"Bearer redacted\"}"
      refute actual =~ token
    end
  end
end
```

The tests will cover and probably any other HTTP library you pick.

## Conclusion

Adding custom filters for Honeybadger can be very straightforward. With a little of regex(I am sorry), you can safely export errors without leaking tokens!

Thanks!

Nick
