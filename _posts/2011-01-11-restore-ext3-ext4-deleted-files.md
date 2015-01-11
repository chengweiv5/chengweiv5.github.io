---
layout: post
title: Linux 下恢复 ext3/ext4 文件系统中删除的文件
tags:
  - linux
---

ext3/ext4文件系统都不支持恢复删除的文件，所以删除文件时还是小心为好rm -i。但是，这里有一个工具可以或许帮上一点忙，但是它及其不靠谱。下面我试了下extundelete。

#### 测试extundelete

extundelete是一个恢复ext3/ext4文件系统中被删除文件的工具，前提是被删除文件的空间未被重新分配，否则恢复不能成功。

{% highlight bash %}
#mkfs.ext4 -L undelete-testing /dev/sda8
#mount /dev/sda8 /mnt
#echo "will be deleted" > /mnt/file
#umount /mnt
#extundelete /dev/sda8 --restore-filefile
#extundelete /dev/sda8 --restore-directorydir
{% endhighlight %}

结果测试，发现extundelete工作不可靠！
