---
layout: post
title: Java Servlet 基础（三）：容器模型
tags:
  - java
---

#### ServletContext
   每个web应用程序都有一个ServletContext对象，它可以被所有servlet所共享。
#### ServletContext的起始参数
在web.xml文件中可以配置ServletContext的起始参数

{% highlight java %}
<web-app> 
  <context-param>
    <param-name>
    <param-value>
  </context-param>
....
{% endhighlight %}

#### 存取ServletContext起始参数的两种方法
使用ServletContext对象，如：
{% highlight java %}
ServeltContext con=getServletContext();
String init_value=con.getInitParameter("init_name");
{% endhighlight %}

使用ServletConfig对象
{% highlight java %}
ServletConfig config=getServletConfig();
ServletContext con=config.getServletContext();
String init_value=con.getInitParameter("init_name");
{% endhighlight %}

#### ServletContext接口提供的日志方法
log()
{% highlight java %}
public void log(String msg);
public void log(String msg,Throwable t);
{% endhighlight %}

第二个方法在记录时会把异常也一起记录，这个方法可以在catch块中使用。

{% highlight java %}
public Object getAttribute(String name)
public void setAttribute(String name,Object o)
public java.net.URL getResource(String path)throws java.net.MalformedURLException
{% endhighlight %}

例如：

{% highlight java %}
URL url=con.getResource("/list.jsp");
{% endhighlight %}

public InputStream getResourceAsStream(String fileName)
例如：

{% highlight java %}
BufferedReader br=new BufferedReader(new InputStreamReader(con.getResourceAsStream("myFile.txt")));
{% endhighlight %}
