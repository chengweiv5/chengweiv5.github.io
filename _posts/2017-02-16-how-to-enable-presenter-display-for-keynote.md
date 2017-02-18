---
layout: post
title: 怎样让 Keynote 在播放幻灯片的时候显示演讲者视图（Presenter Display）
tags:
  - mac
  - keynote
---

演讲者试图在做幻灯片演讲时非常有用，这样演讲者不仅能够从电脑上看到当前播放的页面，
而且还能看到下一页，以便能够组织衔接，还能看到演讲者的注释，已经播放的时间等等，
对演讲者来说非常有用。

效果如下图所示：

![Presenter Display example](/assets/images/keynote-presenter-display/presenter-display-example.jpeg)

最开始买了 MacBook 的时候，发现 keynote 在外接投影仪播放幻灯片的时候默认会显示演讲者视图，
欣喜过望；但是后来不知道怎么弄的，播放的时候就不能显示了。

后来搜索了几次，也没有找到真正的答案，都是在说打开 keynote 软件，然后在 Play 菜单中选择 Customize Presenter Display...，
但是，实际上这个只能调出配置页面而已，并不是在播放的时候就能显示演讲者视图。

后来终于弄明白了，配置的方法如下：

首先，在系统显示设置里（System Preferences -> Displays），勾选上 `Show mirroring options in the menu bar when available`，如下图所示：

![setting Display](/assets/images/keynote-presenter-display/setting-display.png)

这样，当插上外接显示设备时，系统菜单栏会出现一个图标，如下图所示：

![icon in menu bar](/assets/images/keynote-presenter-display/icon-in-menu-bar.png)

插上外接显示后，鼠标左键单击这个图标，然后从中选择 "Use As Separate Display" 即可。

这样，使用 Keynote 播放幻灯片的时候，会自动在外接显示上播放，而在电脑上显示演讲者试图。
