---
layout: post
title: 让你的 Ubuntu Live CD 使用个人配置
tags:
  - ubuntu
---

我们知道Ubuntu Live CD是只读的，并且在主机上运行以后不会留下任何痕迹，但是我们可能需要在别的主机上运行我们配置过的Ubuntu。

当我们使用Live CD的时候，我们所做的修改不能保存下来，因为CD是只读的，所以如果我们想要配置我们的系统然后可以拿到其它主机上使用，这就不可能了，但是Ubuntu已经想到了这一点，利用移动设备辅助即可实现。

原理：将你的个人配置，甚至是安装了软件写入到移动设备中，当然LiveCD启动的时候必须是其他方式。

首先在Ubuntu中将你的U盘设置成特制的文件系统。

{% highlight java %}
$df -h 显示你电脑上的存储设备，当然包括了U盘

$sudo umount  /dev/sdb1      (假设你的U盘是/dev/sdb1） 卸载你的U盘

$sudo mkfs.ext3 -b 4096 -L casper-cow /dev/sdb1 将你的U盘（/dev/sdb1)制作成ext3文件系统，使用mkfs --hlep查看可以知道：-b是块大小，-L是卷标
{% endhighlight %}

设置电脑从光盘启动，插入Live CD，按下F4,进入一个特别模式，在参数后面添加
“ persistent”回车，注意单词前的空格，因为要分隔参数，所以必须使用空格。

Enjoy it!
