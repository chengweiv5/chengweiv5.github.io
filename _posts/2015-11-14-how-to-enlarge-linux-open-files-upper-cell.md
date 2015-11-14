---
layout: post
title: 怎样增大 Linux 系统的 open file(s) 上限
tags:
  - linux
  - ulimit
---

最近在工作中遇到一个问题，尝试直接将服务运行在高配（40core, 192GB；相比虚拟机来说）
的物理机上，但是发现服务打开的文件句柄达到 80 万左右就不能再开更多了。

80 万已经是一个不小的值了，通常情况下，Linux 默认的值都很小，例如：Debian
8(jessie) 给普通用户设置的 open file(s) 限制为 65536，
可以通过下面的命令查看当前限制。

```bash
$ ulimit -n
$ ulimit -Sn
$ ulimit -Hn
```

`ulimit` 是一个 shell（这里使用的是 bash） 内置命令，可以通过 `type ulimit`
验证。

`-n` 即表示查看或者设置 open file(s) 的限制，在 ulimit
中，每个限制都有两种类型：

  - `-S`, soft limit, 软限制，用户可以上调软限制到硬限制
  - `-H`, hard limit, 硬限制，非 root 用户不能修改

如果没有指明，则同时修改软限制和硬限制。

## 修改 ulimit

修改分为临时修改和永久修改，临时修改只对当前 session
有效，登出和重启后都恢复系统设置。

临时修改使用 `ulimit` 命令，以修改 open file(s) 为例。

```bash
# ulimit -n 1024000
# ulimit -n
1024000
```

永久修改需要修改 `/etc/security/limits.conf` 或者在 `/etc/security/limits.d/`
目录下添加一个文件。具体格式参考 `/etc/security/limits.conf`，里面有详细说明。

## open file(s) 上限

回到遇到的问题中来：服务打开 80 万个左右的文件句柄就不能再打开了。所以，
尝试将 ulimit 设置为 1000 万，结果提示出错：

```bash
# ulimit -n 10000000
-bash: ulimit: open files: cannot modify limit: Operation not permitted
```

注意，使用的可以 root 用户，居然没有权限，然后尝试降低到：

  - 500 万，依然错误
  - 300 万，依然错误
  - 200 万，依然错误
  - 100 万，成功了

显然，这里有一个上限，大概在 100-200 万之间。

所以，解决问题的办法，在于怎样提高这个上限！

通过一番搜索，发现 open file(s) kernel 级别有 2 个配置，分别是：

```
fs.nr_open，进程级别
fs.file-max，系统级别
```

`fs.nr_open` 默认设置的上限是 1048576，所以用户的 open file(s)
不可能超过这个上限。

```bash
# sysctl -w fs.nr_open=10000000
# ulimit -n 10000000
# ulimit -n
10000000
```

修改后即可设置更大的 open file(s) 了。

同样，对于 kernel 参数的修改，`sysctl` 命令修改的是当前运行时，如果需要永久修改，
则将配置添加到 `/etc/sysctl.conf` 中，例如：

```bash
# echo "fs.nr_open = 10000000" >> /etc/sysctl.conf
# echo "fs.file-max = 11000000" >> /etc/sysctl.conf
```

注意：`fs.nr_open` 总是应该小于等于 `fs.file-max`。

如果要查看当前打开的文件数，使用下面的命令：

```bash
# sysctl fs.file-nr
fs.file-nr = 1760       0       11000000
```

不过，增大这些值意味着能够打开更多的文件（在 Linux 中，everything is file，包括
socket），但是同时也意味着消耗更多的资源，所以基本上在物理机上才会遇到这种问题。
