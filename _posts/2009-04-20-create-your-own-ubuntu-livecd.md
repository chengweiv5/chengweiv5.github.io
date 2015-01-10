---
layout: post
title: 打造自己的 Ubuntu Live CD
tags:
  - ubuntu
---

#### 基本要求

创建磁盘映像文件需要最多5GB（不是吓人的）的swap和至少3GB的磁盘空间。 添加SWAP空间，当磁盘映像文件压缩的时候，会有两个copy要存在内存上，所以必须有大的SWAP。当然，没有人会分出5GB的SWAP空间，所以我们有另外的方法，如果你的磁盘上有另外的5GB空间如（/tmp下）使用dd命令可以避免格式化就获得额外的SWAP

{% highlight java %}
$ sudo dd if=/dev/zero of=/tmp/swap bs=1M count=5000
{% endhighlight %}

创建SWAPFILE会花很长时间，所以要有耐心，然后你就可以激活你的SWAP

{% highlight java %}
$ sudo mkswap /tmp/swap
$ sudo swapon /tmp/swap
{% endhighlight %}

现在我们添加了新的SWAP,所以现在的SWAP一共就有原来的加上5GB。

#### 获取和安装工具软件

{% highlight java %}
$ sudo apt-get install cloop-utils mkisofs(xcdroast) squashfs-tools
{% endhighlight %}

#### 准备标准的Live CD映像文件
为了避免Unicode问题，执行

{% highlight java %}
$ export LC_ALL=C   这行字你会在使用sort --help是发现！
{% endhighlight %}

#### 挂载映像文件
{% highlight java %}
$ mkdir ~/mnt   在主文件下创建一个目录
$ sudo mount ubuntu-i386.iso -o loop
{% endhighlight %}

挂载映像文件到~/mnt目录，你也可以挂载到其他目录，如你的cdrom目录。

#### 复制挂载后的映像文件

复制除了/casper/filesystem.squashfs文件，这个文件将要单独解压缩。使用rsync命令可以达到目的
{% highlight java %}
$ rsync --exclude=/casper/filesystem.squashfs -a ~/mnt/ ~/extracted_cd
{% endhighlight %}

解释：查看rsync的帮助文件可知，它是一款强大的同步软件，可以同步远程主机，--exclude表示同步时出去后面路径包含的文件，-a表示存档模式，后面的两个参数为：src和destination
测试成功！

#### 解压filesystem.squashfs
squashfs是一种只读文件系统，为了使用它，需要加载squashfs内核模块
{% highlight java %}
$ sudo modprobe squashfs
{% endhighlight %}

然后你就可以挂载和复制它了

{% highlight java %}
$ mkdir squash
$ sudo mount -o loop ~/mnt/casper/filesystem.squashfs squash
$ sudo cp -a squash extracted_fs
{% endhighlight %}

解释：
- 第一句建立一个目录
- 第二句挂载squashfs到目录
- 第三句复制文件到extracted_fs,-a表示存档文件，注意会花比较久时间，要有耐心。现在查看你的squash，你会发现有系统文件夹，成功！

#### 卸载原来的映像文件
{% highlight java %}
$ sudo umount ~/mnt
{% endhighlight %}

#### 设置你的目标文件系统
{% highlight java %}
$ sudo mount -t proc proc ~/extracted_fs/proc
$ sudo mount -t sysfs sysfs ~/extracted_fs/sys
{% endhighlight %}

解释：加载系统模块到proc和sys文件夹下，开始是空目录，但是执行命令后你可以查看一下，会发现什么呢？

现在我们前面创建的extracted_fs是root权限的，所以为了方便操作，我们将我的home绑定到extracted_fs/home中（不要担心不会另外占用空间）。因为我们可能会需要很多home中的文件.

{% highlight java %}
$ sudo mount -o bind /home ~/extracted_fs/home
{% endhighlight %}

#### 使用chroot进入文件系统映像
{% highlight java %}
$ sudo chroot ~/extracted_fs/ /bin/sh
{% endhighlight %}

查看chroot的帮助，可以知道，chroot是更改更目录的命令，/bin/sh可要可不要，因为默认就是/bin/sh
然后会发现当前的提示符变成＃，即有root权限，并且你使用cd /会发现就是在你新设置的根目录下

#### 删除不需要的软件包
因为如果你想要添加自己的软件，当然CD只有这么大，你就必须要删除一些你不要的软件包
{% highlight java %}
$ dpkg-query -W --showformat='{Installed-Size;10} ${Package}//n' | sort -gr | less
{% endhighlight %}

解释：使用less显示出已经安装的包，按大小排序，上面使用了管道，需要注意的是：千万小心你删除的包，因为有些可能是Ubuntu运行必须的。
参考：http://svn.gnome.org/viewvc/livecd-project/trunk/livecd.conf?revision=45&view=markup
可以看到你不能改变的都有哪些包，当你确定要删除的包后，使用下面的命令

{% highlight java %}
$sudo dpkg -r --purge packagename
{% endhighlight %}

#### 安装你自己的包
当我们进入chroot后，使用apt-get install就不能安装软件了，因为这是连DNS都不会解析。

另一种方法是运行新立得软件管理器，标记你要安装的包，然后选择“文件”》生成包下载脚本，然后你可以在chroot中访问下载脚本来下载。但是，这个方法一样不能工作，因为执行脚本时，不能解析主机！但是我们也可以取巧，使用ping命令解析主机后，修改脚本文件的主机为IP地址。

#### 定制/home目录
从Live CD启动时，每次都会从/etc/skel加载，所以如果你要有文件放入/home中，就将它放入skel中，现在我们查看skel，发现只有一个Examples

#### 退出chroot
{% highlight java %}
# exit
$ sudo umount ~/extracted_fs/home
$ sudo umount ~/extracted_fs/sys
$ sudo umount ~/extracted_fs/proc
{% endhighlight %}

#### 现在是时候重新压缩你的文件系统的时候了
但是，最好列出一个清单来显示你所做的更改

{% highlight java %}
$ sudo -s    切换到root
{% endhighlight %}

生成manifest

{% highlight java %}
# chroot extracted_fs dpkg-query -W --showformat='${Package} ${Version}//n'>extracted_cd/casper/filesystem.manifest
{% endhighlight %}

退出root

#### 重新打包你的文件系统
{% highlight java %}
$ sudo mksquashfs extracted_fs extracted_cd/casper/filesystem.squashfs
{% endhighlight %}

同样，上面会花费几分钟时间

#### 生成校验和文件

{% highlight java %}
$ cd ~/extracted_cd
$ find . -type f -print0 | xargs -0 md5sum>md5sum.txt
{% endhighlight %}

解释：find是查找命令，"."表示当前目录，-type f表示类型为文件

#### 生成ISO文件

这里只生成x86(i386)和x86_64(amd64)的ISO文件

{% highlight java %}
$ sudo mkisofs -r -V "Custom Ubuntu 8.10 Live CD" \
-cache-inodes \
-J -l -b isolinux/isoliux.bin \
-c isolinux/boot.cat -no-emul-boot \
-boot-load-size 4 -boot-info-table \
-o custom-ubuntu8.10-i386.iso extracted_cd
{% endhighlight %}

查看mkisofs的帮助文件，可知

{% highlight java %}
-cache-inodes               Cache inodes (needed to detect hard links)
-b FILE, -eltorito-boot FILE
                              Set El Torito boot image name
 -c FILE, -eltorito-catalog FILE
                              Set El Torito boot catalog name
-J, -joliet                 Generate Joliet directory information
-l, -full-iso9660-filenames Allow full 31 character filenames for ISO9660 names
-r, -rational-rock          Generate rationalized Rock Ridge directory information
-V ID, -volid ID            Set Volume ID
-boot-load-size #           Set numbers of load sectors
{% endhighlight %}

Enjoy it!
