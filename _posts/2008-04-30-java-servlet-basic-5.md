---
layout: post
title: Java Servlet 基础（五）：会话管理
tags:
  - java
---

常见的会话管理方式：
- 利用cookie
- 重写URL
- 使用SSL(secure socket layer)

#### 利用cookie
这种方式即是最常使用的方式既HttpSession对象。

{% highlight java %}
HttpServletRequest接口中定义了方法
public HttpSession getSession()
public HttpSession getSession(boolean bool)
{% endhighlight %}

当第二个方法接收的实参是true是，则和第一个方法完全一样，既如果当前没有建立会话，
就建立一个新的会话。当第二个方法接收的实参是false时，则即使没有会话也不会建立新会话。

#### HttpSession接口中的重要方法

{% highlight java %}
 public Object getAttribute(String name)
 public setAttribute(String name,Object o)
 public void removeAttribute(String name)
 public Enumeration getAttributes()
 public void invalidate()
 public void setMaxInactiveInterval(int timeout)
{% endhighlight %}

注意：
- 第一个方法用来获得HttpSession中的属性，注意转换成相应的对象。
- 第二个方法设置属性。
- 第三个方法一处属性，如果属性不存在则会抛出异常。
- 第四个方法获得全部属性的名称，可以使用Enumeration来遍历。
- 第五个和第六个方法都可以使当前会话无效。

第六个方法中的时间单位是秒，而非通常使用的毫秒。如果设定一个负数，则永不超时。还有一种方法可以设置时间：

{% highlight java %}
<session-config>
  <session-timeout>30</session-timeout>
</session-config>
{% endhighlight %}

#### 利用重写URL来管理会话
这种方法的优势是当客户端关闭cookie功能时，一样可以使用。原理就是利用HttpServletResponse 的encodeURL(nextServlet)方法来重写URL，在URL后面会加上一个jsessionid字符串。但是必须使用动态生成网页的技术，既jsp或servlet。比较麻烦，如：

{% highlight java %}
response.setContentType("text/html");
response.setCharacterEncoding("gbk");
PrintWriter out=response.getWriter();
out.println("<html><head><title>Overwrite URL</title></head><body><form action='");
out.println(" +response.encodeURL(ServletTwo)+"' method='post'...");
{% endhighlight %}

这里ServletTwo就是下一个接收请求的Servlet.可见这种方式相当麻烦。
