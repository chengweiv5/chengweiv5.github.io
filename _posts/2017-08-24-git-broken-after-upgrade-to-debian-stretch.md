---
layout: post
title: Debian 升级到 stretch 版本后 git 命令出错
tags:
  - debian
---

Debian 9 “stretch” 发布已经有 2 个来月了，作为 Debian 粉，所以也把办公电脑从 jessie
升级到了 stretch，debian 升级很简单，而且也没有什么风险。

升级之后，确实还遇到几个小问题，比如：

某些服务起不来了，导致系统进不了 X，例如：mysql，mariadb 服务，然后就直接禁止掉了。

[Awesome WM](https://awesomewm.org/) 从 3.x 升级到了 4.x，然后发现配置文件不兼容了，
非常奇怪的是 awesome 命令检查配置文件却说 OK，启动的时候却报错。。。

后来基于新的配置文件模板重新 port 了一遍，才算完事。

还有个问题就是 git clone/fetch/pull 等远程命令不工作了，通通都会报类似下面的错误：

{% highlight console %}
fatal: unable to access 'https://chengwei.unfuddle.com/git/chengwei_awesome/': gnutls_handshake() failed: Public key signature verification has failed.
{% endhighlight %}

Google 发现，这是一个从 stretch 还是 testing 的时候就存在的问题，
真是悲伤，stretch 都已经转正了，这个问题还存在，令人失望。

解决办法就是 downgrade libcurl3-gnutls 到 jessie 中的版本。

方法如下：

1. 添加 jessie 的源到 /etc/apt/sources.list.d/jessie.list，内容如下：

    ```console
    # cat jessie.list
    deb http://mirrors.163.com/debian/ jessie main contrib non-free
    deb-src http://mirrors.163.com/debian/ jessie main contrib non-free
    ```
2. 查看 jessie 中 libcurl3-gnutls 的版本

    ```console
    # apt-get update
    # apt-cache show apt-get -t=jessie install libcurl3-gnutls
    ```
3. 然后安装 libcurl3-gnutls 指定版本，当前 jessie 中的版本是 7.38.0-4+deb8u5

    ```console
    # apt-get install libcurl3-gnutls=7.38.0-4+deb8u5
    ```

安装完成后，git clone/fetch/pull 等命令就正常了。

最后，为了避免下次升级（`apt-get upgrade`）的时候，自动把 libcurl3-gnutls 给升级，可以用 `apt-mark`
锁住这个包，命令如下：

```
# apt-mark hold libcurl3-gnutls
```
