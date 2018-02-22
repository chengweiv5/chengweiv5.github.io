---
layout: post
title: CentOS 7 中 systemd 会忽略 /etc/security/limits.conf
tags:
    - systemd
---

在从 CentOS 6 迁移到 CentOS 7 的过程中，可能有一些地方需要调整，最显著的地方莫过于
systemd 带来的改变，不同的管理服务的方式，不同的日志方式，设置时区，时间等等。
当然，除了这些显而易见的改变之外，有些变化并不是那么引人注目，例如这里要介绍的你可能发现曾经配置了正确的
/etc/security/limits.conf 在 CentOS 7 中却没有生效了。

如果遇到上面的问题，很可能已经导致了你的应用异常了，例如：以前配置了服务的 `nproc` 为一个非常大的数值，
结果现在发现服务不能打开更多的文件。

惊讶之余，查看服务的 `/proc/<pid>/limits` 却发现

{% highlight console %}
Max open files            1024                1024                files
{% endhighlight %}

那么，为什么服务使用 systemd 启动就没有生效呢？

根据 [redhat bugzilla 754285](https://bugzilla.redhat.com/show_bug.cgi?id=754285)，
systemd 实际上是会忽略 `/etc/security/limits.conf`，下面是 systemd 2 号人物的回答：

![systemd ignores limits.conf](/assets/images/systemd/systemd-ignores-etc-security-limits-conf.png)

而这个 bug 也标记为已解决，在 pam 包中解决，这个包包含了 `/etc/security/limits.conf`，
在这个文件中加入了如下注释：

{% highlight console %}
# /etc/security/limits.conf
#
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
#
{% endhighlight %}

注意，这里的 system services 指的是 system wide service，对于 CentOS 7 来说也就是系统的 service。

systemd 还可以对普通用户启动，使用 `--user` 参数。
