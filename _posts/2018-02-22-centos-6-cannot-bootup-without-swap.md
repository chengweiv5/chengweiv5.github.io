---
layout: post
title: CentOS 6 没有 swap 分区不能启动
tags:
  - centos
---

最近刚发现一个 centos 6.x（测试过 6.8, 6.9）的
bug：如果在安装系统的时候，手动分区，并且没有创建 swap 分区，那么系统不能启动。

重现 bug 很简单，用 virt-manager 创建一个虚机，然后下载 centos 6.8/6.9 的镜像，
我试过 minimal image 和 bin-dvd image，均有这个 bug。

**安装的时候手动分区，并且，只创建一个分区，挂载到 / 下。**

安装完成后，重启系统，会发现系统不能启动，如下图所示：

![ centos 6 can not boot ](/assets/images/fail-to-boot.png)

下面尝试解决这个问题。

启动的时候，修改启动参数，添加 kernel 参数 rdshell，这样启动失败会进入 dracut
debug shell，然后，查看 /sysroot/etc/fstab（/sysroot
是真正根分区挂载的地方），没有分配 swap 分区，在 fstab
中也没有，所以感觉不是挂载的问题。

在 dracut debug shell 中查看 dmesg，找到了答案，如下图所示：

![ dmesg ](/assets/images/fail-to-boot-dmsg.png)

发现有一个 fatal error，是因为加载 selinux policy 失败。在 kernel 参数中添加
selinux=0 禁止 selinux，就能启动成功。

所以，这个问题从表现看是没有 swap 分区导致的，因为如果创建了 swap
分区，则系统能够正常启动。

但是，真正原因去和 selinux 有关，而 centos 6.x 在安装的时候，并不能配置
selinux，所以这应该是一个 bug。

在 centos 7.x 中，不创建 swap 分区并不会触发这个 bug。
