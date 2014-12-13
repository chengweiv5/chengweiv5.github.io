---
layout: post
title: CoreOS -- etcd 初探
tags:
  - coreos

---

在我第一次按照 [Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)
中所述步骤搭建了第一个具有 3 个 CoreOS 的小集群后，发现使用 *vagrant ssh*
登陆任意一台 CoreOS 都非常慢，这确实让我摸不着头脑，而且非常不幸没有在网上找到足够的信息来解决：
为什么 *vagrant ssh* 登陆 CoreOS 后键盘会出现卡顿，而且非常严重，
基本上终端反应的速度跟不上键盘敲击的速度。

后来，在github上找到一个[原因](https://github.com/coreos/bugs/issues/79)，
但是只是解决了ssh登陆速度慢的问题，和登陆后卡顿没有关系，而且这个问题在我使用的coreos-alpha 509.1.0中已经解决。
所以，这个问题肯定另有原因。后来发现是 etcd 服务导致的。所以在上一篇
[CoreOS -- 第一次体验]({% post_url 2014-12-06-CoreOS-fisrt-step %})
中故意避过，以免其他人遇到同样的问题。

这里总结一下这些天探索下来对 etcd 的一些体会。

###搭建本地 etcd 服务，为 CoreOS etcd 高可用性服务集群做 bootstrap 服务

正如 [Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)
中介绍的那样，启动 3 个 CoreOS 虚机，使用 discovery 机制让 3 个虚机上的 etcd
服务组成一个服务集群，从而可以让其它更多的 CoreOS 使用高可用性的 etcd 服务。但是，
由于网络问题，会导致 ssh 卡顿，所以这里我们在本地搭建一个单例的 etcd 服务，
从而替代 discovery.etcd.io 来为 3 个 etcd 服务提供 bootstrap 服务。

方法非常简单，可以将这个 etcd 运行在本地主机或者另一个 CoreOS 中，只要将要启动的 3
台 CoreOS 虚机能够连接即可，这里选择将 etcd 直接运行在主机上，etcd 是 CoreOS
的一个主要构件之一，也能运行在普通的 Linux 发行版上。

首先，从 [github](https://github.com/coreos/etcd/releases)上下载 etcd，
这里我使用的是最新的稳定版本 v0.4.6。

{% highlight bash %}
$ curl -L https://github.com/coreos/etcd/releases/download/v0.4.6/etcd-v0.4.6-linux-amd64.tar.gz -o etcd-v0.4.6-linux-amd64.tar.gz
$ tar xzvf etcd-v0.4.6-linux-amd64.tar.gz
$ cd etcd-v0.4.6-linux-amd64
$ ./etcd -name="single-etcd-service"
{% endhighlight %}

至此，etcd 服务已经启动了，由于只有一个实例，etcd 不能提供高可用性服务，
一旦这个服务失败，则整个 etcd 服务失败；值得一提的是：etcd 实现了 Raft 分布式协议。所以，
当只有一个 etcd 实例时，它很快将会自动变成整个服务的 leader。这里推荐一个详解
[Raft协议的动画](http://thesecretlivesofdata.com/raft/)，非常精彩！

现在，可以尝试一下 etcd 的 key/value 存取，例如：

{% highlight bash %}
$ ./etcdctl set mykey "this is awesome"
$ ./etcdctl get mykey
{% endhighlight %}

可以通过 netstat 查看 etcd 监听的端口，如下：

{% highlight bash %}
# netstat -tlnp | grep etcd
tcp6       0      0 :::4001                 :::*                    LISTEN 15469/etcd
tcp6       0      0 :::7001                 :::*                    LISTEN 15469/etcd
{% endhighlight %}

etcd 需要监听两个端口，一个对外提供服务，默认为 4001；一个为形成 etcd 高可用性集群内各个
etcd 之间通信，默认为 7001。

###启动 3 个 CoreOS，提供 etcd 高可用性服务

这里我们参考
[Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant/)
唯一需要修改的地方是：修改 user-data 文件，将其中的 discovery: 行指向我们自己的 etcd 服务，
为了完整性，修改后的 user-data 文件如下：

{% highlight bash %}
#cloud-config

coreos:
  etcd:
      # generate a new token for each unique cluster from https://discovery.etcd.io/new
      # WARNING: replace each time you 'vagrant destroy'
      discovery: http://172.17.8.1:4001/v2/keys/7f15cac7-dd14-4a5c-ac36-0ef612e3b075
      addr: $public_ipv4:4001
      peer-addr: $public_ipv4:7001
  fleet:
      public-ip: $public_ipv4
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start

{% endhighlight %}

注意上面 **discovery: 172.17.8.1**，这里使用了 VirtualBox bridge 的 IP，
这个虚拟设备为所有的虚机提供网络服务，你可以看到每个虚机都有一个 IP 为 172.17.8.x；
这样确保虚机中的 etcd 能够连接上主机上的 etcd，从而使用其提供的 discovery 服务来发现其它 etcd 服务。
当然，VirtualBox 默认还为每个虚机提供了 NAT 网卡设备，所以可以访问主机的真实 IP 地址，
所以这里也可以将 172.17.8.1 替换成主机的真实 IP 地址。
*/v2/keys/7f15cac7-dd14-4a5c-ac36-0ef612e3b075* 中的最后一部分为一个 uuid，可以随便
使用 **uuidgen** 命令生成，也可以随意使用一个合法的 etcd key，例如：mykey，请确保
key 的唯一性，否则有可能加入到其它的 etcd 集群中。

现在，使用 *vagrant up* 启动 3 个虚机即可，启动完成后，可以查看主机上的 etcd 中的键值有何变化。

{% highlight bash %}
$ ./etcdctl ls --recursive /
/7f15cac7-dd14-4a5c-ac36-0ef612e3b075
/7f15cac7-dd14-4a5c-ac36-0ef612e3b075/e956fcd6ca22403b9b8d35ef7f6b4988
/7f15cac7-dd14-4a5c-ac36-0ef612e3b075/f9b8add612ea41b58049c8896ca6e75c
/7f15cac7-dd14-4a5c-ac36-0ef612e3b075/4de7ad5dd0604bbdb469b18d56659aa3
{% endhighlight %}

可见，在我们刚才提供的 key **7f15cac7-dd14-4a5c-ac36-0ef612e3b075** 下面生成了
3 个新的 key，现在查看一下它们的值都是什么。

{% highlight bash %}
$ etcdctl get /7f15cac7-dd14-4a5c-ac36-0ef612e3b075/e956fcd6ca22403b9b8d35ef7f6b4988
http://172.17.8.102:7001
$ etcdctl get /7f15cac7-dd14-4a5c-ac36-0ef612e3b075/f9b8add612ea41b58049c8896ca6e75c
http://172.17.8.103:7001
$ etcdctl get /7f15cac7-dd14-4a5c-ac36-0ef612e3b075/4de7ad5dd0604bbdb469b18d56659aa3
http://172.17.8.101:7001
{% endhighlight %}

可以看到，分别为 3 个虚机中 etcd 监听的地址和端口，正如前面所说，7001 是 etcd
默认监听的端口，各个 etcd 之间通过这个端口来通信，组成高可用性的 etcd 服务集群。

现在，可以任意登陆一个 CoreOS 虚机，
例如：

{% highlight bash %}
$ vagrant ssh core-01
{% endhighlight %}

而且，可以发现，登陆后将不会出现任何
[按键卡顿的现象]({% post_url 2014-12-06-CoreOS-fisrt-step %})

这里，只是介绍了一下 etcd 的初步概念，使用一个单例的 etcd 服务来提供 etcd 服务集群
bootstrap 服务，实际上在生产环境中，我们也可以采用其它方式，例如：为每个 etcd
实例配置其它各个 etcd 将要监听的地址和端口，只是这种方式在配置上稍嫌麻烦，特别
是当 etcd 集群由多个（例如：5 个，7 个）etcd 实例组成时。

**注意：**这里有一个理解上的难点：主机上的 etcd 是否只提供 etcd 集群的 boostrap
服务呢？也就是一旦 etcd 集群启动完成，使命就结束了呢？按照 Raft 协议，显然是
这样的，集群中的各个成员已经可以互相通信了，leader 也可以正常行驶职责。但是，
etcd 集群是一个动态变化的过程，也就是：这个集群的大小可以改变，现在是 3 个，
那么可能会有新的 etcd 服务加入进来，变成 4 个，5 个。所以现有的 etcd
集群会持续和主机上的 etcd discovery 服务保持心跳，来发现是否有新的 etcd 服务实例加入。
所以，主机上的单例 etcd 并没有完成使命，这也是为什么推荐使用 discovery.etcd.io 提供的
etcd 服务的原因。

现在，我们的 etcd 高可用性服务集群已经启动了，可以对外提供服务了；现在，
可以启动任意多个 CoreOS 虚机，让它们连接到这个 etcd 服务集群，充当 worker，所以，
它们可以不再需要启动 etcd 服务。

###启动 CoreOS worker 虚机

这里，还是从 [coreos-vagrant.git](https://github.com/coreos/coreos-vagrant)
出发，这里考虑到主机的性能，只启动一个 worker 虚机。

{% highlight bash %}
$ git clone https://github.com/coreos/coreos-vagrant.git
$ cd coreos-vagrant
$ cp user-data.sample user-data
$ cp config.rb.sample config.rb
{% endhighlight %}

首先，修改 Vagrantfile，让新启动的虚机 IP 不要和以前的冲突，修改如下行：

{% highlight bash %}
-      ip = "172.17.8.#{i+100}"
+      ip = "172.17.8.#{i+200}"

{% endhighlight %}

将其中的 100 修改为 200，这样，新启动的虚机 IP 将从 172.17.8.201 开始，
因为 172.17.8.{101,102,103} 已经被前面的 etcd 服务集群占用。

然后，修改 user-data，禁止 etcd.service，并且让 fleet 服务指向已经搭建好的 etcd
服务集群。修改后的 user-data 内容如下：

{% highlight bash %}
#cloud-config

coreos:
  fleet:
    public-ip: $public_ipv4
    etcd_servers: "http://172.17.8.101:4001,http://172.17.8.102:4001,http://172.17.8.103:4001"
  units:
    - name: fleet.service
      command: start
    - name: docker-tcp.socket
      command: start
      enable: true
      content: |
        [Unit]
        Description=Docker Socket for the API

        [Socket]
        ListenStream=2375
        Service=docker.service
        BindIPv6Only=both

        [Install]
        WantedBy=sockets.target

{% endhighlight %}

注意，这里修改了 fleet 服务中关于 etcd 的配置，fleet 是另一个 CoreOS 的重要组件，
它负责分布式的调度任务到集群中的各个结点，而 fleet 使用了 etcd 服务来存储数据，
例如：已知的各个任务信息，集群结点信息，fleet engine leader 等等。

现在，登陆 3 台 etcd 服务虚机中的任意一台，即可查看当前集群中所有的结点，即所有运行了
fleet 服务并且加入了集群的结点。

{% highlight bash %}
$ vagrant ssh core-03
Last login: Sat Dec  7 22:27:11 2014 from 10.0.2.2
CoreOS (alpha)
core@core-03 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
4de7ad5d...     172.17.8.101    -
b76a0027...     172.17.8.201    -
e956fcd6...     172.17.8.102    -
f9b8add6...     172.17.8.103    -
{% endhighlight %}

可以看到，包括 etcd 集群在内，一共有 4 个结点提供计算服务。

最后，我们前面实际上在 worker CoreOS 中，并没有让 etcd 启动，但实际上却是启动了的，
可以登录 worker CoreOS，使用 systemctl 命令查看。

{% highlight bash %}
$ vagrant ssh
CoreOS (alpha)
core@core-01 ~ $ systemctl status etcd.service
● etcd.service - etcd
   Loaded: loaded (/usr/lib64/systemd/system/etcd.service; static)
   Active: active (running) since Sat 2014-12-06 14:28:56 UTC; 11min ago
 Main PID: 995 (etcd)
   CGroup: /system.slice/etcd.service
           └─995 /usr/bin/etcd
{% endhighlight %}

这是因为：fleet.service 依赖于（Wants）etcd.service，由于只是 Wants 依赖，所以可以
强行禁止掉 etcd.service 即可，这里不再详细介绍方法。
