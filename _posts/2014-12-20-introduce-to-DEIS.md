---
layout: post
title: DEIS 初探
tags:
  - coreos
  - deis
---

[DEIS](http://deis.io/) 是一个基于 [CoreOS](http://coreos.com/) 的 PaaS 平台，
最近刚好体验了一下，所以暂且记录一些东西在这里。

DEIS 官方有非常详细的上手[文档](http://docs.deis.io/en/latest/)，以及博客等。
按照官方文档，可以非常容易的搭建一个小规模的 DEIS PaaS 环境，所以我也基于
Vagrant 平台体验了一下，总的来说，过程非常简单，如果之前熟悉 CoreOS 的话。

但是，也有一些地方需要注意：

- 不要修改 CoreOS 的虚机，DEIS 默认为 CoreoS 配置的 Vagrantfile，使用的虚机
IP 从 172.17.8.100 开始，相应地，它会把这些 IP 添加到 DNS A record 中，
即文档中出现的例如：local3.deisapp.com 会解析到 172.17.8.100, 172.17.8.101,
172.17.8.102 三个 IP；修改后，你将发现使用 local3.deisapp.com
不能访问到你的 CoreOS，导致一些命令不能工作，例如：deis register

- 如果要体验创建 app，那么除了将自己的 ssh public key 添加到 deis 后，
还需要将相应的 ssh private key 添加到 ssh session 中，参考我的
[patch](https://github.com/deis/deis/pull/2771)

- 如果需要配置 CoreOS，修改 deis.git 中的文件 contrib/coreos/user-data

- 如果是在比较老的操作系统中使用 deis 客户端，参考其
[Github](https://github.com/deis/deis/tree/master/client)
主页文档直接使用 deis.py，而不要使用文档中的安装命令，否则会带来依赖噩梦

#### 认识 DEIS

[Understanding Deis](http://docs.deis.io/en/latest/understanding_deis/concepts/)
中描述了 DEIS 的关键概念，架构，组件，对认识 DEIS 很有帮助。

DEIS 首先是基于 CoreOS 的，整个 DEIS 平台的所有组件都运行在 CoreOS 之上，由于
CoreOS 运行程序的标准是使用 Docker 容器，所以 DEIS 的所有组件都是基于 Docker
容器的。

关于 Docker 容器，需要理解的一点是：它是一个应用(Application)容器，而非一个 OS；
理解了这点，也就很容易理解为什么虽然 Docker 镜像往往具有一个操作系统的文件系统，
但是却不会启动例如系统守护进程这类东西。Docker 专为运行应用(Application)而设计，
所以一个容器启动，就只会启动这个应用，非常快速。

[DEIS Architecture](http://docs.deis.io/en/latest/understanding_deis/architecture/)
中有几张架构图，比较详细的介绍了 DEIS 的架构以及各个组件的作用和它们之间的关系。

<img src=/assets/images/deis/deis-system-architecture.png width=730 />

从系统架构图中，可以看出：

系统中有 3 个角色：用户，开发者以及系统管理员；用户是使用设备访问 DEIS
中应用的发起者，开发者这里指应用的开发者，同时也是 DEIS 平台的用户，
系统管理员则负责运维 DEIS 平台。

系统从下往上由 CoreOS 的核心组件：etcd 和 fleet，Control Plan, Data Plan 以及
Router Mesh 组成。

<img src=/assets/images/deis/deis-control-plane-architecture.png width=730 />

在 Control Plane 中，系统管理员使用 deisctl 命令来管理 DEIS 平台，deisctl 命令和
fleet 以及 etcd 打交道，fleet 和 etcd 是 CoreOS 的核心组件；fleet
是一个分布式调度系统，基于 systemd，etcd 是一个分布式 key/value 存储系统，
基于 Raft 协议。

为了对系统管理员这个角色有进一步认识，这里可以看一下 deisctl 都有哪些功能。

{% highlight bash %}
Usage: deisctl [options] <command> [<args>...]

Commands, use "deisctl help <command>" to learn more:
  install           install components, or the entire platform
  uninstall         uninstall components
  list              list installed components
  start             start compnents
  stop              stop components
  restart           stop, then start components
  scale             grow or shrink the number of routers or registries
  journal           print the log output of a component
  config            set platform or component values
  refresh-units     refresh unit files from GitHub
  help              show the help screen for a command
...
{% endhighlight %}

这里并没有贴出整个命令的输出，感兴趣的同学可以参考 *deisctl help*。
这些命令操作的对象都是 DEIS 平台的组件，例如：install platform 会安装整个 DEIS
平台，包括所有组件。list 可以列出所有组件，例如：

{% highlight bash %}
UNIT                            MACHINE                         LOAD    ACTIVE  SUB
deis-builder.service            40ce962e.../172.17.8.200        loaded  active  running
deis-cache.service              eef33a07.../172.17.8.202        loaded  active  running
deis-controller.service         127aa857.../172.17.8.201        loaded  active  running
deis-database.service           eef33a07.../172.17.8.202        loaded  active  running
deis-logger.service             40ce962e.../172.17.8.200        loaded  active  running
deis-logspout.service           127aa857.../172.17.8.201        loaded  active  running
deis-logspout.service           40ce962e.../172.17.8.200        loaded  active  running
deis-logspout.service           eef33a07.../172.17.8.202        loaded  active  running
deis-publisher.service          127aa857.../172.17.8.201        loaded  active  running
deis-publisher.service          40ce962e.../172.17.8.200        loaded  active  running
deis-publisher.service          eef33a07.../172.17.8.202        loaded  active  running
deis-registry@1.service         127aa857.../172.17.8.201        loaded  active  running
deis-router@1.service           eef33a07.../172.17.8.202        loaded  active  running
deis-router@2.service           40ce962e.../172.17.8.200        loaded  active  running
deis-router@3.service           127aa857.../172.17.8.201        loaded  active  running
deis-store-daemon.service       127aa857.../172.17.8.201        loaded  active  running
deis-store-daemon.service       40ce962e.../172.17.8.200        loaded  active  running
deis-store-daemon.service       eef33a07.../172.17.8.202        loaded  active  running
deis-store-gateway.service      127aa857.../172.17.8.201        loaded  active  running
deis-store-metadata.service     127aa857.../172.17.8.201        loaded  active  running
deis-store-metadata.service     40ce962e.../172.17.8.200        loaded  active  running
deis-store-metadata.service     eef33a07.../172.17.8.202        loaded  active  running
deis-store-monitor.service      127aa857.../172.17.8.201        loaded  active  running
deis-store-monitor.service      40ce962e.../172.17.8.200        loaded  active  running
deis-store-monitor.service      eef33a07.../172.17.8.202        loaded  active  running
deis-store-volume.service       127aa857.../172.17.8.201        loaded  active  running
deis-store-volume.service       40ce962e.../172.17.8.200        loaded  active  running
deis-store-volume.service       eef33a07.../172.17.8.202        loaded  active  running
{% endhighlight %}

上面是一个正常安装后的结果，整个安装过程比较慢，视网速而定，因为所有的 DEIS
组件都是基于 Docker 的，所以需要下载整个 Docker 镜像。从上面的结果来看，我修改了 CoreOS
的 IP，以至于后面我需要添加 local3.deisapp.com 和相应的 IP 到 /etc/hosts 中，或者配置本地
DNS 缓存，例如 dnsmasq。

另一方面，另一个主要角色：应用开发者，DEIS 平台的用户则使用 deis 命令和 DEIS 交互，
从而在平台上发布，维护自己的应用。从图中可以看出，应用开发者这里主要和 DEIS 的核心组件交互，
而不是向系统管理员那样去操作组件。

图中并没有画出所有组件，这里只画出了可以和开发者交互的控制组件，例如：builder, controller
等等。同样，为了更好的理解应用开发者，这里来看一下 deis 命令都有些什么功能。

{% highlight bash %}
The Deis command-line client issues API calls to a Deis controller.

Usage: deis <command> [<args>...]

Auth commands::

  register      register a new user with a controller
  login         login to a controller
  logout        logout from the current controller

Subcommands, use ``deis help [subcommand]`` to learn more::

  apps          manage applications used to provide services
  ps            manage processes inside an app container
  config        manage environment variables that define app config
  domains       manage and assign domain names to your applications
  builds        manage builds created using `git push`
  limits        manage resource limits for your application
  tags          manage tags for application containers
  releases      manage releases of an application

  keys          manage ssh keys used for `git push` deployments
  perms         manage permissions for applications
...
{% endhighlight %}

这里并没有贴出整个命令的 help 输出，感兴趣的同学可以自行查看 *deis help*。

首先，应用开发者需要注册（register）一个账号，然后可以登入/登出平台。

然后，可以使用其它命令管理整个应用的生命周期，例如：上传应用代码，构建应用，发布应用，
横向扩展应用（Scale）等等。

这里简单介绍一下 Control Plane 中的几个组件：

- builder, DEIS 支持 3 中方式构建应用：Buildpacks, Dockerfile, Docker 镜像。所以，builder
的任务就是通过这 3 种形式的应用输入来构建一个应用，由于没有使用过 Buildpacks，这里不做介绍。
builder 首先内建了 Git 服务的支持，应用通过 git push 的方式上传代码。
使用 Dockerfile 方式非常简单，builder 首先会使用 'docker build' 通过应用上传的 Dockerfile
构建出 Docker 镜像，然后上传到 registry 组件中；另外，builder 还支持 Docker 镜像，
这对使用私有 docker-registry 服务非常友好。每次构建完成后，builder 会发布（release）应用，
同时，controller 会自动为应用启动容器

- controller, 控制器是 deis API server，负责接收和分发用户请求

- registry, 正如 builder 中介绍的那样，registry 实际上为 DEIS 平台提供了 docker-registry
服务，而底层的存储则由 ceph 提供

- database, PostgreSQL 服务，负责存储平台需要持久化的内容，例如应用的配置等，
以便平台故障恢复时能够恢复到之前的状态

- logger, 中心日志服务，DEIS 平台中，每个应用都将日志输出到标准输出，标准错误，由 logspout
采集，并且投递到 logger 服务，实际上基于 syslog

- store, 由 ceph 组成，它提供了整个平台的基础存储，registry, database, logger
都将数据存储于 store 中

<img src=/assets/images/deis/deis-data-plane-architecture.png width=730 />

Data Plane 中主要是应用部分，为用户提供服务，应用在 DEIS 平台上都以 Docker
容器的方式运行，每个应用的可访问地址都经过 Docker 映射，然后通过 router, proxy
来访问真正的应用，正如 Control Plane 中所说的那样，每个应用的日志通过 logspout 发送到
logger 组件中，便于集中管理和排障。

从我看来，DEIS 提供了一个非常标准的 PaaS 平台，包括但不限于：

- 用户认证
- Git 方式管理应用代码，自动化构建，版本发布，上线，回滚等
- 支持应用的横向扩展，手动方式，自动配置服务的反向代理
- 支持容器的自动失败重启，并且重新配置上线
- 支持对应用容器的资源限制，CPU 和内存

但是也有一些局限性：

- 支持 Dockerfile, Docker 镜像，但是只能导出一个端口（[issue 1156](https://github.com/deis/deis/issues/1156)），
也就是如果一个应用监听在两个端口的话，这是不行的，当然，以后可能支持
- 没有集成很好的应用监控方案，需要二次开发，例如：集成 [cadvisor](https://github.com/google/cadvisor)
- 没有集成很好的应用访问量监控，需要二次开发
- 没有集成很好的负载均衡器，例如：HAProxy, Nginx 等
- 不能自动扩展（Auto Scaling），例如设置一些自动扩展规则，根据 QPS 或者根据容器 CPU, 内存消耗量等
- 没有提供完善的开发者界面，例如可以方便的查看自己应用的历史数据，当前状态，请求报表，资源消耗
情况，计费等等

目前，我只是初步使用了一下 DEIS，对其宏观架构，使用方式上有肤浅的理解，
对于各个组件深入的理解还需要进一步的学习和研究。
