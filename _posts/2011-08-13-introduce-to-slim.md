---
layout: post
title: SLiM 一个简单好用的登录管理器
tags:
  - linux
---

SLiM是一个简单易用易配置的登录管理器，目前最新的版本为1.3.2，在Debian squeeze中的版本为1.3.1-8，对于普通不需要使用网络登录管理的用户来说，slim非常轻便好用，在debian中安装slim后就可以使用，由于我安装之前将gmd3删除了，所以slim是默认的登录管理器，如果slim不是默认管理器，需要修改/etc/X11/default-display-manager为slim，which slim可以找到slim的路径。

SLiM有非常简单容易配置的主题及配置文件，/etc/slim.conf, /usr/share/slim/themes/，修改配置文件可以选择使用的主题，而且创建一个主题也是很简单的。根据现有的配置文件和主题文件以及man手册就可以完成，还可以参考SLiM主页文档以及源代码。

我自己创建了一个海贼王的主题，如下图所示，

![theme](/assets/images/onepiece.jpg "four clients")

需要的可以从[此处](http://download.csdn.net/detail/Leisure512/3516193)
下载，然后将压缩包解压到/usr/share/slim/themes/目录即可，然后修改/etc/slim.conf配置文件。

SLiM 1.3.2及以前版本中有一个bug，就是默认的窗口管理器不能设置，根据/etc/slim.con中的描述，默认窗口管理器为第一个。实际上，在源码中，slim默认并没有获取第一个session，而是空值，也就是除非每次登录使用F1键选择session，否则就会采用系统默认的窗口管理器，当然，除非你从~/.xinitrc中设备。

这个bug我已经提交patch给slim邮件列表，暂时还没有被接受。
