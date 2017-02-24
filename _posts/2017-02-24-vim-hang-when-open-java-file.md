---
layout: post
title: Vim 打开 Java 源码文件的时候卡住
tags:
    - vim
---

之前也注意到 Vim 打开某些类型源码文件的时候会比较慢，有时候可能需要几秒钟，
原因就是安装了很多插件，所以也没多想；但是，今天 vim 居然在打开 storm 源码中的
java 文件时直接卡住了，这就不能忍了。

现象是这样的，下载了 storm 0.9.6 的源码包，解压后，cd 进去，然后用

```bash
$ vim storm-core/src/jvm/backtype/storm/scheduler/Cluster.java
```

打开这个文件，然后就发现 vim 一直不显示文件内容，一直卡住，开始以为是在加载插件，
等几秒也就好了，结果等了几十秒还不好，这就奇怪了。

而且 Ctrl-C 还不管用，更奇怪了。

用 strace 打开看看

```bash
$ strace vim storm-core/src/jvm/backtype/storm/scheduler/Cluster.java &> strace-vim.log
```

发现到了最后，一直是重复如下几行日志

```
recvmsg(7, 0x7ffcd07ed720, 0)           = -1 EAGAIN (Resource temporarily unavailable)
recvmsg(7, 0x7ffcd07ed720, 0)           = -1 EAGAIN (Resource temporarily unavailable)
poll([{fd=7, events=POLLIN}], 1, 0)     = 0 (Timeout)
select(8, [0 7], NULL, [0], {0, 0})     = 0 (Timeout)
```

搜索 `= 7`，发现是个 socket

```
socket(PF_LOCAL, SOCK_STREAM:call <SNR>114_align()
aSOCK_CLOEXEC, 0) = 7
connect(7, {sa_family=AF_LOCAL, sun_path=@"/tmp/.X11-unix/X0"}, 20) = 0
getpeername(7, {sa_family=AF_LOCAL, sun_path=@"/tmp/.X11-unix/X0"}, [20]) = 0|)
```

但是是 unix socket，而且还是和 x11 通信的，没看出来什么毛病。

且罢！

然后用 pstree 看了下 vim 的进程树，想看下 vim 是不是启动了其它什么玩意儿，导致卡住，
不看不知道，一看吓一跳，赫然看到如下东西。

```
├─vim storm-core/src/jvm/backtype/storm/scheduler/Cluster.java
  ├─java -classpath /usr/local/maven/boot/plexus-classworlds-2.5.2.jar -Dclassworlds.conf=/usr/local/maven/bin/m2.conf -Dmaven.home=/usr/local/maven-Dm
  │   └─14*[{java}]
  └─{vim}
```

vim 居然启动了一个 java 进程在搞什么鬼，进入 storm 源码目录，执行下这行命令，看看效果：

```
$ /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -classpath /usr/local/maven/boot/plexus-classworlds-2.5.2.jar -Dclassworlds.conf=/usr/local/maven/bin/m2.conf -Dmaven.home=/usr/local/maven -Dmaven.multiModuleProjectDirectory=/home/chengwei/Downloads/apache-storm-0.9.6 org.codehaus.plexus.classworlds.launcher.Launcher -f storm-core/pom.xml help:effective-pom
[INFO] Scanning for projects...
Downloading: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-javadoc-plugin/2.9/maven-javadoc-plugin-2.9.jar
```

我去，居然是在下载依赖的 jar 包，这不卡住才怪。

看到这里，已经知道确实是有某个插件在搞鬼了，外国人写代码的时候总是不考虑中国基本国情。

回头看下 strace-vim.log，找下 maven, pom.xml 相关的，果然发现

```
stat("/home/chengwei/Downloads/apache-storm-0.9.6/storm-core/pom.xml", {st_mode=S_IFREG:call <SNR>114_align()
a0644, st_size=14871, ...}) = 0
stat("/home/chengwei/Downloads/apache-storm-0.9.6/storm-core/pom.xml", {st_mode=S_IFREG|0644, st_size=14871, ...}) = 0
stat("/home/chengwei/Downloads/apache-storm-0.9.6/storm-core/pom.xml", {st_mode=S_IFREG|0644, st_size=14871, ...}) = 0
getcwd("/home/chengwei/Downloads/apache-storm-0.9.6", 4096) = 44
stat("storm-core/pom.xml", {st_mode=S_IFREG|0644, st_size=14871, ...}) = 0
open("storm-core/pom.xml", O_RDONLY|O_NONBLOCK) = 8
close(8)                                = 0
stat("mvn", 0x7ffcd07e47c0)             = -1 ENOENT (No such file or directory)|})
```

然后，去 ~/.vim 目录中搜一下：

```
bundle/syntastic/syntax_checkers/java/javac.vim:    let pom = findfile('pom.xml', '.;')
```

发现是 syntax_checkers，然后进去看下 javac.vim 就真相大白了。
