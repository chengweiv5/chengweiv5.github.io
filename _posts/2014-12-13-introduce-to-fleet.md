---
layout: post
title: CoreOS -- fleet 初探
tags:
  - coreos
  - fleet
---

fleet 作为 CoreOS 的核心组件之一，基于 systemd 和 etcd 在集群抽象层面实现了一个
进程调度和管理软件。在集群层面，fleet 基于 etcd 存储各个计算节点的元数据，从而
实现将进程从集群层面调度到具体的各个节点上；而在各个节点上，fleet 基于 systemd
来控制和管理各个进程。

正如 fleet [文档主页](https://github.com/coreos/fleet)中介绍的那样：

>fleet ties together systemd and etcd into a distributed init system. Think of
>it as an extension of systemd that operates at the cluster level instead of the
>machine level. This project is very low level and is designed as a foundation
>for higher order orchestration.

fleet 基于 systemd 和 etcd 实现了一个集群初始化系统，但是实现的非常有限的基础的功能，
例如：简单的 fairness 调度算法，总是将新的服务（systemd service）调度到具有最少服务的结点上。

既然是初探，这里从 [CoreOS Quick Start](https://coreos.com/docs/quickstart/) 开始，体验一下 fleet。

使用的 hello.service 直接来自于 CoreOS Quick Start，内容如下：

{% highlight bash %}
[Unit]
Description=My Service
After=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill hello
ExecStartPre=-/usr/bin/docker rm hello
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name hello busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
ExecStop=/usr/bin/docker stop hello
{% endhighlight %}

这个 hello.service 文件完全是一个常规的 systemd service 文件，没有任何特别之处，
如果非要说有什么特别的话，就是它和通常在 Fedora, OpenSuse 这些系统中见到的 systemd service 文件不太一样，
它启动的是一个 [Docker](http://docker.com/) 容器。

简单解释一下 hello.service 各行内容的意义：

- TimeoutStartSec=0，服务启动超时的时间，0为永不超时，因为可能要下载 Docker
	镜像，时间不好估计，所以设置为永不超时，这也是 systemd 的默认值，可以省略
- ExecStartPre=-/usr/bin/docker kill hello，在启动服务主进程之前要执行的命令，
  即杀死叫做 *hello* 的 Docker 容器，*=* 后的 *-* 表示忽略错误，后面两个
  ExecStartPre 也就容易理解了，第一个删除容器，第二个下载容器，但是，
  下载容器的错误没有被忽略，所以一旦失败，整个服务将失败
- ExecStart，从镜像 *busybox* 启动一个容器，命名为 *hello*，这个容器执行一个循环，
  每隔 1 秒钟打印 *Hello World*
- ExecStop，停止容器

### 配置本地集群

首先，启动一个具有 3 个结点的 CoreOS 集群，这里的集群架构是：

- 一个运行在主机上的 etcd 服务为集群提供发现服务
- 使用 Vagrant 启动 3 个 CoreOS 虚机，使用主机上的 etcd
	提供的发现服务组成一个集群

更详细的解析可以参考
[CoreOS -- etcd 初探]({% post_url 2014-12-07-CoreOS-etcd-first-view %})。

这里的 3 个 CoreOS 都使用一个 Vagrant 配置启动，修改 config.rb 如下：

{% highlight bash %}
...
$num_instances=3
...
{% endhighlight %}

修改 user-data，为 etcd 配置发现服务为主机上的 etcd 服务，如下：

{% highlight bash %}
...
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    # WARNING: replace each time you 'vagrant destroy'
    discovery: http://172.17.8.1:4001/v2/keys/ac39ba13-9c5a-43e8-b1bd-cc754a498173
    addr: $public_ipv4:4001
    peer-addr: $public_ipv4:7001
...
{% endhighlight %}

discovery url 中的最后一部分为一个合法的 etcd key 即可，注意不要和已有的 key
冲突，所以这里使用 **uuidgen** 生成了一个 key。

然后，使用 **vagrant up** 启动集群即可，启动完成后，登陆其中一台 CoreOS，查看集群中现有的结点。

{% highlight bash %}
$ vagrant ssh core-01
CoreOS (alpha)
core@core-01 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
77cc95ec...     172.17.8.103    -
91a0bbf1...     172.17.8.101    -
dc394f72...     172.17.8.102    -
{% endhighlight %}

可以看到，目前集群中有 3 个结点，说明 3 个结点已经加入到同一个集群当中。fleetctl
为 fleet 提供的命令行工具，不带任何子命令或者参数可以查看 fleetctl 的用法。
基本的用法包括：查看集群中的结点，集群中的服务，提交服务，停止服务等。由于 fleet
需要将服务调度到集群中的各个结点，而 fleet 通过 ssh 来登陆各个结点，所以在需要一些配置。

例如：接着上面的命令，我们在 core-01 中继续执行下面的命令：

{% highlight bash %}
core@core-01 ~ $ fleetctl ssh 77cc95ec
Failed building SSH client: SSH_AUTH_SOCK environment variable is not set. Verify ssh-agent is running. See https://github.com/coreos/fleet/blob/master/Documentation/using-the-client.md for help.
{% endhighlight %}

可见，core-01 中的 fleetctl ssh 并不能登陆 77cc95ec(172.17.8.103) 这个 CoreOS。
在 Vagrant 平台中，Vagrant 的 ssh public key 默认已经加入了 CoreOS 镜像中的 core
用户认证的 ssh public key 中，所以我们可以通过 **vagrant ssh** 登陆各个 CoreOS 系统。
而这个过程，vagrant 使用的 private key 为用户 HOME 目录下的
*.vagrant.d/insecure_private_key* 文件。所以，我们可以通过认证转发的功能，让 core
用户可以从一台 CoreOS 系统登陆另一台，而非只有从主机使用 **vagrant ssh** 来登陆。
方法如下：

{% highlight bash %}
[chengwei@controller fleet]$ eval $(ssh-agent)
Agent pid 45260
[chengwei@controller fleet]$ ssh-add ~/.vagrant.d/insecure_private_key 
Identity added: /home/chengwei/.vagrant.d/insecure_private_key (/home/chengwei/.vagrant.d/insecure_private_key)
[chengwei@controller fleet]$ vagrant ssh core-01 -- -A
Last login: Sat Dec 13 09:00:53 2014 from 10.0.2.2
CoreOS (alpha)
core@core-01 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
77cc95ec...     172.17.8.103    -
91a0bbf1...     172.17.8.101    -
dc394f72...     172.17.8.102    -
core@core-01 ~ $ fleetctl ssh 77  
The authenticity of host '172.17.8.103' can't be established.
RSA key fingerprint is a1:bc:37:c8:e2:5c:d0:cd:a7:ed:60:83:99:ca:45:fc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.17.8.103' (RSA) to the list of known hosts.
CoreOS (alpha)
core@core-03 ~ $ 
{% endhighlight %}

这里简单解释一下上面命令的含义：

- eval $(ssh-agent)，启动 ssh-agent 并且将环境变量导入到当前 shell 中，
  以便后续命令能够使用，注意：不要更换 console 窗口
- ssh-add ~/.vagrant.d/insecure_private_key，正如前面介绍的那样，将 vagrant
  的 private key 加入到 ssh agent session 中，这样，在每次执行 ssh 需要认证的时候，
  不用指定 private key
- vagrant ssh core-01 -- A，注意最后两个参数 *-- A*，**--** 表示后面的参数不是
  vagrant 的，而是 ssh 的，而 **-A** 这个 ssh 参数表示转发本地的ssh-agent
  session 到远程登陆 shell

了解了以上知识，就可以理解为什么后面从 core-01 中的 fleetctl ssh 77 能够登陆
core-03 机器了，注意：77 是 77cc95ec 的缩写，这是 fleetctl 的一个特性，
只要是能够获得唯一的机器，ID 就可以缩写，所以，77 甚至可以缩写为 **7**。

### fleetctl 基本用法简介

在配置好了使用环境后，可以使用 core-01 作为控制机，在当前目录（/home/core）
下创建前面介绍的 hello.service 文件，然后就可以使用 fleetctl 来控制它了。

提交 hello.service 到 fleet registry 中

{% highlight bash %}
core@core-01 ~ $ fleetctl submit hello.service
core@core-01 ~ $ fleetctl list-unit-files
UNIT            HASH    DSTATE          STATE           TARGET
hello.service   0d1c468 inactive        inactive        -
{% endhighlight %}

提交后，只是表明 fleet 知道了这个服务的存在，并且任何之后对文件的修改，
对于 fleet 来说都是不可见的，要让修改生效，必须删除之前的服务，重新提交。

现在来调度这个已经提交的服务

{% highlight bash %}
core@core-01 ~ $ fleetctl load hello.service
Unit hello.service loaded on 77cc95ec.../172.17.8.103
core@core-01 ~ $ fleetctl list-units
UNIT            MACHINE                         ACTIVE          SUB
hello.service   77cc95ec.../172.17.8.103        inactive        dead
{% endhighlight %}

现在，hello.service 已经被调度到了 core-03 这个结点上，但是目前并没有启动。

接下来启动这个服务

{% highlight bash %}
core@core-01 ~ $ fleetctl start hello.service
Unit hello.service launched on 77cc95ec.../172.17.8.103
{% endhighlight %}

稍等一段时间后，可以查看这个服务的状态

{% highlight bash %}
core@core-01 ~ $ fleetctl status hello.service
● hello.service - My Service
   Loaded: loaded (/run/fleet/units/hello.service; linked-runtime)
   Active: active (running) since Sat 2014-12-13 09:25:01 UTC; 6s ago
  Process: 1394 ExecStartPre=/usr/bin/docker pull busybox (code=exited, status=0/SUCCESS)
  Process: 1383 ExecStartPre=/usr/bin/docker rm hello (code=exited, status=1/FAILURE)
  Process: 1320 ExecStartPre=/usr/bin/docker kill hello (code=exited, status=1/FAILURE)
 Main PID: 1453 (docker)
   CGroup: /system.slice/hello.service
           └─1453 /usr/bin/docker run --name hello busybox /bin/sh -c while true; do echo Hello World; sleep 1; done

Dec 13 09:24:23 core-03 systemd[1]: Starting My Service...
Dec 13 09:24:23 core-03 docker[1320]: Error response from daemon: No such container: hello
Dec 13 09:24:23 core-03 docker[1320]: 2014/12/13 09:24:23 Error: failed to kill one or more containers
Dec 13 09:24:23 core-03 docker[1383]: Error response from daemon: No such container: hello
Dec 13 09:24:23 core-03 docker[1383]: 2014/12/13 09:24:23 Error: failed to remove one or more containers
Dec 13 09:24:32 core-03 docker[1394]: busybox:latest: The image you are pulling has been verified
Dec 13 09:25:01 core-03 docker[1394]: Status: Downloaded newer image for busybox:latest
Dec 13 09:25:01 core-03 systemd[1]: Started My Service.
Dec 13 09:25:01 core-03 docker[1453]: Hello World
Dec 13 09:25:02 core-03 docker[1453]: Hello World
{% endhighlight %}

可见服务已经启动了，现在可以停止服务

{% highlight bash %}
core@core-01 ~ $ fleetctl stop hello.service
Unit hello.service loaded on 77cc95ec.../172.17.8.103
core@core-01 ~ $ fleetctl list-units
UNIT            MACHINE                         ACTIVE          SUB
hello.service   77cc95ec.../172.17.8.103        deactivating    stop
{% endhighlight %}

然后，可以删除之前的调度，也就是让服务处于未调度的状态，可以重新进行调度，
重新调度可能会将其调度到其它结点中。

{% highlight bash %}
core@core-01 ~ $ fleetctl unload hello.service
Unit hello.service inactive
{% endhighlight %}

最后，还可以将服务从 fleet registry 中删除

{% highlight bash %}
core@core-01 ~ $ fleetctl list-unit-files
UNIT            HASH    DSTATE          STATE           TARGET
hello.service   0d1c468 inactive        inactive        -
core@core-01 ~ $ fleetctl destroy hello.service
Destroyed hello.service
core@core-01 ~ $ fleetctl list-unit-files
UNIT    HASH    DSTATE  STATE   TARGET
{% endhighlight %}

从上面的实践可以进一步了解到：服务在 fleet 中有几个状态，并且由命令控制状态之间的相互转换，
另外：start 和 destroy 是两个多合一的命令，start 可以完成 submit, load, start
操作，而相反：destroy 则可以完成 stop, unload, destroy 操作。

这里简单的介绍了 fleet 的用法，使用结点中的一个 CoreOS 作为控制结点，
完全使用 systemd unit 属性，没有使用 fleet 扩展属性，没有介绍 fleet engine，
调度算法等；这些将在后续的 fleet 文章中进一步介绍。
