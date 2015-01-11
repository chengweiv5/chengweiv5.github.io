---
layout: post
title: 使用QEMU模拟搭建ARM开发平台（二）：加入busybox支持
tags:
  - qemu
---

在上一篇文章中，搭建的arm平台只有一个最小化的initramfs，只是可以验证可以启动，但没有实用性，busybox是嵌入式环境中的杀手级应用，将busybox集成进initramfs变得非常实用。

首先要安装qemu, arm toolchain，还要下载busybox源码。我下载的是busybox-1.18.5.tar.bz2

{% highlight bash %}
$tar xjf busybox-1.18.5.tar.bz2
$cd busy box-1.18.5
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- defconfig
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
{% endhighlight %}

选择将busybox编译成静态文件, "Busybox Settings --> Build Options"

{% highlight bash %}
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- install
{% endhighlight %}

编译安装完成后，会在当前目录下生成_install目录，此为默认的安装目录，也可以在Busybox Settings中设置为别的目录。现在制作initramfs。

{% highlight bash %}
$cd _install
$find . | cpio -o -H newc | gzip $HOME/versatile-busybox
{% endhighlight %}

现在用qemu模拟。

{% highlight java %}
$cd $HOME
$qemu-system-arm -M versatilepb -kernel versatile-zImage -initrd versatile-busybox -m 128M -append "root=/dev/ram rdinit=/bin/sh"
{% endhighlight %}

这里如果不加rdinit=/bin/sh，那么/linuxrc将会试图挂载根文件系统，并且运行新根文件系统中的init，由于我们没有另外的真正的根文件系统，所以使用rdinit=/bin/sh，启动到sh中，敲入回车，将会出现shell命令提示符。在当前root中，没有/proc,/sys存在，所以例如mount等这些以来/proc, /sys的命令不能正常工作。在虚拟机中执行

{% highlight bash %}
#mkdir /proc /sys
#mount -t proc proc /proc
#mount -t sysfs sysfs /sys
{% endhighlight %}

也可以将其加入到启动脚本中，关闭虚拟机，然后修改versatile-busybox

{% highlight bash %}
$cd busybox-1.18.5/_install
$mkdir -p etc/init.d
$cd etc/init.d
$cat > rcS <<EOF
#!/bin/sh
>mkdir /proc /sys
>mount -t proc proc /proc
>mount -t sysfs sysfs /sys
>mdev -s
EOF
$chmod +x rcS
$cd busybox-1.18.5/_install
$find . | cpio -o -H newc | gzip > $HOME/versatile-busybox
{% endhighlight %}

现在用qemu模拟

{% highlight java %}
$ qemu-system-arm -M versatilepb -kernel versatile-zImage -initrd versatile-busybox -m 128M -append "root=/dev/ram rdinit=/sbin/init"
{% endhighlight %}

注意这里的rdinit=/sbin/init，前面之所以是rdinit=/bin/sh，是因为/sbin/init会执行/etc/init.d/rcS，而前面并没有创建这个文件，所以会打印很多错误！特别是由于没有启动mdev。

详细请参考 http://balau82.wordpress.com/2010/03/27/busybox-for-arm-on-qemu/
