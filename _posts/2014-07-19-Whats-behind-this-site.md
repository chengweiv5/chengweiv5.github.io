---
layout: post
title: What's behind this site
tags:
  - misc
---

After several weekends cook, finally I have my blog site up and running. It's
simple, small but to me it's good enough to hosts my humble words.

This small site consists of many great stuff which I'm appreciated.

- [github pages](https://pages.github.com)
- [jekyll](http://jekyllrb.com)
- [solar-theme-jekyll](https://github.com/redwallhp/solar-theme-jekyll)
- ...

Especially thanks to [derekwyatt.org](http://derekwyatt.org/), where I saw
[the solar-theme-jekyll](https://github.com/redwallhp/solar-theme-jekyll) first
time.

I'm not going to say how I cooked out this site, since it's not hard for a
programmer, all the components have detail documents, guides can be found
everywhere.

The single part I feel not so clear is how it works to redirect to my own
domain? say *www.chengweiyang.cn*.

To do that, first I have to register my own domain from an domain registry
organization, in China, the most famous one is [www.net.cn](www.net.cn) but I
choose [another one](www.xinnet.com), why? Because it's cheaper. :-).

Once you have a domain registered, you need an public IP address to host your
site, so the others can find you by domain name. The public IP address here
names **A record**, address record. So in a traditional way, I have a buy a
**VPS**(Virtual Private Server) to hosts my web pages. And fill up my domain
name's A record with that VPS IP address.

However, that isn't the way how I did. Thankful the [github
pages](pages.github.com) service, It works like a VPS and the thing attracted me
most is I can update my site with git rather than upload my site through like
ftp every time when I wrote a new blog, the traditional VPS way.

So what the magic here?

- when I access [my github pages](http://chengweiv5.github.io) it will
	redirect to [my domain](http://www.chengweiyang.cn)
- when I access [my domain](http://www.chengweiyang.cn) it will show the web
	pages hosted on github.
- all the pages hosted by github
- I didn't have any VPS on github, so no a A record

The magic is use a [CNAME](http://en.wikipedia.org/wiki/CNAME_record) and only a
CNAME record, CNAME record in simple is an alias for a domain, the DNS name server
will point to the alias if you access the domain. For me, I have a CNAME record
registered for my domain (tips: CNAME record can be configured with your domain
provider, always you can do that from a tool or a web page.) like below:

{% highlight bash %}
www.chengweiyang.cn    chengweiv5.github.io
{% endhighlight %}

So all the access requests to www.chengweiyang.cn will redirect to
chengweiv5.github.io, where the web contents located.

So how about the reverse? github pages understand CNAME well and will redirect to
my domain when a request to github pages directly. See
[here](https://help.github.com/articles/adding-a-cname-file-to-your-repository)
for detail. So that's it.

Finally, if you like this site, you can even fork me on
[github](https://github.com/chengweiv5/chengweiv5.github.io) and put your posts
into *\_posts* directory, you got it.
