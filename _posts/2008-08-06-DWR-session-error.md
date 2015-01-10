---
layout: post
title: 解决 DWR session error
tags:
  - java 笔记
---

使用DWR出现Session error,控制台出现

{% highlight java %}
A request has been denied as a potential CSRF attack.
{% endhighlight %}

这个错误是存在csrf(cross-site request forgeries,跨站请求伪造)攻击.

这必须将web.xml中配置dwr Servlet初始参数时必须设置这个参数

{% highlight java %}
<init-param>
  <param-name>crossDomainSessionSecurity</param-name>
  <param-value>false</param-value>
</init-param>
{% endhighlight %}
