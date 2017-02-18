---
layout: post
title: 理解 SSH 客户端连接超时配置
tags:
    - linux
    - ssh
---

Linux 中的 ssh 命令在连接到远程主机时，可以指定一个超时时间（ConnectTimeout），但是，细心的同学可能已经发现，即使指定了一个**非常大**的时间，
也不会生效。

这里就来分析下，为什么一个**非常大**的超时时间配置并不会生效。

**首先，尝试连接一个不存在的主机**

```bash
$ date;ssh -v 10.20.30.40; date
Mon Feb 13 16:37:11 CST 2017
OpenSSH_6.7p1 Debian-5+deb8u3, OpenSSL 1.0.1t  3 May 2016
debug1: Reading configuration data /home/chengwei/.ssh/config
debug1: /home/chengwei/.ssh/config line 195: Applying options for 10.*
debug1: /home/chengwei/.ssh/config line 199: Applying options for *
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: Applying options for *
debug1: Connecting to 10.20.30.40 [10.20.30.40] port 22.
debug1: connect to address 10.20.30.40 port 22: Connection timed out
ssh: connect to host 10.20.30.40 port 22: Connection timed out
Mon Feb 13 16:39:18 CST 2017
```

可以看到，大概 2 分 7 秒之后，连接失败，超时退出。

**现在，指定超时时间为 5 秒**

```bash
$ date; ssh -o ConnectTimeout=5 10.20.30.40; date
Sat Feb 18 11:56:28 CST 2017
ssh: connect to host 10.20.30.40 port 22: Connection timed out
Sat Feb 18 11:56:33 CST 2017
```

可以看到，5 秒钟后，ssh 即连接失败，超时退出。

**那么，指定超时时间为 180 秒呢？**

```bash
$ date; ssh -o ConnectTimeout=180 10.20.30.40; date
Sat Feb 18 11:57:47 CST 2017
ssh: connect to host 10.20.30.40 port 22: Connection timed out
Sat Feb 18 11:59:55 CST 2017
```

可以看到，超时时间并没有是 180 秒，而是 2 分 8 秒，只比 2 分 7 秒多 1 秒，甚至可以设置一个更大的超时，
也会发现，ssh 超时退出总是在 2 分 7 秒左右。

**ssh 源代码**

分析 ssh 连接到远程这部分相关代码，可以看到，ssh 在不配置 ConnectTimeout 选项时，默认使用 Linux 系统调用 connect(2)
来建立 TCP 连接，代码如下：

sshconnect.c:L340

```c
if (*timeoutp <= 0) {
    result = connect(sockfd, serv_addr, addrlen);
    goto done;
}
```

而如果指定了 ConnectTimeout 选项时，ssh 会使用一个多路复用系统调用 select(2) 来等待连接，并且设定 ConnectTimeout 为 select 的超时参数。

sshconnect.c:L577

```c
rc = select(connection_in + 1, fdset, NULL,
    fdset, &t_remaining);
```

而底层建立 TCP 连接，依然使用 connect(2) 系统调用，所以一旦 connect 超时（2 分 7 秒）时，select 的 stderr 这个 fdset 就变成 read ready 了，
所以 ssh 能够判断出失败了，从而退出或者开始新一轮尝试（ConnectionAttempts 选项）。

所以，ConnectTimeout 只能小于等于 Linux kernel 的 connect(2) 超时时间，大于并无意义；如果要获得更大的超时时间，
可以通过配置 ConnectionAttempts 来实现。

**关于 Linux kernel 的 connect(2) 超时分析，请参考 [Linux 建立 TCP 连接的超时时间分析]({% post_url 2017-02-18-linux-connect-timeout %})**。
