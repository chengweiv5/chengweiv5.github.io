---
layout: post
title: 禁止 debian 中的各种 beep
tags:
  - udev
---

在debian squeeze中，有各种讨人厌的beep，特别是从gnome切换到awesome后，上网找了很久，终于把所有的beep都禁止了。

#### 禁止登录窗口beep（GDM3）
编辑/etc/gdm3/greeter.gconf中的行，defaults/desktop/gnome/sound/event_sounds true->false，将其值修改为false。

#### 禁止console的beep
console的beep最烦人，连TAB补全也会产生beep。

{% highlight bash %}
$cat >> ~/.inputrc <<EOF
$>set beep-style none
$>EOF
{% endhighlight %}

#### 禁止X的beep
当我使用Mutt读邮件的时候，很多提示都会产生beep，例如到达邮件末尾。

{% highlight bash %}
$set b off
{% endhighlight %}

然后再启动mutt就可以了，也可以将其添加到~/.bashrc中，或者直接添加到bash的系统配置文件/etc/profile中。

#### 禁止kernel中的pcspkr

{% highlight bash %}
#lsmod | grep pcspkr
#modprobe -r pcspkr
{% endhighlight %}

或者直接加入黑名单

{% highlight bash %}
#cat >> /etc/modprobe.d/blacklist.conf <<EOF
#>blacklist pcspkr
#>EOF
{% endhighlight %}

这个世界安静了！
