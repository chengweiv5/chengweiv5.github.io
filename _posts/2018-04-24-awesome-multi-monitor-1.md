---
layout: post
title: 在 awesome wm 下使用两个显示器
tags:
  - awesome
---

awesome wm 使用了快十年了，至从使用了 awesome 之后，就没有再用过其它
wm，操作系统依然使用 debian。

最近工作上配的笔记本用起来比较费劲，所以外接了一个大一些的显示器。由于平时开会也不少，
所以希望的应用场景是：插上外接显示器，笔记本上的所有内容自动切换到外接显示器上，当拔掉外接显示的时候，
所有内容自动转移到笔记本显示屏上。

调研了一天，发现目前还达不到上面的目的，暂且先记录下现在能够用的，待日后有时间再解决剩下的。

这里先列一下关键难点：

- awesome system tray 只能同时在一个 screen 上，根据 awesome FAQ 的说法，这是
  X11 server 的限制，后面会介绍绕过的办法
- 切换 screen 的时候，内容能够发送到另一个 screen
  上，但是所有内容会被发送到同一个 awesome screen tag
  上，也就是一个桌面上，而不是原先所在的 tag
- Dell 显示器在插到笔记本之后，会自动进入休眠状态

目前的状态：笔记本显示器和外接显示器默认做为两个独立的 screen 使用，两个 screen
位于左右两侧，可以通过鼠标来切换 focus screen 或者通过快捷键 `modkey + ctrl + j`。

另外，通过 `modkey + o` 快捷键可以发送 window 到另一个 screen。

另外，还有一个 [awesome recipe](https://awesomewm.org/recipes/xrandr/)
支持快捷键配置多个 screen 的布局方式，例如：只用某一个 screen，两个 screen，谁在左，谁在右等。

现在，来看下怎样绕过 system tray 的问题，awesome systray 会跟随 primary
screen，所以，只需要修改 primary screen，systray 会自动切换。

这样子，稍微修改下上面的 awesome recipe，让它在支持 2 个 screen 的时候，将第一个
screen 配置为 primary screen 即可。

剩下的问题是：当只使用一个 screen 的时候，切换 screen 的时候，所有窗口和 systray
自动发送到另一个 screen 去，且保持布局不变。

这部分下次再解决，其实还需要解决显示器休眠问题，以及自动切换的问题（可以通过
udev 来实现），当外接显示器拔掉的时候，自动将所有内容和 systray
保持布局，发送到笔记本上；插上外接显示的时候则反之。
