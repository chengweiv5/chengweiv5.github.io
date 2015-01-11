---
layout: post
title: 怎样编写 udev 规则
tags:
  - udev
---

#### udev和devfs
devfs是linux 2.4中管理设备的文件系统，由于它将设备文件静态创建在设备目录下，通常为/dev，在linux支持设备越来越多时，/dev/目录变得异常庞大，这是因为即使系统中没有安装相应的设备，devfs也必须创建相应的设备文件，以备设备安装时使用。
udev实现了动态设备文件管理，当设备插入时，根据内核产生的uevent事件以及sysfs信息，根据相应的规则来管理设备。

udev主要有如下优点：
- 可以对设备重命名
- 让设备具有永久的名称而不是由内核根据其安装的时间生成的名称
- 根据某个程序的执行结果来命名设备
- 管理设备的用户和组
- 在设备安装或删除时执行动作
- 重命名网络设备

#### udev rules
udev规则文件存放在 /lib/udev/rules.d,/etc/udev/rules.d/, /dev/.udev/rules.d, udev在匹配规则时，在三个目录中按照文件的字面排序，并且一直到所以规则检查完毕。
udev规则文件中的规则不能跨行，任何形式的续行符都是无效果的，规则由两部分组成，一部分是匹配，使用比较运算符，另一部分是动作，当匹配成功时执行。例如可以为设备命名，执行某个程序等。

匹配某个设备常用以下几个域来匹配
- KERNEL kernel给设备的名字，如sda, sdb1
- SUBSYSTEM 设备所在的subsystem，例如block
- ATTR 设备属性，例如size
- DRIVER 设备驱动程序

例如，要匹配某块磁盘，知道其现在在系统中的设备文件为/dev/sda，那么可以首先查看其属性。

{% highlight bash %}
#udevadm info --attribute-walk --name=/dev/sda
{% endhighlight %}

然后选择某些唯一的特性来识别它，然后建立规则，例如

{% highlight bash %}
KERNEL=="sda", SUBSYSTEM=="block", ATTR{size}=="<size>", SYMLINK+="first-disk"
{% endhighlight %}

上面的规则在sda为固定磁盘的时候工作很好，因为它总是被内核分配名为sda，如果为移动磁盘，显然就不合适了，这是往往不使用KERNEL来匹配名称，而是使用ATTR来匹配。将匹配规则添加到一个rules文件中，然后将此文件放入/etc/udev/rules.d下即可，为了让udev首先执行我们的匹配规则可以将其添加一个小的数字前缀，例如/etc/udev/rules.d/10-local-disk.rules。
然后重新启动系统或使用udevadm trigger来触发udev处理。有时，还可以通过匹配设备父层次上的设备来进行匹配，这时使用如下对应的关键字。
- KERNELS
- SUBSYSTEMS
- ATTRS
- DRIVERS

需要注意的是，它们必须都是同一个父设备的域，而不能跨越父设备，例如KERNELS=="parent-1-name", SUBSYSTEMS=="parent-parent-1-subsystem", ...则是错误的匹配规则。
在写规则时，空白字符是可以随意添加和删除的，udev并不关心空白字符。在写规则时，udev还提供了类似printf的格式化输出，例如
- %k / $kernel 设备从内核处获得的名称，例如sda
- %n / $number 设备从内核处获得的序号，往往时从设备号，例如/dev/sda1的序号为1

还可以执行简单的字符串匹配，例如*, ?, []等。
