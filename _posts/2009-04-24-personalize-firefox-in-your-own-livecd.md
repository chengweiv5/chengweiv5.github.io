---
layout: post
title: Ubuntu 8.04 下配置 Live CD 中的 Firefox
tags:
  - ubuntu
---

这篇文章的目的是在[打造自己的 Ubuntu Live CD]({% post_url 2009-04-20-create-your-own-ubuntu-livecd %})
时，定制Firefox。

首先找到Firefox的配置文件

{% highlight bash %}
$ cd ~/.mozilla/firefox/*.default
{% endhighlight %}

打开prefs.js，请注意，这个配置文件是动态生成的，而不是动态修改的，也就是当你编辑完配置文件以后，然后使用了Firefox，在Firefox中编辑了“首选项”，那么Firefox会重新生成一个prefs.js文件覆盖掉原来的文件，所以不要在里面添加你自己的东西。

修改主页，如下：

{% highlight bash %}
user_pref("browser.startup.homepage","http://www.chengweiyang.cn/");
user_pref("browser.startup.page",1);
{% endhighlight %}

这里最后的值：1表示打开时显示主页，2表示显示一个空白页，3表示显示上次关闭时的页面。没有其它值。

修改下载文件的存放位置，并且设置下载完成后关闭下载窗口：

{% highlight bash %}
user_pref("browser.download.dir","/home/yourname/FirefoxDownload");
user_pref("browser.download.manager.closeWhenDone",true);
{% endhighlight %}
