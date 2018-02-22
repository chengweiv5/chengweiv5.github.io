---
layout: post
title: 在 Awesome 窗口管理器中开启锁屏快捷键
tags:
  - linux
---

Awesome 是一个非常棒的平铺窗口管理器，非常适合在高分辨率大屏显示器下工作，
强大的快捷键组合能够解放大部分鼠标操作，大大减少手离开键盘的频次，提高工作效率。
自从使用了 Awesome 之后，就没有在使用过其它的窗口管理器，也已经记不清到底使用
Awesome 多少年了。

使用 Awesome 之后，很多 GNOME 的小工具就不能很好的工作了，特别是 GNOME 3.0
之后，很多小工具和 GNOME 绑定得越来越紧密了，这里介绍下怎样在 Awesome
下开启快捷键锁屏（Ctrl + Alt + l）。

首先，需要安装一个锁屏工具，我一直使用 Debian 系统，所以默认的 GNOME
环境就已经安装 gnome-screensaver 了，可以直接使用。

其次，需要在进入系统时，启动 gnome-screensaver，这样才能响应锁屏操作，
启动 gnome-screensaver 可以将下面的语句添加到 Awesome 的配置文件中
`~/.config/awesome/rc.lua`。

{% highlight console %}
awful.util.spawn_with_shell("/usr/bin/gnome-screensaver")
{% endhighlight %}

当然，也可以在其它地方启动 gnome-screensaver，例如 `~/.xinitrc`。

最后，绑定快捷键操作，在 `~/.config/awesome/rc.lua` 中的 `globalkeys` 的定义中，
添加一行：

{% highlight console %}
awful.key({ "Mod1", "Control" }, "l",     function () awful.util.spawn("gnome-screensaver-command --lock") end),
{% endhighlight %}

这行语句定义了一个快捷键组合：Alt + Ctrol + l，当按下组合键时，会执行命令
`gnome-screensaver-command --lock`，即启动锁屏。

**这里需要特别注意的是：Alt 键在 Awesome 中定义为 Mod1 而非 Alt**
