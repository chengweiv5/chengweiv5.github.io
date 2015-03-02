---
layout: post
title: CoreOS 与 Docker 度过蜜月期
tags:
  - misc
---

Docker 可谓是 2014 年最火的云计算开源项目，获得了一大批科技界巨头的鼎力支持；
另外，围绕 Docker 的创业公司也如雨后春笋般破土而出，国内许多人都感叹 Docker
发展太快，生态圈太大，简直目不暇接！

CoreOS 作为 Docker 生态圈的佼佼者，专注于成为最适合 Docker 运行的生产云环境，
CoreOS 作为一个最小化 Linux 发行版，引入了许多创新，包括但不限于：

- 可运行于多种平台（bare metal, 各种云环境）
- 最小化系统
- 整体系统升级/回滚方案
- 容器化所有非系统应用，无包管理器
- 集群化调度器 fleet
- 分布式高可靠 key/value 存储系统 etcd
- ...

CoreOS 在创业之初，也积极参与到 Docker 的开发中，为 Docker 贡献了许多代码，
二者在社区合作上表现非常亲密，但是好景不长。

CoreOS 于 2014 年 12 月 1 号[宣布](https://coreos.com/blog/rocket/)了 Rocket
项目，定位于标准化应用容器，标准称之为 Application Container Spec，通常简称为
appc spec。

Rocket 以及 appc spec 最近刚[发布](https://coreos.com/blog/app-container-and-docker/)了 0.3.2 版本，
并且表示已经支持 Docker 镜像，而且在博客中还发现很有趣的现象：CoreOS
表示已经将支持 appc spec 的 patch 提交给 Docker，但是并未被采纳。

另一方面，在 2014 年 12 月 4,5 号举行的 DockerCon 2014 EU 会议上，Docker 推出了三板斧：

- Docker Machine
- Docker Swarm
- Docker Compose

而 Docker, Inc 的速度相当快，Docker Machine 已经发布 [Beta](https://blog.docker.com/2015/02/announcing-docker-machine-beta/)
版本，而 Docker Swarm 和 Compose 也宣布可供[下载测试](https://blog.docker.com/2015/02/orchestrating-docker-with-machine-swarm-and-compose/)。

而 Docker 发布的这三件套对 CoreOS 会带来什么影响呢？简单的说：Docker
三件套也是为了规范 Docker 编排调度的，形成容器云的标准，这正是 CoreOS
的领域。

为什么会出现 CoreOS 和 Docker 直接叫阵的情况，背后情况也不得而知，
在表面上，CoreOS 标榜树立中立的 appc spec，而 Docker 并不买账，这里可以看看 Docker
CTO Solomon 对于 CoreOS 提交的 [PR](https://github.com/docker/docker/pull/10776) 中的 comment。

>Can someone explain to me how the user benefits from this? Are we talking about improving the image distribution system with cryptographic signature and better mapping to the DNS namespace? Then send a patch to improve the existing image system, or even better, join the ongoing effort to do exactly that. If you don't think it's worth your effort to improve Docker's system, that's fine, just do it in your own project and let the best project win. But guys, you have to choose one or the other. This whole masquerade about "protecting the community" with made-up standards sounds great in the press, good for you. But here, we care about the users. And it makes no sense for the user to have to deal with 2 incompatible universes of images with the same tool. It creates artificial constraints and solves zero problems. These 2 formats are almost completely identical, and the differences will only diminish as we progress on our respective roadmaps.

>Please, if you want to change my mind, walk me through a concrete user scenario where the user is better off dealing with 2 "standards" instead of simply continuing to improve the one we have.

>Otherwise, I beg you, stop the charade, and let's go back to competing normally like the rest of the open-source world does.

Solomon 已经不能压抑自己心中对 CoreOS 发起 appc spec 项目的怒火，直接叫板 CoreOS 正面竞争，
显然对于一个做开源软件的技术人员来说，说出上面的话是不容易的。

当然，CoreOS 并不甘心，其 CTO Brandon Philips 又另外提交了一个 [proposal](https://github.com/docker/docker/issues/10777)，
但是，显然，已经没有任何 Docker 核心开发者 feedback 了，当然，Brandon 自己例外。

所以，可以看出，Docker 与 CoreOS 的蜜月期已经过了，你以为这就完了？

另外，还有一个小插曲：mesosphere 很高兴的对世人[宣布](https://mesosphere.com/2015/02/26/deploying-with-docker-swarm/)
Docker CTO Solomon Hykes 表示 mesos 是大规模运行生产容器环境的“黄金标准”。

>For everyone involved with Apache Mesos and the Mesosphere Datacenter Operating System (DCOS), it was a real honor when Solomon Hykes (CTO of Docker) said at DockerCon EU in December that Mesos is the "gold standard" for large-scale production clusters running containers.
