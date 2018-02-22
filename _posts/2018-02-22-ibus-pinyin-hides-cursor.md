---
layout: post
title: ibus-pinyin 输入法会隐藏 terminal 的光标
tags:
  - ibus
---

在 debian 9 中，我发现 ibus 1.5.x 在切换到 ibus-pinyin 输入法时，terminal
的光标会自动隐藏，对于使用 vim 做为日常编辑器的我来说，非常不方便。

因为，当移动光标的时候，我根本不知道现在移动到哪儿了。

同样，直接在命令行中切换到 ibus-pinyin 并且输入中文后，光标也会消失。

ibus 和 ibus-pinyin 的版本如下：

{% highlight console %}
root@chengwei-hp-debian:~# dpkg -l ibus ibus-pinyin
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                            Version              Architecture Description
+++-===============================-====================-====================-====================================================================
ii  ibus                            1.5.14-3             amd64 Intelligent Input Bus - core
ii  ibus-pinyin                     1.5.0-4              amd64 Pinyin engine for IBus
{% endhighlight %}

这个 bug 在 debian 8 中并不存在，所以我在 ibus-pinyin 的上反馈了这个
[bug](https://github.com/phuang/ibus-pinyin/issues/11)；不过，很遗憾，几个月来一直没有任何动静。

我目前还没有找到绕过这个 bug 的办法，如果大家有好办法，欢迎评论。

笨办法有两个：

1. 从 ibus-pinyin 切换回 english 输入法，注意：不是用 *shift* 键来让 ibus-pinyin
   输入英文

2. 切换一下窗口，再切回来

其它办法我尝试过：

1. 从 debian 8 的源中安装 ibus 和 ibus-pinyin，安装指定版本用 `apt-get install <pkg>=<version>`,
   不过，无效，所以看起来确实是 debian 9 才有的问题
2. 使用其它 terminal 而非 gnome-terminal，我试过 terminator 依然有这个问题
