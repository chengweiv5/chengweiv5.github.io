---
layout: post
title: 使用QEMU模拟搭建ARM开发平台（三）——添加SCSI和MTD以及NAND flash支持
tags:
  - qemu
---

使用versatile_defconfig编译的内核不能满足要求，现在，添加SCSI磁盘，MTD以及NAND flash的支持。

#### 交叉编译linux内核
下载codesourcery的交叉编译工具链 https://sourcery.mentor.com/sgpp/lite/arm/portal/subscription?@template=lite， 选择目标OS为GNU/Linux。下载后解压，将/path/to/arm-2011.03/bin 添加到PATH中。

{% highlight bash %}
$ cd linux-2.6.39.2
$ make ARCH=arm versatile_defconfig
$ make ARCH=arm menuconfig

Kernel Features
  -> Use the ARM EABI to compile the kernel
  -> Allow old ABI binaries to run with this kernel (EXPERIMENTAL)
$ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi-
{% endhighlight %}

编译完成后，会在arch/arm/boot/目录下生成zImage，可以使用qemu来测试编译的内核是否可以启动，如果没有安装qemu，则安装

{% highlight bash %}
# apt-get install qemu-system
$ qemu-system-arm -M versatilepb -kernel arch/arm/boot/zImage -nographic -append "console=ttyAMA0"
{% endhighlight %}

可以看到，内核能启动，但是由于没有根文件系统而panic。

#### 构建根文件系统
{% highlight bash %}
$ mkdir rootfs
{% endhighlight %}

将codesourcery工具链中针对arm的lib库复制到根文件系统中，这一步是可选的，因为在下面将把busybox编译成静态链接的包。

{% highlight bash %}
$ cp -r /path/to/arm-2011.03/arm-none-linux-gnueabi/libc/lib rootfs
{% endhighlight %}

将库文件strip，减小大小，可选的。

{% highlight bash %}
$ cd rootfs/lib
$ arm-none-linux-gnueabi-strip *.so
{% endhighlight %}

提示libgcc_s.so不能识别，没关系，它是一个ASCII ld脚本，忽略即可。
现在，编译busybox-1.18.5

{% highlight bash %}
$ cd busybox-1.18.5
$ make ARCH=arm defconfig
$ make ARCH=arm menuconfig
Busybox Settings
--> Build Options
  --> Build BusyBox as a static binary (no shared libs)
{% endhighlight %}

如果不将busybox编译成静态链接程序，那么前面的复制lib工作就是必须的。

{% highlight bash %}
$ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi-
$ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- CONFIG_PREFIX=/patch/to/rootfs install // 将busybox安装到rootfs中
将rootfs打包成一个initramfs
$ cd rootfs
$ find . | cpio -o -H newc | gzip -9 > ../rootfs.img
$ qemu-system-arm -M versatilepb -nographic -kernel zImage -initrd rootfs.img -append "console=ttyAMA0"
{% endhighlight %}

可以看到，内核还是发生了panic，报错没有根文件系统，这里已经不是由kernel直接报错，而是有rootfs.img报错，因为它试图挂载真正的根文件系统。可以做如下测试

{% highlight bash %}
$ qemu-system-arm -M versatilepb -nographic -kernel zImage -initrd rootfs.img -append "console=ttyAMA0 root=/dev/ram"
{% endhighlight %}

现在，尽管最后还是panic，但是并不是没有合适的root=参数了，而是文件系统不能正确挂载。再进一步

{% highlight bash %}
$ qemu-system-arm -M versatilepb -nographic -kernel zImage -initrd rootfs.img -append "console=ttyAMA0 root=/dev/ram rdinit=/bin/sh"
{% endhighlight %}

可以发现，现在不再发生panic了，因为系统不再试图挂载真正的根文件系统。现在启动到了initramfs中，由于没有启动一个设备管理器，没有挂载proc, sysfs文件系统，busybox不能很好的工作，可以做如下测试

{% highlight bash %}
$ qemu-system-arm -M versatilepb -nographic -kernel zImage -initrd rootfs.img -append "console=ttyAMA0 root=/dev/ram rdinit=/sbin/init"
{% endhighlight %}

当busybox的/sbin/init启动后，会提示很多错误，包括没有设备文件等。现在来解决这个问题。

{% highlight bash %}
$ cd rootfs
$ mkdir -p etc/init.d
$ cat > etc/init.d/rcS <<EOF
> #!/bin/sh
> mkdir /proc
> mkdir /sys
> mount -t proc proc /proc
> mount -t sysfs sysfs /sys
> mdev -s
> EOF
$ cdmod +x etc/init.d/rcS
$ find . | cpio -o -H newc | gzip -9 > ../rootfs.img
$ qemu-system-arm -M versatilepb -nographic -kernel zImage -initrd rootfs.img -append "console=ttyAMA0 root=/dev/ram rdinit=/sbin/init"
{% endhighlight %}

现在，系统可以工作正常了，出现了提示

{% highlight bash %}
Please press Enter to activate this console. 
{% endhighlight %}

#### 编译mtd-utils

{% highlight bash %}
$ mkdir -p mtd/install
$ tar xjf zlib-1.2.5.tar.bz2
$ cd zlib-1.2.5
$ ./configure --prefix=../mtd/install
{% endhighlight %}

在Makefile中，将gcc, ar, ranlib命令加上前缀arm-none-linux-gnueabi-

{% highlight bash %}
$ make
$ make install

$ tar xzf lzo-2.05.tar.gz
$ cd lzo-2.05
$ ./configure --host=arm-none-linux-gnueabi --prefix=/root/BELS/mtd/install // --prefix需要绝对路径
$ make
$ make install

$ git clone git://git.infradead.org/mtd-utils.git
$ cd mtd-utils
$ git checkout v1.4.5 -b v1.4.5
{% endhighlight %}

现在，需要对mtd-utils打一个编译补丁，设置相应的头文件和库目录为刚才已经编译好的zlib, lzo。

{% highlight bash %}
diff --git a/Makefile b/Makefile
index 8bdba8e..ec7608f 100644
--- a/Makefile
+++ b/Makefile
@@ -1,6 +1,16 @@

# -*- sh -*-

+PREFIX = /root/BELS/mtd/install
+
+ZLIBCPPFLAGS = -I$(PREFIX)/include
+LZOCPPFLAGS = -I$(PREFIX)/include/lzo
+
+ZLIBLDFLAGS = -L$(PREFIX)/lib
+LZOLDFLAGS = -L$(PREFIX)/lib
+
+CFLAGS ?= -O2 -g $(ZLIBCPPFLAGS) $(LZOCPPFLAGS)
+
CPPFLAGS += -I./include $(ZLIBCPPFLAGS) $(LZOCPPFLAGS)

ifeq ($(WITHOUT_XATTR), 1)
@@ -12,7 +22,8 @@ else
LZOLDLIBS = -llzo2
endif

-SUBDIRS = lib ubi-utils mkfs.ubifs
+#SUBDIRS = lib ubi-utils mkfs.ubifs
+SUBDIRS = lib ubi-utils
TESTS = tests

TARGETS = ftl_format flash_erase nanddump doc_loadbios \
diff --git a/common.mk b/common.mk
index 0f3d447..fea0651 100644
--- a/common.mk
+++ b/common.mk
@@ -21,7 +21,7 @@ ifneq ($(WITHOUT_LARGEFILE), 1)
endif

DESTDIR?=
-PREFIX=/usr
+PREFIX?=/usr
EXEC_PREFIX=$(PREFIX)
SBINDIR=$(EXEC_PREFIX)/sbin
MANDIR=$(PREFIX)/share/man
{% endhighlight %}

注意：上面的PREFIX变量的值设置为zlib和lzo的安装目录！

{% highlight bash %}
$ cd mtd-utils
$ patch -p1 < /path/to/this.path
{% endhighlight %}

然后，运行如下命令编译

{% highlight bash %}
$ WITHOUT_XATTR=1 make CROSS=arm-none-linux-gnueabi-
$ WITHOUT_XATTR=1 make CROSS=arm-none-linux-gnueabi- install
{% endhighlight %}

由于ubifs有额外的依赖关系，所以这里暂不编译ubifs，补丁中有一行删除了对ubifs的编译。。

#### 支持SCSI，MTD， NAND flash

{% highlight bash %}
$ make ARCH=arm menuconfig
menuconfig需要配置的选项如下
Kernel Features
* --> Use the ARM EABI to compile the kernel
* --> Allow old ABI binaries to run with this kernel (EXPERIMENTAL)
Bus support
* --> PCI support
Device Drivers
--> Generic Driver Options
(/sbin/mdev) path to uevent helper // 这样，mdev能够帮助动态创建删除设备
* --> Memory Technology Device (MTD) support
--> Self-contained MTD device drivers
[*|M] --> MTD using block device
[*|M] --> NAND Device Support
[*|M] --> Support for NAND flash Simulator
[*|M] --> Support for generic platform NAND driver
[*|M] --> Enable UBI - Unsorted block images
[*|M] --> MTD device emulation driver (gluebi)
--> SCSI device support
[*] --> SCSI device support
[*] --> SCSI disk support
[*] --> SCSI low-level drivers
<*> --> SYM53C8XX Version 2 SCSI support // 一定要选，否则不能支持SCSI设备 
--> File systems
[*] --> Miscellaneous filesystems
[M] --> UBIFS file system support
{% endhighlight %}

现在重新编译

{% highlight bash %}
$ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi-
$ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- INSTALL_MOD_PATH=/path/to/rootfs/ modules_install
{% endhighlight %}

重新打包rootfs

{% highlight bash %}
$ cd /path/to/rootfs
$ find . | cpio -o -H newc | gzip -9 > ../rootfs.img
{% endhighlight %}

用qemu模拟

{% highlight bash %}
$ qemu-img create -f raw block.img 64M // 创建一个64M大小的磁盘
$ qemu-system-arm -M versatilepb -kernel rootfs/boot/zImage-2.6.39.2 -initrd rootfs.img -nographic -append "console=ttyAMA0 root=/dev/ram rdinit=/sbin/init" -hda block.img
{% endhighlight %}

启动过后，就可以使用block2mtd来模拟MTD设备，也可以使用nandsim模块来模拟NAND flash设备，由于添加了/sbin/mdev为uevent helper，所以会在/dev/目录下自动创建/删除相应的设备。
