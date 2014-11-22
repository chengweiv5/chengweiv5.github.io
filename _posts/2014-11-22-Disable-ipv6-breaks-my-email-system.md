---
layout: post
title: Why disable IPv6 breaks my email system
tags:
  - debian
  - ipv6
  - email
---

I was following Docker guide to install it on my Debian wheezy system, so I
installed new kernel from wheezy-backports and by the way disabled ipv6 and then
have a reboot.

Everything goes well except today I found that my fetchmail doesn't work any
longer. It complains in its log like below.

{% highlight bash %}

fetchmail: reading message me@chengweiyang.cn@imap.zoho.com:69 of 69 (2540 header octets) (log message incomplete)
fetchmail: Connection errors for this poll:
name 0: connection to localhost:smtp [127.0.0.1/25] failed: Connection refused.
name 1: connection to localhost:smtp [127.0.0.1/25] failed: Connection refused.
fetchmail: SMTP connect to localhost failed
fetchmail: SMTP transaction error while fetching from me@chengweiyang.cn@imap.zoho.com and delivering to SMTP host localhost

{% endhighlight %}

I'm wondering why it was trying to connect to localhost:smtp and got refused.
Yes, apparently because there isn't anyone listen at 127.0.0.1/25. From the last
line, it seems that fetchmail first fetch mail from ZOHO and then try to
delivering to local SMTP server.

So first, I'm trying to find my answer from Google, and then Bing.com because
Google service is always stable in China. I'm lucky to find my answer that it's
true fetchmail will deliver email to local MTA once it fetched email.

And in addition, Exim4 is the default MTA in Debian wheezy, and I found that
it's not running now and it fail to start by:

{% highlight bash %}

# service exim4 start

{% endhighlight %}

And there is a paniclog in /var/log/exim4 says below

{% highlight bash %}

2014-11-22 20:36:25 socket bind() to port 25 for address ::1 failed: Cannot assign requested address: daemon abandoned

{% endhighlight %}

Apparently, it was trying to bind to a ipv6 address, ::1 is the local address in
ipv6 just like 127.0.0.1 to ipv4. So there are two solutions:

- Enable IPv6 for loopback device, lo
- Ask exim4 do not trying to bind to local ipv6 address

I did the second by do

{% highlight bash %}

# dpkg-reconfig exim4-config

{% endhighlight %}

Do not change anything except one like below

<img src="/assets/images/exim4-config.png" width="730"/>

Delete ::1 here and once it has done, exim4 restarted, and now just remove
/var/log/exim4/paniclog to avoid ALERT from **service exim4 start** like below

{% highlight bash %}

ALERT: exim paniclog /var/log/exim4/paniclog has non-zero size, mail system possibly broken

{% endhighlight %}

OK, once exim4 started, fetchmail works fine!
