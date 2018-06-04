---
layout: post
title: 在 Linux 系统上阅读 kindle 电子书
tags:
  - misc
---

由于在 kindle 上买了一些技术类的电子书，适合在上班时间看，而工作电脑是 Linux，
没有 windows 或者 mac 环境，所以没法用 kindle app 来阅读，而 kindle cloud reader
则不对国内服务。

所以，解决 linux 上阅读 kindle 电子书就是一个问题了。

google 一下，也很容易找到几篇文章介绍怎样使用 calibre 来阅读 kindle
电子书，大概分为以下几步：

1. 安装 calibre，非常简单，按照 calibre 官方文档，一行命令就 OK
2. 安装 DeDRM 插件
3. 配置 DeDRM 插件，输入你的 kindle 序列号，如果没有 kindle 序列号，现在貌似无解
4. 导入图书

详细可以参考[这篇文章](https://www.geoffstratton.com/remove-drm-amazon-kindle-books)。

那么，我为什么写这篇博客呢？看起来没有一点营养，还不如自己写一篇汉化版的 step by
step guide 对大家的帮助大。

主要是因为我按照官方配置之后，导入 kindle azw3
电子书后，依然无法打开，报下面的错。

![locked by drm](/assets/images/calibre-drm-error.png)

各种姿势试了很多次，也怀疑是不是打开的方式不对，例如：

- 退出 calibre 重新进入
- 重新配置插件

就差重启电脑了。

后来，出于直觉，充命令行启动 calibre，并且在加入图书后，看到了如下的错误：

```
DeDRM v6.5.5: Failed to decrypt with error: No key found in 0 keys tried.
DeDRM v6.5.5: Looking for new default Kindle Key after 0.0 seconds
DeDRM v6.5.5: Running kindlekey.py under Wine
DeDRM v6.5.5: Command line: 'WINEPREFIX="/home/chengwei/Calibre Library" wine python.exe "/home/chengwei/.config/calibre/plugins/DeDRM/libraryfiles/kindlekey.py" "/home/chengwei/.config/calibre/plugins/DeDRM/libra
ryfiles/winekeysdir"'
/bin/sh: 1: wine: not found
DeDRM v6.5.5: Found and decrypted 0 key files
DeDRM v6.5.5: Ultimately failed to decrypt after 0.1 seconds. Read the FAQs at Harper's repository: https://github.com/apprenticeharper/DeDRM_tools/blob/master/FAQs.md
Running file type plugin DeDRM failed with traceback:
Traceback (most recent call last):
  File "site-packages/calibre/customize/ui.py", line 171, in _run_filetype_plugins
  File "calibre_plugins.dedrm.__init__", line 618, in run
  File "calibre_plugins.dedrm.__init__", line 568, in KindleMobiDecrypt
DeDRMError: DeDRM v6.5.5: Ultimately failed to decrypt after 0.1 seconds. Read the FAQs at Harper's repository: https://github.com/apprenticeharper/DeDRM_tools/blob/master/FAQs.md
```

乖乖，为什么在尝试用 wine 来启动 python.exe，最关键的，插件安装在了 `~/.config/calibre/plugins/DeDRM` 目录中，
而且我发现，版本为什么 v6.5.5，我刚下载了一个更新版本的。

所以，问题来了：重新添加新版本的插件，实际没有起作用，还是之前的那个版本，所以，果断进入到 `~/.config/calibre/plugins` 目录下，把 DeDRM 相关的都删掉，
除了 DeDRM 这个目录外，还有几个配置文件。

然后，重新安装新的插件，重新配置，完成之后，重启 calibre，导入 kindle azw3 文件，尝试打开，
奇迹发生了，生效了，可以打开图书了！

calibre 还可以转换图书的格式，例如：转换为 PDF，但是，正如 DeDRM 的作者说的那样，
尊重知识产权，不要将移除 DRM 保护的内容上传到公共网络上，请支持购买正版。
