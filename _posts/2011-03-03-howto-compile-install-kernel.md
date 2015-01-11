---
layout: post
title: 怎样编译安装 Linux 内核
tags:
  - linux
---

下载内核源代码，解压，假设为$HOME/linux-2.6-36

{% highlight bash %}
$cd $HOME/linux-2.6-36
$make mrproper          #确保内核源码是干净的
$make menuconfig       #这会默认使用当前内核的配置文件来生成新的.config文件，你可以定制。
$make
$sudo make modules_install    #安装新的内核模块到/lib/modules/目录下
$sudo make install                   #安装内核到/boot/目录下，包括System.map-VERSION, config-VERSION, vmlinuz-VERSION
{% endhighlight %}

生成initrd文件，这一步至关重要，在Kernel源代码的README中没有提到，initrd是内核启动初期的驱动模块，需要它来探测和加载真正的模块，否则系统不能正常启动。

{% highlight bash %}
$cd /boot
$sudo mkinitramfs -o initrd.img-2.6.36 2.6.36    #mkinitramfs是mkinitrd的新版本，内核中使用initramfs代替了initrd
{% endhighlight %}

更新grub

{% highlight bash %}
$sudo update-grub
{% endhighlight %}

重启即可

{% highlight bash %}
$sudo reboot
{% endhighlight %}
