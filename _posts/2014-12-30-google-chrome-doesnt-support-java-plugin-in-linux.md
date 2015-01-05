---
layout: post
title: Google Chrome 浏览器在 Linux 下不支持 Java
tags:
  - misc
---

从 Firefox 浏览器切换到 Chrome 其实也是几经波折的，以前一直是 Firefox 的粉丝，
虽然听过 Chrome 的各种好，也用过一段时间的 Chrome。但是，Firefox
一直有一些我比较不舍的地方：

- 非盈利组织开发
- 非常多的插件，例如：vimperator, sync 等
- 。。。想不起来了，毕竟已经切换了一段时间了

但是，后来发现 Chrome 的生态发展更快，以前很多没有的插件现在有了，
并且还有很多插件在 Firefox 上根本没有，或者还不完善，种种迹象已经表明越来越多的开发者已经开始向
Chrome 倾斜了。

初始我彻底迁移主要有两个因素，一个是发现 Chrome 有以前但是 Firefox 有的插件，例如：vimperator；
而且 Chrome 现在还有 Firefox 有但是不完善的插件，例如：Evernote Web Clipper；还有 Firefox
上没有的好用的插件，例如：Proxy SwitchySharp。另一个因素是自己搭建了 GoAgent 后，
能够完全无障碍的访问 Google 服务，所以浏览器的书签能够很好的同步，所以 Firefox Sync
也就没有优势了。

跑题跑完了，现在回到主题上：Linux 下的 Chrome 浏览器不支持 Java，这是我偶然发现的，
在一次使用 Dell iDRAC 的时候，发现不能启动 Virtual Console，而这个 Virtual Console
是基于 JWS(Java Web Start) 的。

在各种折腾后，安装 jre, jdk, 为 Chrome 安装 java 插件，修改浏览器配置。。。

最后发现一个事实：Linux 下 Google Chrome 从 v35 开始就不支持 Java 了，可以参考下面
[Oracle 的文档](https://java.com/en/download/faq/chrome.xml)或者
[chromium bug](https://code.google.com/p/chromium/issues/detail?id=409056)

所以，在 Linux 下，对我来说，还是有用得上 Firefox 的时候。
