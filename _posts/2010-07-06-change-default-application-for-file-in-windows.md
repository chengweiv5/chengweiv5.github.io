---
layout: post
title: 修改 Windows 中文件的默认打开程序
tags:
  - windows
---

打开一个命令提示符窗口

{% highlight bash %}
assoc 命令可以查看系统中所有文件后缀名和文件类型的关联
ftype 命令可以修改文件类型打开软件的关联
{% endhighlight %}

以修改txt文件的默认关联为例。

打开一个命令提示符窗口，然后输入

{% highlight bash %}
assoc .txt
{% endhighlight %}

可以看到.txt后缀名表示文件类型是txtfile

使用ftype修改默认软件关联

{% highlight bash %}
ftype txtfile
{% endhighlight %}

可以看到默认使用C:/WINDOWS/notepad.exe打开txtfile类型文件，现在修改为使用vim打开

{% highlight bash %}
ftype txtfile=C:/path/to/gvim.exe %1
{% endhighlight %}

修改后，双击.txt后缀名文件即使用vim打开。
