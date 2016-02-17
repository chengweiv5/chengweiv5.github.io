---
layout: post
title: Hibernate inverse
tags:
  - java
---

最近在学习Hibernate,对于inverse这个属性有点了解了，inverse主要是为了提高hibernate运行效率的，如果不是用inverse的话，hibernate就会从关系（还有set的类型）的两端来维护数据的统一，所以更新一项数据时会使用3条sql语句，即两条insert into加上一条update而如果使用了inverse就只有一条insert into和一条update语句。通常在一对多关系中在“多”端来维护数据，即在“一”端使用inverse="true"。而在多对多中由于效率相当所以选择合适的一端就可以了。举例来说：

一对多关系：如一个User可以发表多篇Blog 在User.hbm.xml中关于blogs属性的配置如下

{% highlight java %}
<set name="blogs" inverse="true" cascade="all">
  <key column="user_id" not-null="true" unique="true"/>
  <one-to-many class="Blog"/>
</set>
{% endhighlight %}

而在Blog.hbm.xml文件中user属性配置如下

{% highlight java %}
<many-to-one name="user" class="User">
  <column name="user_id" not-null="true" unique="false"/>
</many-to-one>
{% endhighlight %}

问题：为什么两个配置中对于user_id的unique属性设置的值不一样呢？

多对多关系中的inverse
这个根据自己的需要设置就行了，主要是看你在程序中操作谁。比如User订阅某些News,而News可以被很多人订阅。

这是我们编写程序时，通常是针对某个User来操作，这是就需要把inverse设置在News.hbm.xml文件中，表示由User来维护。

千万不要在两端都使用inverse="true"，这样的后果是你不能建立关系，数据库中的数据不会改变。
