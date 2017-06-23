---
layout: post
title: 怎样清空 google-chrome 浏览器的 DNS 缓存
tags:
  - dns cache
  - google-chrome
---

有时候，在做一些服务测试，切换域名 IP 之后，想让浏览器立即生效，
但是发现可能并没有生效，而是还在访问切换前的 IP。

例如：通过修改 `/etc/hosts` 文件将域名指向了一个新的 IP，但是 google-chrome
浏览器访问域名时，还是获得原来的 IP。

这时，首先想到的就是清空 google-chrome 的 DNS cache，google 一下也确实知道了怎样清理。

方法很简单，打开 google-chrome，在地址栏中输入：`chrome://net-internals/#dns`，
打开的页面如下所示：

![google-chrome dns cache](/assets/images/google-chrome-configure-dns-cache.png)

点击 `Clear host cache`，发现并没有作用，刷新页面还是解析到原来的地址。

究其原因，是因为 http keepalive 会继续使用以前的 socket，所以，这里还需要关掉原来的 socket，
使其强制重建。

关掉 socket 的方法如下图所示：

![google-chrome flush sockets](/assets/images/google-chrome-dns-flush-socket.png)

点击 `Flush sockets` 即可。关掉已有的 sockets 之后，就好了。
