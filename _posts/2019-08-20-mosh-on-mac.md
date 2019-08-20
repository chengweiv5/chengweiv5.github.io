---
layout: post
title: 解决 MacBook 上 mosh 连接失败的问题
tags:
  - mosh
  - mac
---

[mosh](https://mosh.org/) 是一个非常不错的 ssh 软件，通过 ssh over udp
的方式，能够解决网络切换导致 ssh session 断开的问题，搭配 tmux 使用非常好用。

tmux 解决了远程工作 session 的持续性，重新 ssh 连接后，直接 attach 即可；而 mosh
则解决了重新连接的问题，mosh 能够做到在网络切换（例如：在工位上是有线网络，抱着笔记本去开会的时候，会连接到无线网络）
导致 ssh 断开连接的问题。

mosh 的使用这里不再介绍，非常简单，远程和本地都安装 mosh 这个包即可。

这里讲一下在 mac 上实际遇到一个问题，mac 上连接远程的时候，mosh
报错，连不上。打印的输出如下：

```
The locale requested by LC_CTYPE=UTF-8 isn't available here.
Running `locale-gen UTF-8' may be necessary.

The locale requested by LC_CTYPE=UTF-8 isn't available here.
Running `locale-gen UTF-8' may be necessary.

mosh-server needs a UTF-8 native locale to run.

Unfortunately, the local environment (LC_CTYPE=UTF-8) specifies
the character set "US-ASCII",

The client-supplied environment (LC_CTYPE=UTF-8) specifies
the character set "US-ASCII".

locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
...
省略
...
```

上面这个问题咋一看会忽略，因为通常 locale
问题都不是问题，但是这里确实关键；上面的错误提示当前 locale 的 `LC_CTYPE`
不满足需求，所以用 locale 命令查看一下：

```
LANG="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_CTYPE="UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_ALL=
```
可以看到，`LC_CTYPE` 的值是 **UTF-8**，也就是只有字符集，没有设置语言，也就是常见的 `zh_CN`, `en_US` 这种。

这里一个是在终端直接导出：

```
export LC_CTYPE="en_US.UTF-8"
```

或者可以配置 iTerm2 的属性，在 Preferences -> Advanced
里搜索 `LC_CTYPE`，然后将默认的 value 修改为 `en_US.UTF-8` 即可。

然后，新开启 iTerm2 console，mosh 就能正常连接了。
