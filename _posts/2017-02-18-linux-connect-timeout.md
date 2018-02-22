---
layout: post
title: Linux 建立 TCP 连接的超时时间分析
tags:
    - linux
    - network
---

Linux 系统默认的建立 TCP 连接的超时时间为 127 秒，对于许多客户端来说，这个时间都太长了，
特别是当这个客户端实际上是一个服务的时候，更希望能够尽早失败，以便能够选择其它的可用服务重新尝试。

socket 是 Linux 下实现的传输控制层协议，包括 TCP 和 UDP，一个 socket 端点由 IP 和端口对来唯一标识；
如果开启了地址复用，那么可以进一步由协议，IP 和端口来唯一标识。

系统调用 connect(2) 则是用来尝试建立 socket 连接（TCP）或者和远程协商一致（UDP）的函数。
connect 对于 UDP 来说并不是必须的，而对于 TCP 来说则是一个必须过程，注明的 TCP 3 次握手实际上也由 connect 来完成。

**注：这里只分析 TCP 连接超时**

网络中的连接超时非常常见，不管是广域网还是局域网，为了一定程度上容忍失败，所以连接加入了重试机制，
而另一方面，为了不给服务端带来过大的压力，重试也是有限制的。

在 Linux 中，连接超时典型为 2 分 7 秒，而对于一些 client 来说，这是一个非常长的时间；
所以在编程中，可以使用非阻塞的方式来实现，例如：使用 poll(2), epoll(2), select(2) 等系统调用来实现多路复用等待。

下面来看看 2 分 7 秒是怎样来的，以及怎样配置 Linux kernel 来缩短这个超时。

## 2 分 7 秒
---

2 分 7 秒即 127 秒，刚好是 2 的 7 次方减一，聪明的读者可能已经看出来了，如果 TCP 握手的 SYN 包超时重试按照 2 的幂来 backoff，
那么：

  0. 第 1 次发送 SYN 报文后等待 1s（2 的 0 次幂），如果超时，则重试
  1. 第 2 次发送后等待 2s（2 的 1 次幂），如果超时，则重试
  2. 第 3 次发送后等待 4s（2 的 2 次幂），如果超时，则重试
  6. 第 4 次发送后等待 8s（2 的 3 次幂），如果超时，则重试
  6. 第 5 次发送后等待 16s（2 的 4 次幂），如果超时，则重试
  6. 第 6 次发送后等待 32s（2 的 5 次幂），如果超时，则重试
  7. 第 7 次发送后等待 64s（2 的 6 次幂），如果超时，则超时失败

上面的结果刚好是 127 秒。也就是说 Linux 内核在尝试建立 TCP 连接时，最多会尝试 7 次。

那么下面通过具体方法来验证。

**首先，配置 iptables 来丢弃指定端口的 SYN 报文**

{% highlight console %}
# iptables -A INPUT --protocol tcp --dport 5000 --syn -j DROP
{% endhighlight %}

**然后，打开 tcpdump 观察到达指定端口的报文**

{% highlight console %}
# tcpdump -i lo -Ss0 -n src 127.0.0.1 and dst 127.0.0.1 and port 5000
{% endhighlight %}

**最后，使用 telnet 连接指定端口**

{% highlight console %}
$ date; telnet 127.0.0.1 5000; date
{% endhighlight %}

上面命令的输出如下：

{% highlight console %}
Tue Jan  3 16:39:05 CST 2017
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection timed out
Tue Jan  3 16:41:12 CST 2017
{% endhighlight %}

而从 `tcpdump` 命令的输出可以看到：

{% highlight console %}
16:39:05.690238 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179222486 ecr 0,nop,wscale 7], length 0
16:39:06.686988 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179222736 ecr 0,nop,wscale 7], length 0
16:39:08.690980 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179223237 ecr 0,nop,wscale 7], length 0
16:39:12.702973 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179224240 ecr 0,nop,wscale 7], length 0
16:39:20.718991 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179226244 ecr 0,nop,wscale 7], length 0
16:39:36.766986 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179230256 ecr 0,nop,wscale 7], length 0
16:40:08.830996 IP 127.0.0.1.58933 > 127.0.0.1.5000: Flags [S], seq 2286786481, win 43690, options [mss 65495,sackOK,TS val 179238272 ecr 0,nop,wscale 7], length 0
{% endhighlight %}

其中，`Flags [S]` 表示为 SYN 报文，可以看到总共发送了 7 次 SYN 报文，最后一次的时间为 **16:40:08**，而 telnet 超时退出的时间为 **16:41:12**，相差 64 秒。

## 怎样修改 connect timeout
---

对于很多客户端程序来说，127 秒都是一个很长的时间，特别是对于局域网来说，公司内部往往都具有网络质量较好的局域网，
访问内部的服务并不需要等待这么长的超时，而可以 fail earlier。

Linux 内核中，`net.ipv4.tcp_syn_retries` 表示建立 TCP 连接时 SYN 报文重试的次数，默认为 6，可以通过 sysctl 命令查看。

{% highlight console %}
# sysctl -a | grep tcp_syn_retries
net.ipv4.tcp_syn_retries = 6
{% endhighlight %}

将其修改为 1，则可以将 connect 超时时间改为 3 秒，例如：

{% highlight console %}
# sysctl net.ipv4.tcp_syn_retries=1
{% endhighlight %}

再次使用 telnet 验证超时时间，如下：

{% highlight console %}
$ date; telnet 127.0.0.1 5000; date
Fri Feb 17 09:50:12 CST 2017
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection timed out
Fri Feb 17 09:50:15 CST 2017
{% endhighlight %}

**注意：sysctl 修改的内核参数在系统重启后失效，如果需要持久化，可以修改系统配置文件**

例如：，对于 CentOS 7 来说，添加 `net.ipv4.tcp_syn_retries = 1` 到 /etc/sysctl.conf 中即可。
