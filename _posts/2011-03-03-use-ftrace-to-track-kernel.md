---
layout: post
title: 利用 Ftrace 进行内核跟踪调试
tags:
  - linux
---

Ftrace (function trace)是Linux内核开发中很常用的一个执行路径跟踪程序，不同于strace，它可以跟踪所有内核函数执行路径而不是只有系统调用，所以它显得非常强大。

#### 挂载debugfs文件系统
通常，debugfs文件系统应该挂载在了/sys/kernel/debug/安装点上，如果没有挂载，那么可以使用下面的命令挂载

{% highlight bash %}
#mount -t debugfs nodev /sys/kernel/debug
{% endhighlight %}

Ftrace，它的名称只代表了它的最初设计目的，后来有许多tracer加入其中，这里只介绍最常用的function tracer和function graph tracer。

#### 相关的控制文件

在/sys/kernel/debug/tracing/目录下导出了一系列控制接口文件，最常用的有
README 从这儿开始，更详细的ftrace介绍可以在Linux源码包/Documentation/trace/ftrace.txt中找到

- available_tracers 当前内核支持的tracer，编译时可定制
- current_tracer 当前使用的tracer
- available_filter_functions 当前内核导出的可以跟踪的函数
- set_ftrace_filter 设置当前跟踪的函数集
- set_ftrace_pid 设置要跟踪的进程
- trace_options 一些有用的选项
- traceing_enabled 开关文件
- trace 输出文件

#### 最简单的使用方法

{% highlight bash %}
#cd /sys/kernel/debug/tracing
#echo 'function_graph' > current_tracer
#echo 1 > tracing_enabled
#do something
#echo 0 > tracing_enabled
#cat trace 查看结果
{% endhighlight %}

以上是最简单的使用方法，但比较不实用，因为电脑的速度总是远远敲键盘的速度，所以即使你只想跟踪一条命令，例如touch file，当你关闭ftrace，查看trace文件时，也可以吓死你，几万行的输出。

#### 有用的tips

1). 只收集感兴趣的函数调用

{% highlight bash %}
#echo '*:mod:btrfs' > set_ftrace_filter
{% endhighlight %}

这里我只对btrfs文件系统中的执行路径感兴趣。

2). 让function_graph显示TASK-PID

function tracer默认会显示进程名和进程号，这非常有用，因为往往我们只关心自己执行的命令，但是function graph tracer默认不会显示，执行下面命令可以设置

{% highlight bash %}
#echo 'funcgraph-proc' > tracer_options
{% endhighlight %}

3). 只收集指定进程

例如，我关心建立一个文件，文件系统需要执行的操作，但是touch file这条命令转瞬即逝，怎样才能将它的PID写入set_ftracer_pid文件呢？在Documentation/trace/ftrace.txt中提供了一种以C语言实现的解决方法，这里我用shell脚本实现了一个，因为shell是在一个进程中执行所有命令的。

{% highlight bash %}
============trace_me.sh============

#!/bin/bash

COMMAND="$@"
PID=$$
DEBUG_FS=/sys/kernel/debug
CURRENT_TRACER="function_graph"
SET_FUNCTION_GRAPH="*:mod:btrfs"

echo "$CURRENT_TRACER" > $DEBUG_FS/tracing/current_tracer
echo "$SET_FUNCTION_GRAPH" > $DEBUG_FS/tracing/set_ftrace_filter
echo "$PID" > $DEBUG_FS/tracing/set_ftrace_pid
echo 'funcgraph-proc' > $DEBUG_FS/tracing/trace_options
echo 1 > $DEBUG_FS/tracing/tracing_enabled
eval "$COMMAND"

cp $DEBUG_FS/tracing/trace /root/ftrace/$1-$PID.ftrace
echo 0 > $DEBUG_FS/tracing/tracing_enabled
echo -1 > $DEBUG_FS/tracing/set_ftrace_pid
echo '' > $DEBUG_FS/tracing/set_ftrace_filter
echo 'nop' > $DEBUG_FS/tracing/current_tracer
=============end trace_me.sh==========
{% endhighlight %}

总结，通过以上设置后，ftrace就使用起来就很实用了。要跟踪指定操作只需要执行

{% highlight bash %}
#./trace_me.sh command
{% endhighlight %}

例如

{% highlight bash %}
#./trace_me.sh touch file
{% endhighlight %}
