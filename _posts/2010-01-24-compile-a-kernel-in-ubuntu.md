---
layout: post
title: Ubuntu 下编译的第一个内核模块
tags:
  - linux
---

今天终于弄出了第一个模块，很高兴，我辛辛苦苦弄了几乎两天，遇到太多问题了。

#### 错误的开始
最初，我从网上下载了kernel-2.6.31，然后开始编译安装这个内核。假设下载后的包放在了$HOME/Software/linux-kernel-2.6.31.tar.bz2

{% highlight bash %}
$cd $HOME/Software
$tar xjf linux-kernel-2.6.31.tar.bz2
解压后生成目录linux-kernel-2.6.31，然后开始配置编译
$cd linux-kernel-2.6.31
$make oldconfig
$make
{% endhighlight %}

过了好久好久，差不多两个小时，内核编译好了，在arch/x86/boot/下有个bzImage文件，现在编译需要的模块

{% highlight bash %}
$make modules
$sudo make modules_install
{% endhighlight %}

make modules_install会将编译的模块安装到目录/lib/modules/2.6.31/下，所以需要root权限，现在可以安装这个内核了，因为内核模块需要特定的版本支持才能插入内核中，所以开发基于这个内核版本的模块时，需要运行这个内核。

{% highlight bash %}
$sudo cp arch/x86/boot/bzImage /root/kernel-2.6.31
{% endhighlight %}

然后在/boot/grub/grub.cfg中添加一个menuentry，不过我不知道，为什么我添加后，启动时却没有显示出来，难道只能在/etc/grub.d/中修改？但是我修复被windows7覆盖的grub也是直接在/boot/grub/grub.cfg中修改的，都能生效！现在先不管这个问题。最后重启后我是手动启动的内核，在启动菜单时，按下C键，获得一个grub shell

{% highlight bash %}
grub>root (hd0,7)
{% endhighlight %}

这里是你的/boot分区，如果没有单独分区，这是/分区，注意：grub2中的分区号从1开始，也就是(hd0,7)表示/dev/sda7。

{% highlight bash %}
grub>linux /boot/kernel-2.6.31 root=/dev/sda7 ro queit splash
{% endhighlight %}

这里的root=/dev/sda7表示/分区，注意和前面的root命令区别

{% highlight bash %}
grub>boot
{% endhighlight %}

启动内核后，如果你没有安装模块，将会发现不正常，一定要确保前面执行了$sudo make modules_install

#### 正确的hello world
我照着书上写了一个hello world模块，假设位于$HOME/Kernel_module/hello/下，如下

{% highlight c %}
-------------------------hello.c------------------------------------
#include<linux/kernel.h>
#include<linux/init.h>
#include<linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
static int __init hello_init(void){
    printk(KERN_ALERT "hello from hello world/n");
    return 0;
}
static void __exit hello_exit(void){
    printk(KERN_ALERT "goodbye from hello world/n");
}
module_init(hello_init);
module_exit(hello_exit);
{% endhighlight %}

然后，编写一个Makefile，内容如下：

{% highlight bash %}
obj-m:=hello.o
{% endhighlight %}

然后使用如下命令编译

{% highlight bash %}
$make -C $HOME/Software/linux-kernel-2.6.31 M=`pwd` modules
{% endhighlight %}

编译会成功，插入模块

{% highlight bash %}
$sudo insmod ./hello.ko
{% endhighlight %}
提示invalid module format 这种情况重复了好多次，就是不明白为什么不能插入内核。

#### 正确的做法
最后，经过多次折磨后，我直接使用Ubuntu系统上的内核来编译模块。注意，这时候要运行ubuntu自带的内核，如2.6.31-17-generic
使用如下命令来编译

{% highlight bash %}
$make -C /usr/src/linux-headers-2.6.31-17-generic M=`pwd` modules
{% endhighlight %}

编译成功后，插入模块

{% highlight bash %}
$sudo insmod ./hello.ko
{% endhighlight %}
没有提示任何东西，表示成功了，但是为什么没有输出"hello from hello world"？，如果你在字符终端而不是终端模拟器下运行的话，就会输出，因为在终端模拟器下时会把内核消息输出到日志文件/var/log/kern.log中。

如果上面编译失败，如modpost不存在，那么需要首先编译这些脚本

{% highlight bash %}
$cd /usr/src/linux-headers-2.6.31-17-generic
$sudo make scripts
{% endhighlight %}
