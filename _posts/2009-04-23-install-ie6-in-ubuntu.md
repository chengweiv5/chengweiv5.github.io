---
layout: post
title: Ubuntu 9.04 下安装 IE6 浏览器
tags:
  - ubuntu
---

虽然我们喜欢Ubuntu，想远离Window$，但是介于现在的国情，我们某些地方还是要用Window$，有时候仅仅是为了使用IE。
现在我们可以在Ubuntu上安装IE，步骤如下：

首先要安装Wine和cabextract。wine至少要0.99以上。不过现在的源里面的稳定版本是1.01，我使用的是winehq.com的wine1.19beta版

在下面的网址下载ies4linux

{% highlight java %}
http://www.tatanka.com.br/ies4linux/page/Installation
{% endhighlight %}

解压到一个目录，设为~/ies4linux

进入ies4linux/tools

{% highlight java %}
$ cd ~/ies4linux/tools
{% endhighlight %}

运行ies4linux

{% highlight java %}
$ ./ies4linux
{% endhighlight %}

上面默认是使用gui界面，你可以使用--no-gui参数来使用命令行模式

注意：
- 在gui下面下载时容易死掉，发生error!
- 如果你选择了安装Adobe Flash Player 9，请做好安装失败的心理准备
- 如果你安装成功发现网页的字体显示超级难看，那么是因为默认安装并没有安装字体

现在我们使用下面方式安装
{% highlight java %}
$ ./ies4linux --no-gui --no-flash --install-corefonts
{% endhighlight %}

Enjoy it！
