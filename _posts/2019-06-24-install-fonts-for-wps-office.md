---
layout: post
title: 在 Debian 环境中给 WPS-office 安装微软雅黑，仿宋/simsum，calibri 字体
tags:
  - WPS-office
  - linux
---

在 Linux（这里是 Debian 9）中使用 WPS-office
是一件非常爽的事情，基本上没有碰到什么不兼容的问题，不得不说 WPS
还是一家非常有情怀的公司了，最近也推出了 Mac 版本的 office 套件。

在 Linux 下使用 WPS-office 唯一一个比较困扰我的问题就是字体了，很多 PPT
都使用微软雅黑，仿宋之类的字体，所以当我用 Linux
打开的时候，就提示找不到字体，如下图所示：

![missing font warning](/assets/images/wps/missing-font-warning.png)

点击上面的提示后，可以看到 wps office（这里是以 wpp
为例）将使用其它字体代替找不到的字体，如下图所示：

![font replacing](/assets/images/wps/font-replacing-notice.png)

默认有替换的字体可以显示，但是如果要编辑的时候，就找不到需要的字体了。
下面介绍怎样安装字体。

为 WPS-office 安装字体很简单，只需要 2 步：

1. 用 google 搜索字体下载的地方，上面缺少的 3 种字体都提供免费下载
2. 将下载后的压缩包解压放到 `/usr/share/fonts/wps-office/` 目录下即可

完成后，重启 wps-office 即可，再次打开之前的 ppt
可以看到没有提示了，并且在字体下拉框里也能找到刚才安装的字体。
