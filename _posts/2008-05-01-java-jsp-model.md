---
layout: post
title: Java JSP 模型
tags:
  - java 笔记
---

#### JSP 模型

{% highlight java %}
<%!定义成员变量%>
<%@ 页面指令%>
<%  java语句%>
<%--注释--%>
{% endhighlight %}

#### JSP的生命周期
- 网页解释
- JSP编译
- 载入类
- 建立Servlet实体
- 调用jspInit()方法（JspPage接口中定义）
- 调用_jspService()方法（HttpJspPage接口中定义）
- 调用jspDestroy()方法（JspPage接口中定义）

#### JSP内置对象

{% highlight java %}
对象名                 作用域
request                request
response               page
out                    page
config                 page
pageContext            page
page                   page
exception              page
session                session
application            application
{% endhighlight %}

#### JSP的异常处理机制

{% highlight java %}
<%@ page isErrorPage="true"or"false" %> 指明当前页面是否是错误处理页面
<%@ page errorPage="errors.jsp"%> 指明当前页面出错时处理错误的页面
{% endhighlight %}
