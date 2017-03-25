---
layout: post
title: 机器能 ping 通但是不能 ssh
tags:
    - linux
    - network
---

最近发现一种情况，从办公室网络能 ping 通一台数据中心的虚机，但是 ssh 却连不上，
后来发现这台虚机加上一条内部网络路由之后就能 ssh 了。那么，这是不是有点奇怪，
因为不可能是因为路由的问题，因为显然 ping 能通那么就表示路由一定是可达的。

现在就来看看到底是什么原因导致的。

假设办公室机器为 A，IP 为 10.1.24.155；目标虚机为 B，IP 为 10.110.19.89。

从 A ping B，并且在 A 上抓包。

ping 命令的输出如下：

```
$ ping -c1 10.110.19.89
PING 10.110.19.89 (10.110.19.89) 56(84) bytes of data.
64 bytes from 36.110.208.44: icmp_seq=1 ttl=57 time=2.42 ms

--- 10.110.19.89 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.429/2.429/2.429/0.000 ms
```

抓包结果如下：

```
# tcpdump -ni eth0 host 10.110.19.89
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
13:00:21.424199 IP 10.1.241.55 > 10.110.19.89: ICMP echo request, id 28135, seq 1, length 64
```

发现了吗？ping 是通了，但是响应的包的 src ip 是 36.110.208.44，并不是 B 的 IP，
而且从 tcpdump 抓包结果看，确实没有收到 src ip 为 B 的 ICMP reply 包。

现在来看下 ssh B 以及抓包结果。

ssh 命令及输出如下：

```
$ ssh -v 10.110.19.89
OpenSSH_6.7p1 Debian-5+deb8u3, OpenSSL 1.0.1t  3 May 2016
debug1: Reading configuration data /home/chengwei/.ssh/config
debug1: /home/chengwei/.ssh/config line 210: Applying options for 10.*
debug1: /home/chengwei/.ssh/config line 214: Applying options for *
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: Applying options for *
debug1: Connecting to 10.110.19.89 [10.110.19.89] port 220.
```

ssh 一直连接不上，而从 tcpdump 中可以看到一直在重试 SYN 包，直到发送 7 次后超时，
关于更多 ssh 超时方面的分析，可以参考[理解 SSH 客户端连接超时配置]{% post_url 2017-02-18-ssh-connect-timeout %}。

```
13:04:31.150610 IP 10.1.241.55.32955 > 10.110.19.89.22: Flags [S], seq 565869186, win 29200, options [mss 1460,sackOK,TS val 645699889 ecr 0,nop,wscale 7], length 0
13:04:32.147611 IP 10.1.241.55.32955 > 10.110.19.89.22: Flags [S], seq 565869186, win 29200, options [mss 1460,sackOK,TS val 645700139 ecr 0,nop,wscale 7], length 0
13:04:34.151595 IP 10.1.241.55.32955 > 10.110.19.89.22: Flags [S], seq 565869186, win 29200, options [mss 1460,sackOK,TS val 645700640 ecr 0,nop,wscale 7], length 0
... 省略 4 行输出 ...
```

那么，ssh 的同时，在 B 上抓包看看

```
# tcpdump -ni eth0 host 10.1.241.55                                                                                                                                          
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
16:12:46.099785 IP 10.1.241.55.43766 > 10.110.19.89.ssh: Flags [S], seq 1174678115, win 29200, options [mss 1460,sackOK,TS val 626923624 ecr 0,nop,wscale 7], length 0
16:12:46.099832 IP 10.110.19.89.ssh > 10.1.241.55.43766: Flags [S.], seq 1226453374, ack 1174678116, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
16:12:47.096727 IP 10.1.241.55.43766 > 10.110.19.89.ssh: Flags [S], seq 1174678115, win 29200, options [mss 1460,sackOK,TS val 626923874 ecr 0,nop,wscale 7], length 0
16:12:47.096765 IP 10.110.19.89.ssh > 10.1.241.55.43766: Flags [S.], seq 1226453374, ack 1174678116, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
16:12:48.500075 IP 10.110.19.89.ssh > 10.1.241.55.43766: Flags [S.], seq 1226453374, ack 1174678116, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
16:12:49.100714 IP 10.1.241.55.43766 > 10.110.19.89.ssh: Flags [S], seq 1174678115, win 29200, options [mss 1460,sackOK,TS val 626924375 ecr 0,nop,wscale 7], length 0
16:12:49.100745 IP 10.110.19.89.ssh > 10.1.241.55.43766: Flags [S.], seq 1226453374, ack 1174678116, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
... 省略 ...
```

从 B 上抓包，可以看到 B 确实有回复 ACK 报文，但是为什么 A 没有收到呢？

来看看 B 的路由表

```
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.110.31.1     0.0.0.0         UG    0      0        0 eth0
10.110.16.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
```

看出来了，唯一匹配 10.1.241.55 的就是默认路由，而默认路由的网关是 10.110.31.1，
这个 IP 是一个 LVS SNAT 服务器，上去看看。

在 ssh B 的同时，在 LVS SNAT 服务器上抓包：

```
# tcpdump -n host 10.1.241.55
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
13:13:30.376596 IP 10.110.19.89.ssh > 10.1.241.55.34097: Flags [S.], seq 2899453339, ack 1292079022, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
13:13:30.376610 IP 36.110.208.44.ssh > 10.1.241.55.34097: Flags [S.], seq 2899453339, ack 1292079022, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
13:13:31.372555 IP 10.110.19.89.ssh > 10.1.241.55.34097: Flags [S.], seq 2899453339, ack 1292079022, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
13:13:31.372564 IP 36.110.208.44.ssh > 10.1.241.55.34097: Flags [S.], seq 2899453339, ack 1292079022, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0
... 省略 ...
```

可以看到，LVS 收到了来自 B 的 ACK 报文，并且做了 SNAT 转换（替换 src IP）之后发往机器 A，
但是，可以看到 src IP 是 36.110.208.44，这不正是从 A 上 ping B 的时候收到的 ICMP echo reply 包的源 IP 吗！

所以，进一步明朗了，虽然 ICMP echo reply 包中的 dst IP 和 echo request 的 src IP 不同，
但是却可以通过，而 TCP ACK 包中的 src IP 和 TCP SYN 包中的 dst IP 不同，却不能通过，
很显然是交换机或防火墙的限制。

那么到底是外网防火墙还是内网呢？

这需要弄明白 LVS 的包是从哪里出来的，查看 LVS SNAT 服务器的路由表：

```
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
36.110.208.0    0.0.0.0         255.255.255.0   U     0      0        0 eth1
10.110.16.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
10.0.0.0        10.110.31.254   255.0.0.0       UG    0      0        0 eth0
0.0.0.0         36.110.208.254  0.0.0.0         UG    0      0        0 eth1
```

可以看到，目标网段 10.0.0.0 会从 eth0 发出，并且网关是 10.110.31.254。

这里一句话解释一下 LVS SNAT 的工作原理：LVS 收到包之后，替换包的 src IP，然后转发该包，
和普通的发包类似，转发的过程会查找 Linux 内核路由表，所以，虽然这个包具有 eth1 的 IP，
其实却是从 eth0 发出。
