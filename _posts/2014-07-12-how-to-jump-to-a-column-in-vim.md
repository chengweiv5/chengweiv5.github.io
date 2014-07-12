---
layout: post
title: How to jump to a column directly in Vim
tags:
  - vim
---

OK, this is quite a small tip but maybe useful sometimes, for example, I got
this when I was formatting a blog into 80 columns width.

It's quite convenience if I can just goto column 80 directly by pressing a
key or a key-binding rather than stroke *f/F*, *w/e* several times.

So here it is: the magic key is **|**, yes, the separator key, not the **I**
or **l** character.

For example, to jump to 80 column in vim, just press below keys in normal mode.

{% highlight vim %}
80|
{% endhighlight %}
