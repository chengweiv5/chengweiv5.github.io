---
layout: post
title: 透明 Cache 服务导致 apt-get update Hash Sum mismatch
tags:
  - linux
---

上周在构建 Docker 镜像时，突然发现 `apt-get update`
总是失败，一开始以为是因为用的国内源，可能是同步没有完成；所以也没当回事儿，等等再试试呗；
但是后来还是不行，换成 Ubuntu 官方源还是不行，这就奇怪了。

Google 了一阵，发现 `Hash Sum mismatch` 错误最常见的解决办法就是

```
# apt-get clean

或者简单粗暴的删除 /var/lib/apt/lists/*，然后再执行

# apt-get update
```

上面的办法对于我的情况来说，没有丝毫好转，而且 `apt-get update`
每次报告的 Hash Sum mismatch 出现的 repo 并不完全一样，这就非常有趣了。

```
# apt-get update
...
Get:26 http://mirrors.163.com trusty/universe amd64 Packages [7589 kB]
W: Failed to fetch http://mirrors.163.com/ubuntu/dists/trusty-security/main/binary-amd64/Packages Hash Sum mismatch

W: Failed to fetch http://mirrors.163.com/ubuntu/dists/trusty-security/universe/binary-amd64/Packages Hash Sum mismatch

E: Some index files failed to download. They have been ignored, or old ones used instead.
```

然后在 launchpad 上找到一个 issue，上面介绍了怎样开启 apt-get debug，如下：

```
# apt-get update -o Debug::Acquire::http=true
```

从结果来看，惊呆了

```
GET /files/11160000000028CF/mirrors.aliyun.com/ubuntu/dists/trusty/universe/binary-amd64/Packages.gz HTTP/1.1
Host: 10.20.30.40
Cache-Control: max-age=0
User-Agent: Debian APT-HTTP/1.3 (1.0.1ubuntu2)


HTTP/1.1 200 OK
Server: nginx
Date: Thu, 03 Mar 2016 02:15:16 GMT
Content-Type: application/octet-stream
Content-Length: 7588885
Last-Modified: Tue, 16 Feb 2016 08:16:24 GMT
Connection: keep-alive
Accept-Ranges: bytes

Get:47 http://mirrors.163.com trusty/universe amd64 Packages [7589 kB]
```

请求的 163 服务器上的 metadata，`10.20.30.40` 这个 Cache 机返回的是 aliyun.com 上的 Cache。
后来经 IT 解释 `10.20.30.40` 是正在测试锐捷的 Cache 机，真是神机！
