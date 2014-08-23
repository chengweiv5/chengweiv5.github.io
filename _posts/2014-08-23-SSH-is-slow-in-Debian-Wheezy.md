---
layout: post
title: SSH is Slow in Debian Wheezy
tags:
  - debian
  - ssh
---

I'm using Debian for years and glad to upgrade from Squeeze to Wheezy once it
was been released. However, I noticed that ssh costs a lot of time to login to
remote host from Debian Wheezy but much faster on my MacBook Air.

This surprised me a little and I run 'ssh' into verbose mode and found something
different.

From my Debian Wheezy.

{% highlight c %}

# ssh -v remote-host
...
debug1: Authentications that can continue:
publickey,gssapi-keyex,gssapi-with-mic
debug1: Next authentication method: gssapi-keyex
debug1: No valid Key exchange context
debug1: Next authentication method: gssapi-with-mic
debug1: Unspecified GSS failure.  Minor code may provide more information
Credentials cache file '/tmp/krb5cc_1000' not found

debug1: Unspecified GSS failure.  Minor code may provide more information
Credentials cache file '/tmp/krb5cc_1000' not found

debug1: Unspecified GSS failure.  Minor code may provide more information


debug1: Unspecified GSS failure.  Minor code may provide more information
Credentials cache file '/tmp/krb5cc_1000' not found

debug1: Next authentication method: publickey
debug1: Offering RSA public key: /home/chengwei/.ssh/id_rsa_fdo

{% endhighlight %}

From my MacBook Air.

{% highlight c %}

# ssh -v remote-host
...
debug1: Authentications that can continue:
publickey,gssapi-keyex,gssapi-with-mic
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /home/chengwei/.ssh/id_rsa_fdo
...

{% endhighlight %}

SSH login to remote-host from Debian Wheezy costs about another 20 seconds
that's because it first try to get authenticated with
[GSSAPI](http://en.wikipedia.org/wiki/Generic_Security_Services_Application_Program_Interface).

From the ssh_config(5) man page, I see *GSSAPIAuthentication* disabled by default,
but that's not true in Debian Wheezy, so generally this is an inconsistency from
document and behavior. So I filed a bug to Debian Community.

The solution is very simple: just disable *GSSAPIAuthentication* in your
ssh_config, you can do that in system-wide as set below in /etc/ssh/ssh_config

{% highlight c %}

GSSAPIAuthentication no

{% endhighlight %}

Or in user wide by setup in *$HOME/.ssh/ssh_config* Or even by ssh command line
option.
