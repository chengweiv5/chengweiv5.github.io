---
layout: post
title: MeeGo NetBook 系统镜像分析
tags:
  - linux
---

查看文件 /isolinux/initrd/init 做了些什么事：

- 创建和挂载虚拟文件系统，例如udev, devpts, proc, sysfs等，建立设备文件。
- 检查/etc/fstab，获取根文件系统挂载信息，如果为/dev/root，则从启动参数中获取，这也就是isolinux.cfg中传递的root=CDLABEL="LABEL" rootfstype=iso9660信息，挂载的选项rootflags为空，即默认。
- 分析根设备，添加一条udev规则，然后建立链接/dev/root指向真正的根设备。
- 启动udevd守护进程，触发udev。
- 判断是否为runlevel 1，如果是，则进入bash。
- 挂载CDROM到/sysroot。
- 查找CDROM上的根文件系统，然后losetup，将其挂载到/sysroot，由于根文件系统在CDROM中，所以是只读的，使用device mapper在内存中创建一个根文件系统的snapshot，然后将重新挂载到/sysroot，这时，根文件系统变得可写。所有的写入都通过COW机制发生在snapshot中。
- 现在/sysroot就可读写了，设置locale和复制modprobe.conf，创建udev rules。最后重新挂载/sysroot为只读。这里不要疑惑，因为在进入根文件系统执行/sbin/init后，sysvinit然后才会将根文件系统挂载为可读写。
- 杀死udevd然后chroot到根文件系统中并执行init程序。
- 使用device mapper的snapshot机制让只读文件系统可写例子。

{% highlight bash %}
#mkdir /tmp/ISO
#mount -o loop meego-netbook-VERSION.img /tmp/ISO
#mkdir /tmp/squashfs
#mount -o loop /tmp/ISO/LiveOS/squashfs.img/tmp/squashfs
#mkdir /tmp/rootfs
#mount -o loop /tmp/squashfs/LiveOS/ext3fs.img/tmp/rootfs
#mount 可以看到mount包括如下几行输出。
/dev/loop0 on /tmp/ISO type iso9660 (rw)
/dev/loop1 on /tmp/squashfs type squashfs (ro)
/dev/loop2 on /tmp/rootfs type ext3 (ro)
{% endhighlight %}

尽管显示/tmp/ISO为rw，但是由于iso9660本身是一个只读文件系统，所以不可写。现在光盘中的根文件系统已经挂载到了/tmp/rootfs中，现在要让它变得可写。

{% highlight bash %}
#umount /tmp/rootfs
#losetup -f /tmp/squashfs/LiveOS/ext3fs.img
#dd if=/dev/zero of=/snapshot.disk bs=4k count=25000
#losetup -f /snapshot.disk
#echo "0 `blockdev --getsize /dev/loop2` snapshot/dev/loop2/dev/loop3 p 8" | dmsetup create rootfs-rw
{% endhighlight %}

在上面的命令中，/dev/loop2是ext3fs.img的loopback设备，/dev/loop3是/snapshot.disk的loopback设备，可以使用 losetup -a来查看。

{% highlight bash %}
#mkdir /tmp/rootfs-rw
#mount /dev/mapper/rootfs-rw/tmp/rootfs-rw
#touch /tmp/rootfs-rw/test.file
{% endhighlight %}

可以看到，现在rootfs变成可写的了。
