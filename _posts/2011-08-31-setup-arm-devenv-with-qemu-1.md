---
layout: post
title: 使用QEMU模拟搭建ARM开发平台（一）：交叉编译Linux内核
tags:
  - qemu
---

安装需要的包，我的主机环境是debian squeeze/x86_64，首先需要安装交叉编译工具链，安装qemu模拟器，下载linux内核。

#### 安装交叉编译工具链
将下列源添加到/etc/apt/sources.list或在/etc/apt/sources.list.d/目录下新建一个文件debian-arm-toolchain.list。

{% highlight bash %}
#
# -- Emdebian cross toolchains
#
# deb http://www.emdebian.org/debian/ unstable main
# deb http://www.emdebian.org/debian/ testing main
deb http://www.emdebian.org/debian/ squeeze main
{% endhighlight %}

然后，执行

{% highlight bash %}
#apt-get update
#apt-get install gcc-4.4-arm-linux-gnueabi
{% endhighlight %}

安装工具链的方法可以参考 http://wiki.debian.org/

#### 安装qemu

{% highlight bash %}
#apt-get install qemu-system
{% endhighlight %}

#### 下载linux内核

我下载的是linux-2.6.39.2.tar.bz2。存放在$HOME/目录下。

{% highlight bash %}
$cd $HOME
$tar xjf linux-2.6.39.2.tar.bz2
$make mrproper #保证原始干净环境
$make ARCH=arm versatile_defconfig #使用versatile平台默认配置
$make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- all #编译
{% endhighlight %}

几分钟过后，arch/arm/boot/zImage即生成了，然后使用qemu模拟。之所以选择versatile平台，是因为qemu可以完全模拟。

{% highlight bash %}
$cp arch/arm/boot/zImage $HOME/versatile-zImage
$qemu-system-arm -M versatilepb -kernel versatile-zImage -m 128M
{% endhighlight %}

由于没有提供根文件系统，所以kernel会崩溃，由于找不到合适的root挂载项。下面制作一个最小的initramfs，使其可以正常运行。

{% highlight bash %}
$mkdir $HOME/versatile-initramfs
$cd $HOME/versatile-initramfs
$cat > init.c <<EOF
> #include <stdio.h>
> #include <stdlib.h>
> int main(void)
> {
> printf("hello arm\n");
> while(1);
> return 0;
> }
> EOF
{% endhighlight %}

然后，编译并且将其打包成initramfs。

{% highlight bash %}
$arm-linux-gnueabi-gcc -static -o init init.c
$rm init.c
$find . | cpio -o -H newc | gzip > ../versatile-initrd
$qemu-system-arm -M versatilepb -kernel versatile-zImage -initrd versatile-initrd -m 128M
{% endhighlight %}
