---
layout: post
title: Java Servlet 基础（四）：异常处理
tags:
  - java 笔记
---

#### 声明式异常处理
在web.xml文件中使用下述方式

{% highlight java %}
<error-page>
  <error-code>404</error-code>
  <location>/errors/notFound.html</location>
</error-page>
<error-page>
  <exception-type>java.io.IOException</exception-type>
  <location>/errors/ioexception.html</location>
</error-page>
{% endhighlight %}

上面是两种错误处理，注意location只能搭配一种，不能既有error-code又有exception-type
常见的错误代码：

{% highlight java %}
404  ：请求的资源未找到
500  ：服务器内部错误（程序异常或错误）
401  ：用户权限不足
403  ：服务器已经受到请求但是拒绝回应
408  ：连接超时
{% endhighlight %}

缺点：如果有很多中错误和异常，就必须定义很多错误处理文件。

#### 程序式处理
这种方式就是在可能发生异常的地方使用try---catch块，就和普通的java代码一样，
但是不能处理http请求异常。但是这种处理方式效率比较低，如果大型项目的话，就必须写很多异常处理代码，所以另一种方法是使用一个专门的servlet来处理异常。考虑如下

{% highlight java %}
try{
    //do something
}catch(IOException e){
    ServletContext context=getServletContext();
    RequestDispatcher rc=context.getNamedDispatcher(ExceptionHandlerServlet);
    request.setAttribute("exce",e);
    rc.forward(request,response);
}
{% endhighlight %}

然后在ExceptionHandlerServlet中在取出exce，使用instanceof来判断属于那一种异常，
然后执行相应的操作，如果你不判断而全都执行一种操作，也没什么，就是不利于发现程序中的错误，然后如果是Number的子类，则可以和error-code进行比较。

#### 记录方法log()
在ServletContext中提供了日志方法：

{% highlight java %}
  public void log(String msg)
  public void log(String msg,Throwable t)
{% endhighlight %}

第二个方法会记录异常栈，使用方法：

{% highlight java %}
ServletContext context=getServletContext();
context.log(s);
{% endhighlight %}

技巧：在catch块中使用。

在GenericServlet类中提供了log()方法：

{% highlight java %}
public void log(String msg)
public void log(String msg,Throwable t)
{% endhighlight %}
方法同上：但是由于HttpServlet继承自GenericServlet，所以在Servlet中可以直接使用log()方法，如：log(s); 两种log方法的区别：GenericServlet中的log方法会自动加上Servlet的名字，而ServletContext 中的log不会。
