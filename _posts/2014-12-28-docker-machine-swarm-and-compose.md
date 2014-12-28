---
layout: post
title: Docker 新特性 -- Machine, Swarm, Compose
excerpt: Docker 公司在 12 月初的 DockerCon14, EU 上发布了即将到来的 3 个 Docker 新特性：Docker Macine, Docker Swarm 以及 Docker Compose。在我看来，这不仅是 Docker 更加平台化的特性，而且还将和现有的生态环境中的某些创新公司产生竞争，特别是专注将 Docker 引入数据中心的 CoreOS。
tags:
  - docker
---

刚在 [Docker 首页](https://www.docker.com/)上，
发现在其显著位置介绍了即将到来的新特性：

> New: Docker Orchestration Services

> Assemble multi-container apps, run on any infrastructure.

从这个新特性来看，Docker 已经不再满足于做容器以及容器仓库了，
而是将要向上和向下延伸：向下到达计算节点层，以及集群管理调度层；
向上，进一步提高 Docker 容器间协作能力。

这篇[官方博客](https://blog.docker.com/2014/12/docker-announces-orchestration-for-multi-container-distributed-apps/)中介绍了
Docker 的三个新特性:

- [Docker Machine](https://github.com/docker/machine)
- [Docker Swarm](https://github.com/docker/swarm)
- Docker Compose

### Docker Machine

Docker Machine 将改变传统看待 Docker 的角度：

- 最传统的方式为一个 Linux Distribution 加上 Docker，这里的 Docker
在使用者眼中看来，其实和其它 Linux 上的软件概念上没有什么不同，
只是一个应用程序而已
- 而 CoreOS 则将 Docker 的地位进一步提升，在 CoreOS 中，Docker
是用户要运行其它软件的核心平台，除此之外，别无它法，所以在用户眼中，Docker
就显得异常圣神
- Docker Machine 的出现将再次提升 Docker 的地位，用户在创建 Docker Machine
时就已经只关心 Docker 了，全然不再关心 Machine 里的其它组件

Docker Machine 目前处于早期开发阶段，运行平台支持 VirtualBox, AWS,
Digital Ocean，后续应该会支持更多的平台，例如：OpenStack? 我想不会缺席。

从 Docker Machine 来看，目标是否和 CoreOS 类似呢？二者都是专为 Docker 打造（最近 CoreOS 发布了自己的容器平台 Rocket），
只关心运行 Docker 容器；当然，CoreOS 的目标是更适合运行在集群环境中，
包括了重要组件 etcd, fleet 等。我这里还未使用过 Docker Machine，
所以不清楚其基础镜像是什么，包含什么组件，具有什么特性，是否支持传统软件运行等等。

### Docker Swarm

Docker Swarm: a Docker-native clustering system，刚说到 CoreOS
的主战场是集群环境，Docker Swarm 就来报道了，这里可以将 Docker Swarm 和 CoreoS
fleet 对比，二者都是用于构建集群的工具，fleet 基于 etcd 来存储数据以及进行
leader (fleet engine) 选举，fleet agent 则运行在每个 CoreOS 上，负责执行 fleet engine
分发的任务，目前 fleet 的调度算法非常简单：总是调度到具有最小任务数的机器上，
很显然，非常简单，而且不够用。Docker Swarm 还未深入学习过，有待调研，
这里不做评论。但是，很显然 CoreOS 在发射 [Rocket](https://github.com/coreos/rocket)
后，Docker 做出了回应。

Docker Swarm 自身集成了调度器，但是宣称依然遵守

> "batteries included but removable"

甚至特别提到了 [Apache Mesos](http://mesos.apache.org/)，
目前基于 Mesos 的创业公司 Mesosphere 也在最新一轮融资中获得了 4000 万美元注资，
并宣布明年 Q1 将推出 [DCOS(Data Center Operating System)](http://mesosphere.com/)，
相信数据中心之战将在明年更加激烈！当然，容器方面的竞争也期待 CoreOS Rocket 的表现。

### Docker Compose

最后一个新特性是 Docker Compose，这个特性看起来中规中矩，是 Docker
应用场景的自然延生，因为就 Docker 的设计准则来说，它的目标是足够轻量，
只运行一个应用，安静的做一个应用容器，而不是慢慢的变成一个 OS 容器，
变得笨重不堪。但是，单个应用容器使用的场景毕竟有限，
现在很难想象一个软件将所有事情做完，而不需要和其它软件协作；虽然有 Docker
已经有 link, volume 这类容器见协作的功能，但是对于用户来说，使用起来并不是很容易，
特别是要架构多个容器，以及考虑后期的扩展性等，所以在容器协作方面往往是由 PaaS
平台来简化，现在基于 Docker 的 PaaS 平台不在少数，比较热门的有：Google Kubernetes,
DEIS, Flynn 等。

Docker Compose 引入了 group 概念，将协作容器组成一个 group，然后使用 yaml 文件
来定义 group，同时对 Docker 各个标准 API 都进行了扩展以支持 group, build, pull,
run, start, stop 等等，还引入了新的命令 docker up。

最后，所有 3 个新特性预计在明年 Q2 ready。
