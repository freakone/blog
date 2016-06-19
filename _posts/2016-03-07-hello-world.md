---
layout: post
title: Hello world!
---

Hello! It's a sample first post.

{% highlight python %}

def read_word_with_sign(adr):
    val = read_word(adr)
    if (val >= 0x8000):
        return -((65535 - val) + 1)
    else:
        return val

{% endhighlight %}
