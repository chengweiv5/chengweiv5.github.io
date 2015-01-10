---
layout: post
title: Ubuntu 下设置 Java 环境变量和自举类路径
tags:
  - java 笔记
---

在Ubuntu8.04下设置Java环境变量如下：

{% highlight bash %}
$ echo $PATH回车 查看当前的环境变量
$ gedit ./.bashrc回车（使用gedit打开.bashrc文件）
{% endhighlight %}

然后在这个文件的最后添加上
{% highlight java %}
export JAVA_HOME="/home/dengni/SDK/jdk"
export CLASSPATH=".:/home/dengni/SDK/jdk/lib/dt.jar:/home/dengni/SDK/jdk/lib/tools.jar"
export PATH="$PATH:/home/dengni/SDK/jdk/bin"
{% endhighlight %}

以上设置完成Java开发的环境变量和ClASSPATH.
保存文件后，重新打开一个终端，使用echo $PATH查看环境变量，可见环境变量中已经添加了Java的路径。
