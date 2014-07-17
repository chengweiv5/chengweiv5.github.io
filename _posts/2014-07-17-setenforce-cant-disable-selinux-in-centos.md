---
layout: post
title: setenforce can't disable SELinux in RHEL
tags:
  - centos
  - pitfall
---

So far I haven't experienced with SELinux a lot since I'm a Debian user, not
a fan of RHEL and its families. However, when I was working with RHEL 6.3
recently I found that **setenforce 0** can't disable SELinux at all.

That did surprised me since that's not what I found on the web, for example:
[4 Effective Methods to Disable SELinux Temporarily or Permanently](http://www.thegeekstuff.com/2009/06/how-to-disable-selinux-redhat-fedora-debian-unix/).

I did below commands

{% highlight bash %}
# setenforce 0
# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   permissive
Mode from config file:          disabled
Policy version:                 24
Policy from config file:        targeted
{% endhighlight %}

As you can see, after **setenforce 0**, the **Current mode:** is still
**permissive**.

After several another unlucky search, I found the answer from setenforce
manpage, see below.

>Use  Enforcing  or  1  to  put  SELinux  in enforcing mode.  Use
>Permissive or 0 to put SELinux in permissive mode.  You need to modify
>/etc/grub.conf or /etc/selinux/config to disable SELinux.

So there isn't such a way to disable SELinux temporarily in RHEL 6.3
at all.
