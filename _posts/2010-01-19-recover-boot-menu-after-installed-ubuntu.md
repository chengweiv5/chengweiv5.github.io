---
layout: post
title: 安装 Ubuntu-9.10 后安装 Windows7 修复启动
tags:
  - windows
---

以前修复MBR都是使用grub-0.97修复的，现在grub-1.97（即常说的grub2）释出以后，改变了很多，从修复指令，支持的模块化到使用/boot/grub/grub.cfg替代了/boot/grub/menu.lst。而且win7释出了，虽然不喜欢M$，但是还是有必要体验一下，所以在虚拟机上实验了一下。

#### 测试环境
- 主机：Ubuntu-9.10(Karmic Koala)
- 虚拟机软件：VirtualBox-3.1
- 虚拟主机：Lubuntu-9.10（即采用LXDE的Ubuntu-9.10，LXDE是国人开发的轻量级窗口管理器）
- 虚拟主机：Windows7

#### 安装步骤
##### 安装Lubuntu-9.10
首先要注意：分配给Lubuntu-9.10的磁盘空间要足够大，因为在同一个虚拟磁盘上后面要安装windows7，至少要给windows7预留近6G的磁盘空间。
在安装Lubuntu-9.10的过程中，发现了几个问题，我首先进入Live环境然后安装。

- 进入LXDE桌面环境后，双击桌面上的Install Ubuntu图标没有任何反应，或者使用右键执行也没有反应。 然后，我就打开一个终端查看了安装图标的desktop文件，找到里面的启动命令，然后在终端中执行，结果提示错误，最后是用sudo提升权限来执行才成功出现安装界面，这个bug非常严重，对于新手来说，怎样安装？
- 对非latin字符支持完全没有，包括中文，所以安装过程只能使用English。
- 安装完成以后，GDM提示可以使用GNOME/openbox会话，但是系统上根本没有完整的GNOME环境，启动不了，单独使用openbox倒是可以。


##### 安装windows7
安装windows非常简单，只是在创建虚拟机的时候注意选择“使用现有的磁盘驱动器”并且选择Lubuntu-9.10的磁盘，这样才能模拟在一个磁盘上安装ubuntu后安装windows将覆盖MBR的情况。

#### 修复grub2
安装完两个客户虚拟机后，就会发现grub2启动菜单已经不见了，现在默认只能启动windows7系统，现在方法和修复grub-0.97类似，使用LiveCD来修复。

##### 进入Ubuntu LiveCD
##### 打开一个终端，然后执行如下命令：
###### 如果boot单独分区：

{% highlight bash %}
    $sudo mkdir -p /mnt/ubuntu_root/boot
    -p参数表示如果最终目录的父目录不存在，则递归创建。
    $sudo mount /dev/sda1 /mnt/ubuntu_root
    $sudo mount /dev/sda2 /mnt/ubuntu_root/boot
    上面的/dev/sda1假设是安装ubuntu的root分区，/dev/sda2是单独的/boot分区
{% endhighlight %}

###### 如果只有root分区：

{% highlight bash %}
    $sudo mkdir /mnt/ubuntu_root
    $sudo mount /dev/sda1 /mnt/ubuntu_root
    命令的含义同上。
{% endhighlight %}

挂载完成以后，就可以修复Grub了。使用如下命令修复：

###### 单独挂载boot分区
    $sudo grub-install --root-directory=/mnt/ubuntu_root/boot /dev/sda

###### 没有单独挂载boot分区

{% highlight bash %}
$sudo grub-install --root-directory=/mnt/ubuntu_root /dev/sda
{% endhighlight %}

注意：最后的参数是磁盘而不是某个分区，因为MBR不属于分区！

通常，这一步不会出现什么错误，现在修复了MBR，但是grub2并不能引导windows7，所以还得修改/mnt/ubuntu_root/boot/grub/grub.cfg文件，添加windows7的启动入口。

{% highlight bash %}
$sudo chmod u+w /mnt/ubuntu_root/boot/grub/grub.cfg
{% endhighlight %}

添加文件的用户写权限，因为grub.cfg默认是任何用户只读。

{% highlight bash %}
$sudo vi /mnt/ubuntu_root/boot/grub/grub.cfg
{% endhighlight %}

在文件中，找到menuentry段落，在最后添加一段

{% highlight bash %}
menuentry "Windows 7 " {
    insmod ntfs
    set root=hd(0,3)
    drivemap -s (hd0) ${root}
    search --no-floppy --fs-uuid --set UUID_OF_DEV_SDA3
    chainloader +1
}
{% endhighlight %}

解释：insmod ntfs插入ntfs模块支持，如果你要使用fat32格式，那么就是insmod fat32，不过安装windows7默认就是使用NTFS格式分区
set root=hd(0,3)，grub2和grub0.97的又一个不同点，很容易弄混，因为grub0.97中的hd(x,y)中的y是从0开始的，而grub2中是从1开始的，这里我的windows7安装在/dev/sda3中，所以设置成hd(0,3)

drivemap -s (hd0) ${root}设置磁盘映射

search ...设置文件系统，这句一定不能少，否则启动的时候会提示设备错误，这里还有就是UUID_OF_DEV_SDA3怎么获得？使用如下命令：

{% highlight bash %}
$ls -l /dev/disk/by-uuid    即可。
{% endhighlight %}

chainloader +1表示将控制权交给windows7，不过这内部的机制我就不清楚了。
