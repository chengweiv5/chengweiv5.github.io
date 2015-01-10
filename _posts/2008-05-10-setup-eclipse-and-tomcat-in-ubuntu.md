---
layout: post
title: linux 下配置 eclipse3.3 + tomcat6.0
tags:
  - java 笔记
---

今天终于配置成功了Java Web开发环境，环境是Ubuntu8.04＋eclipse3.3+tomcat6.0。
上网google了很多次，都没有成功，今天终于成功了，所以希望对正在google的同学有点帮助。

#### 首先是要安装jdk6.0
安装很容易，然后设置环境变量，$sudo gedit /etc/.bashrc。设置环境变量。在文件的末尾加上下面三行

{% highlight java %}
export JAVA_HOME=/home/leisure/jdk1.6
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/rt.jar:$JAVA_HOME/lib/tools.jar
{% endhighlight %}

#### 安装tomcat6.0
其实只用解压tomcat6.0就行了，然后什么都不用做，记住不要去修改什么startup.sh文件。只要你的环境变量设置正确了，tomcat就会在环境变量中找到jvm。现在可以进入/home/.../tomcat6.0/bin目录，然后可能需要修改目录的权限才能运行。

{% highlight java %}
$ chmod -R 770 ........../bin(注意前面表示省略目录，根据你的安装目录。
{% endhighlight %}

然后运行startup.sh就能成功启动tomcat.

#### 安装eclipse3.3
下载eclipse3.3 for linux,解压后，需要把jdk1.6目录下的jre目录复制到eclipse主目录下也就是和plugins一个目录，然后把下载的tomcatplugin插件解压后复制到eclipse/plugins目录下。
打开eclipse然后进入菜单windows-->preferences-->tomcat在右边配置tomcat,选择tomcat的版本为6.x。
输入tomcat的主目录也就是你把tomcat6.0解压到的目录，千万不要填入插件的目录。在下面的tomcat配置中选择使用server.xml.它自动会填入文件。一切完工。点击eclipse上的tomcat图标就可以启动tomcat了。
