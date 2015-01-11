---
layout: post
title: 创建 Linux 系统 ISO 的主要技术分析
tags:
  - linux
---

本文主要记录了怎样自己创建一个 Linux 系统的 ISO。

#### 安装isolinux

{% highlight bash %}
$mkdir -p isolinux-test/isolinux
{% endhighlight %}

下面复制的文件从syslinux的源码目录中来，可能需要编译。

{% highlight bash %}
$cp core/isolinux.bin isolinux-test/isolinux
$cp com32/menu/menu.c32 isolinux-test/isolinux
$cat > isolinux.cfg <<EOF
>UI menu.c32
>label isolinux-test
> menu label isolinux-test
>EOF
$genisoimage -no-emul-boot -boot-info-table -boot-load-size 4 \
>-o isolinux-test.iso -b isolinux/isolinux.bin -c isolinux/boot.cat \
>isolinux-test
{% endhighlight %}

上面命令创建一个可以启动的ISO， -c参数是可选的，如果没有指定，那么将
在ISO的根目录下生成boot.catalog文件。

{% highlight bash %}
$qemu -M pc -cdrom isolinux-test.iso -boot d
{% endhighlight %}

应该可以看到虚拟机从光盘启动，并且显示了isolinux的启动菜单。
选择启动后会发现系统不能启动，这是因为没有可以启动的系统内核。

#### 安装可启动的Linux内核

isolinux做为一个bootloader，可以很方便的配置可以启动的内核。

{% highlight bash %}
$cp /boot/vmlinuz isolinux-test/isolinux
$cp /boot/initrd isolinux-test/isolinux
{% endhighlight %}

复制kernel和initramfs文件，这两个文件可以直接从系统/boot目录下获得，
注意内核架构应该和将要模拟的一致，这里我们模拟的是pc，在qemu中默
认为i686，如果kernel在编译时已经包含了正确的initramfs，那么可以不复
制相应的initramfs文件。现在，应该修改isolinux的配置文件，让其启动
kernel。

{% highlight bash %}
$cat >> isolinux-test/isolinux/isolinux.cfg <<EOF
> kernel vmlinuz
> append initrd=initrd root=CDLABEL=isolinux-test rootfstype=iso9660 ro
>EOF
{% endhighlight %}

然后，使用genisoimage创建ISO文件。

{% highlight bash %}
$genisoimage -no-emul-boot -boot-info-table -boot-load-size 4 \
>-o isolinux-test.iso -b isolinux/isolinux.bin -c isolinux/boot.cat \
>-V "isolinux-test" isolinux-test
{% endhighlight %}

最后，用qemu模拟虚拟机

{% highlight bash %}
$qemu -M pc -cdrom isolinux-test.iso -boot d
{% endhighlight %}

启动虚拟机后，可以看到虚拟机可以正常启动，但是最后由于没有可以挂载
的根文件系统，虚拟机进入initramfs提供的shell环境。

#### 创建可以运行的ISO系统

自己创建一个可以运行的ISO系统比较复杂，主要的复杂性在系统启动阶段，initramfs要能够正确的引导系统，分析ISO文件中的内容，正确的挂载文件系统，还要使根文件系统可写，这可以使用device mapper的snapshot和aufs等来实现。
#### 让ISO可以直接写入U盘启动
syslinux提供的isobybrid工具可以让ISO直接写入U盘进行启动，直接运行

{% highlight bash %}
$isohybrid image.iso
{% endhighlight %}

即可。

#### linux-live和aufs

linux-live项目让Linux live CD/USB变得可写，可以存储用户数据，从而变得非常易用，linux-live只是一些列脚本，通过aufs来实现可写。基本思想是利用aufs可以将不同的文件系统分支挂载到同一地点，例如，ISO是只读的，如果将它和另一个可写的文件系统挂载在一起，那么对ISO的写入将会通过COW存储在另一个可写文件系统中。如果能够让挂载后的aufs成为Live CD/USB的根文件系统，那么表面上只读的Live CD/USB就变成了可以保存持久化数据的可写文件系统，许多Live CD都使用aufs和tmpfs来实现读写，但是由于tmpfs存储在内存中，所以一般的LiveCD不能将数据持久化，而Linux-Live的目标正是如此。这里有一个简单的aufs使用的例子，从aufs文档而来

{% highlight bash %}
$ mkdir /tmp/rw/tmp/aufs
# mount -t aufs -o br=/tmp/rw=rw:${HOME}=ro none /tmp/aufs
{% endhighlight %}

在/tmp/aufs中可以读写文件，但是$HOME目录却没有任何改变，而在/tmp/rw目录下可以发现所做的修改。
另外，也有一些Live CD/USB使用device mapper来实现文件系统可写，使用device mapper的snapshot机制即可，例如MeeGo。
