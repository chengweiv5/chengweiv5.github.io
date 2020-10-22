---
layout: post
title: Google Chrome 浏览器有时候不能显示图片
tags:
  - chrome
---

最近在工作中发现，google chrome 浏览器打开 exchange web mail 查看 HTML
格式的邮件时，邮件里嵌入的图片不能显示，通过查看嵌入图片的
url，发现其实是公司内部网络的一张图片，直接用浏览器打开 url 是可以展示图片的。

那么，为什么在邮件中无法展示呢？

打开浏览器 console，发现有错误，如下所示：

![inspect error](/assets/images/chrome/inspect-error.png)

首先可以查看下错误提示中的[文档](https://blog.chromium.org/2019/10/no-more-mixed-messages-about-https.html)，
发现 google chrome 在新版本中会默认禁止 https 站点加载一些 http 内容，例如上面邮件服务本身是 https 服务，但是邮件中嵌入的图片
url 却是 http。

接下来设置为允许加载 http 内容。

首先点击站点图标，进入设置，如下图所示：

![site setting](/assets/images/chrome/site-setting.png)

然后在设置页面找到`Insecure content`，修改为`Allow`，如下图所示：

![secure setting](/assets/images/chrome/security-setting.png)

然后重新加载即可。
