---
layout: post
title: Elixir Phoenix's Ecto strange error
---

Recently I started new Elixir project on a clean OS and when I ran ```ecto.create``` command
I got this strange looking error. No message, nothing.

{% highlight bash %}
λ mix ecto.create
** (ArgumentError) argument error
    (stdlib) :io.put_chars(:standard_error, :unicode, [[[], <<42, 42, 32, 40, 77, 105, 120, 41, 32, 84, 104, 101, 32, 100, 97, 116, 97, 98, 97, 115, 101, 32, 102, 111, 114, 32, 79, 102, 102, 105, 99, 101, 69, 108, 105, 120, 105, 114, 46, 82, 101, 112, 111, 32, 99, 111, 117, 108, ...>>], 10])
    (mix) lib/mix/cli.ex:67: Mix.CLI.run_task/2
    (elixir) lib/code.ex:363: Code.require_file/2
{% endhighlight %}

The solution is pretty simple: either change postgres credentials in your system to default
```postgres:postgres``` or define proper password in your ```dev.exs``` confix file.
