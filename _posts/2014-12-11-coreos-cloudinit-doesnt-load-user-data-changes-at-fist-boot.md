---
layout: post
title: Vagrant CoreOS 在 user-data 更新后的第一次重启不会生效
tags:
  - vagrant
  - coreos
---

最近学习 CoreOS 时发现一个非常奇怪的现象：coreos-cloudinit 的文档声称在每次系统
重启时都会加载 cloudinit 配置的文件；但是，我在 Vagrant 平台上修改 user-data
后，重启后却发现并没有生效，但是更新后的 user-data 文件已经出现在了 CoreOS 文件
系统中。

下面详细介绍一下这个问题。

首先，按照
[Running CoreOS on Vagrant](https://coreos.com/docs/running-coreos/platforms/vagrant)
文档，可以非常容易启动一个实例CoreOS，也可以参考我的另一篇博客
[CoreOS -- 第一次体验]({% post_url 2014-12-06-CoreOS-fisrt-step %})

这里，假设 user-data 文件内容如下：

{% highlight bash %}
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    #discovery: https://discovery.etcd.io/<token>
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001
  fleet:
    public-ip: $public_ipv4
  units:
    - name: etcd.service
      command: start
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
#write_files:
#  - path: /etc/test.txt
#    content: |
#      hello, write_files
{% endhighlight %}

当 CoreOS 启动后，我们可以取消上面最后几行的注释，所以，我们的本意是想让 CoreOS
在下次启动的时候能够新建一个 /etc/test.txt 文件，并且内容为*hello, write_files*。

修改 user-data 后，使用 vagrant 命令重启 CoreOS 并且重新加载 user-data。

{% highlight bash %}
$ vagrant reload --provision
{% endhighlight %}

启动完成后，登陆
core-01，查看文件*/var/lib/coreos-vagrant/vagrantfile-user-data*的内容。

{% highlight bash %}
core@core-01 ~ $ cat /var/lib/coreos-vagrant/vagrantfile-user-data 
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    #discovery: https://discovery.etcd.io/<token>
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001
  fleet:
    public-ip: $public_ipv4
  units:
    - name: etcd.service
      command: start
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
write_files:
  - path: /etc/test.txt
    content: |
      hello, write_files
{% endhighlight %}

可以看到，文件已经更新了，然后，看看是否生成了 /etc/test.txt 呢。

{% highlight bash %}
core@core-01 ~ $ ls /etc/test.txt
ls: cannot access /etc/test.txt: No such file or directory
{% endhighlight %}

这就非常奇怪了，因为根据[coreos-cloudinit](https://github.com/coreos/coreos-cloudinit)
的[文档](https://github.com/coreos/coreos-cloudinit/blob/master/Documentation/cloud-config.md)
所述：

>Your cloud-config is processed during each boot. Invalid cloud-config won't be
>processed but will be logged in the journal. 

每次重启 coreos-cloudinit 都会加载用户配置的 cloud-config，但是，从上面的实验看，
显然是没有生效的，从 coreos-cloudinit 的启动日志中，可以看到如下日志：

{% highlight bash %}
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: Checking availability of "local-file"
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: Fetching user-data from datasource of type "local-file"
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: Fetching meta-data from datasource of type "local-file"
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Parsing user-data as cloud-config
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: Processing cloud-config from user-data
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Updated /etc/environment
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Writing unit docker-tcp.socket to filesystem at path /etc/systemd/system/docker-tcp.socket
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Placed unit docker-tcp.socket at /etc/systemd/system/docker-tcp.socket
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Enabling unit file docker-tcp.socket
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Enabled unit docker-tcp.socket
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Writing unit etcd.service to filesystem at path /run/systemd/system/etcd.service.d/20-cloudinit.conf
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Placed unit etcd.service at /run/systemd/system/etcd.service.d/20-cloudinit.conf
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Ensuring runtime unit file etcd.service is unmasked
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 /run/systemd/system/etcd.service.d/20-cloudinit.conf is not null or empty, refusing to unmask
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Writing unit fleet.service to filesystem at path /run/systemd/system/fleet.service.d/20-cloudinit.conf
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Placed unit fleet.service at /run/systemd/system/fleet.service.d/20-cloudinit.conf
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 Ensuring runtime unit file fleet.service is unmasked
Dec 11 01:38:45 core-01 coreos-cloudinit[540]: 2014/12/11 01:38:45 /run/systemd/system/fleet.service.d/20-cloudinit.conf is not null or empty, refusing to unmask
{% endhighlight %}

显然没有任何错误，但是也没有任何信息关于创建 /etc/test.txt。所以，一开始，我向
coreos-cloudinit 提交了[错误报告](https://github.com/coreos/coreos-cloudinit/issues/278)，
然后继续 debug。

最后，终于发现原因在于：vagrant 将用户配置文件加载到 CoreOS 中的时间点**晚于**
coreos-cloudinit 读取文件的时间，所以导致修改 user-data 后的第一次重启后没有
读取到更新的 user-data，而只是读取到了上一次 vagrant 放进 CoreOS 中的旧的 user-data。

这不仅从 coreos-cloudinit 的日志中（参考
[github 讨论](https://github.com/coreos/coreos-cloudinit/issues/278)，我修改了 coreos-cloudinit，
将其读取的文件内容打印到了日志）可以发现，还可以从 coreos-cloudinit 运行是的时间
和 */var/lib/coreos-vagrant/vagrantfile-user-data* 的修改时间中发现。

所以，这不是一个 coreos-cloudinit 的 bug，但是一个平台集成的 bug，CoreOS 和 Vagrant
集成的 bug。
