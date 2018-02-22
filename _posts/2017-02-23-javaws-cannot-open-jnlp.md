---
layout: post
title: javaws 打开 jnlp 提示没有权限打开没有安全签名的 jars
tags:
    - java
---

Dell 服务器的 iDRAC 管理平台使用 javaws 来打开控制台，而在 Linux（Debian）环境下，
即使是 java，用起来也比较费劲；今天莫名其妙（很久没有用过 iDRAC 了）的 javaws 不能打开 iDRAC jnlp 文件了。

使用 `javaws viewer.jnlp` 命令打开时，一路确定后，最后报错，从命令行输出可以看到如下错误：

{% highlight console %}
netx: Initialization Error: Could not initialize application. (Fatal: Application Error: Cannot grant permissions to unsigned jars. Application requested security permissions, but jars are not signed.)
net.sourceforge.jnlp.LaunchException: Fatal: Initialization Error: Could not initialize application. The application has not been initialized, for more information execute javaws from the command line.
        at net.sourceforge.jnlp.Launcher.createApplication(Launcher.java:782)
        at net.sourceforge.jnlp.Launcher.launchApplication(Launcher.java:522)
        at net.sourceforge.jnlp.Launcher$TgThread.run(Launcher.java:904)
Caused by: net.sourceforge.jnlp.LaunchException: Fatal: Application Error: Cannot grant permissions to unsigned jars. Application requested security permissions, but jars are not signed.
        at net.sourceforge.jnlp.runtime.JNLPClassLoader$SecurityDelegateImpl.getClassLoaderSecurity(JNLPClassLoader.java:2358)
        at net.sourceforge.jnlp.runtime.JNLPClassLoader.setSecurity(JNLPClassLoader.java:315)
        at net.sourceforge.jnlp.runtime.JNLPClassLoader.<init>(JNLPClassLoader.java:285)
        at net.sourceforge.jnlp.runtime.JNLPClassLoader.createInstance(JNLPClassLoader.java:351)
        at net.sourceforge.jnlp.runtime.JNLPClassLoader.getInstance(JNLPClassLoader.java:418)
        at net.sourceforge.jnlp.runtime.JNLPClassLoader.getInstance(JNLPClassLoader.java:394)
        at net.sourceforge.jnlp.Launcher.createApplication(Launcher.java:774)
        ... 2 more
{% endhighlight %}

根据错误提示 Google 好一会儿，无非都是说 java 从哪个版本开始加强了安全限制，
也没说具体版本，总之就是没有找到直接能绕过的办法。

后来试了系统上安装的 openjdk 1.8，1.7，1.6，全都是一样的错，至少从 openjdk 来看，
并不是版本带来的问题。

然后下载了 oracle java，解压后，运行起来同样的错。最后，看了下 jre/bin 目录下的可执行文件，
看到一个 `ControlPanel` 的链接文件，指向 `jcontrol`。

打开看了下，通过在 Security 面板中，将安全等级修改为最低的 **Medium**，果然就能打开了，如下图所示：

![jcontrol security setting](/assets/images/misc/jcontrol-security-medium.png)
