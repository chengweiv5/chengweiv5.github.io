---
layout: post
title: Shell 脚本处理配置文件
tags:
  - shell
---

通常，在写shell脚本的时候需要使用配置文件来和用户交互。配置文件的变量可以直接使用解析文件的方法来获得，还有一种简单常见的方法是使用source或(.)来读取配置文件，然后在当前进程中就可以直接使用了，省去了不少功夫，需要注意的是，由于使用source或(.)来读取配置文件，所以变量的赋值需要遵循shell变量赋值，即等号的两端不要有空格。

例如：

{% highlight bash %}
======conf.sh BEGIN======
#!/bin/bash
CONF_FILE=$(basename $0).conf
[ -f $CONF_FILE ] && . $CONF_FILE
echo "$var1"
======conf.sh END=======

======conf.sh.conf BEGIN===
# comments
var1=yes
======conf.sh.conf END====
{% endhighlight %}
