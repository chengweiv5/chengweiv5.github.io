---
layout: post
title: 注册 RedHat 开发者也不能访问 subscription only contents
tags:
    - redhat
---

一直发现有很多关于 CentOS 或者 Linux kernel or system 的许多问题，在 RedHat 上都有解答，
但是大多数都是 subscription only 的内容，所以没法一睹芳容。

而尝试自己在 RedHat 官方上去订阅（基础版 $349 一年每台服务器）的时候发现，网上自助订阅不支持中国区，
中国区是比较特殊，而是必须联系中国区的代理才行。

所以，暂且放下，看看有没有其它法子，然后搜索到 RedHat 开发者计划向开发者免费了 RedHat 订阅以及开发软件。
然后就注册了开发者，然后下载了最新的 RedHat 7.3，以虚机的形式安装上。

**这里需要注意的是：安装过程中选择安装图形界面，这样更方面注册**

注册完之后，attach 到开发者账号中，然后发现确实可以使用 yum 来升级系统，也就表示至少软件源是可以使用了。

现在来看看 subscription only 的文档内容是否可以查看，从 system tools 中打开 `Red Hat Access`，搜索
"NMI received for unknown reason on CPU"，打开搜索出来的一条结果，如下图所示：

![redhat access](/assets/images/redhat/redhat-access.png)

依然不能看到对问题的解决方法，只能看到问题描述，所以很遗憾，RedHat 开发者计划并没有被赋予
knowledge base 的访问权限。
