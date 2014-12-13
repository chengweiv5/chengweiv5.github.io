---
layout: post
title: CoreOS -- 第一次体验
tags:
  - coreos
  - vagrant
---

[CoreOS](https://coreos.com/) 这个着眼于运行容器的Linux发行版，搭上了 Docker
这列高速列车；不过，现在 CoreOS 已经不再想将自己绑在一颗树上，已经宣布了自己的容器方案：
[Rocket](https://coreos.com/blog/rocket/)。

几个月前在参加一次 Docker meetup 的时候，曾有人介绍过 CoreOS，但当时由于是初次听闻，
也没有太多兴趣，只是觉得是个新东西，有些非常有趣的设计，例如：fleet,
etcd。当时看到 etcd 这个东西的时候，脑子里闪过一个问题：为什么不使用
[zookeeper](http://zookeeper.apache.org/)?

暂且不纠结这么多，先来体验一下 CoreOS 到底是一个什么样的 Linux。

首先，参考 CoreOS [Quick Start](https://coreos.com/docs/quickstart/)，基于
[Vagrant](http://www.vagrantup.com/) 和 [VirtualBox](https://www.virtualbox.org/)
搭建一个虚机环境，在虚机上运行一个 CoreOS。

安装 Vagrant 和 VirtualBox 都非常简单，这里简单介绍下我所理解的 Vagrant 和 VirtualBox。

Vagrant

- Vagrant 是一套方便管理虚机的方案，能够将对虚机的操作脚本化，从而可重复化。
- Vagrant 还可以将修改后的虚机重新打包成 Box 格式，便于迁移。
- 另外，本地的可重复化意义不是很大，所以 Vagrant 提供了 Vagrant Cloud 服务；
  从而实现便携，可迁移性。

VirtualBox 自不用多说，老牌的虚拟机产品，源自 Sun，后被 Oracle 打包购买。

现在，按照 CoreOS Quick Start 中说的那样，参照
[Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)
文档，首先克隆 coreos-vagrant 代码。

{% highlight bash %}
$ git clone https://github.com/coreos/coreos-vagrant.git
$ cd coreos-vagrant
{% endhighlight %}

代码目录中有一个 Vagrantfile 文件以及另外几个示例文件，用过Vagrant就知道，Vagrantfile
是配置 Vagrant 的脚本，vagrant 命令会解析这个脚本，然后构建虚拟机。

这里，我们只将 user-data.sample 复制为 user-data，将 config.rb.sample 复制为 config.rb。

{% highlight bash %}
$ cp user-data.sample user-data
$ cp config.rb.sample config.rb
{% endhighlight %}

然后，不做任何修改，启动虚机，如下：

{% highlight bash %}
$ vagrant up
{% endhighlight %}

启动完成后，可以使用 *vagrant status* 查看虚机状态。

{% highlight bash %}
$ vagrant status
Current machine states:

core-01                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
{% endhighlight %}

然后，通过 vagrant ssh 登陆 CoreOS 虚机。

{% highlight bash %}
$ vagrant ssh core-01
Last login: Sat Dec  6 04:06:07 2014 from 10.0.2.2
CoreOS (alpha)
core@core-01 ~ $ 
{% endhighlight %}

然后，可以查看一些系统信息，例如：文件系统信息，内存使用信息，CPU等。
可以惊奇的发现，CoreOS在这种情况下，使用的资源非常少：

- 文件系统占用36M
- 内存使用170M
- CPU使用率不足1%

可以看出，CoreOS 确实非常精简。现在，可以进一步体验 CoreOS，例如运行一些 systemd 命令，
查看一些启动服务的状态，系统启动日志等。

**需要注意的是：**这里并没有按照文档中描述的那样，使用 discovery.etcd.io/new
来创建一个 key，并且创建 3 个虚机组成一个 etcd 服务集群。这是有原因的，
由于在国内访问 discovery.etcd.io 的服务非常慢，你会发现，登陆 ssh 到 CoreOS 虚机后，
键盘输入会出现卡顿现象！当然，解决办法也是有的，作为非生产环境，我们可以在本地搭建一个单例的etcd服务，
提供发现服务，替代 discovery.etcd.io，然后再启动 3 个虚机形成一个 etcd 集群。待续。。。
