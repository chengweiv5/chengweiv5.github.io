---
layout: post
title: Java Servlet 基础（一）
tags:
  - java
---

通常我们写的Servlet都是继承自HttpServlet，同时由IDE生成的往往还有doGet和doPost方法，init()，destroy方法，这也是一个Servlet的生命周期的体现，其中最重要的就是service()方法，虽然我们不用重写service()因为对于http协议来说，我们通常都只用重写doGet()或doPost(),分别对应的http请求方式是GET和POST，而还有另外5种方式都不怎么用，也就是HEAD,PUT,DELETE

#### 如何取得http请求传递的参数

{% highlight java %}
String getParameter(String param_name);
String[] getParameterValues(String param_name);
Enumeration getParameterNames();
{% endhighlight %}

#### 获取http请求头

HttpServletRequest接口中定义了如下方法：

{% highlight java %}
public String getHeader(String name);
public Enumeration getHeaders(String name);
public Enumeration getHeaderValues();
{% endhighlight %}

#### 设置Servlet回应的通常方法：

{% highlight java %}
response.setContentType("text/html");
response.setCharacterEncoding("gb2312");
PrintWriter out= response.getWriter();//接下来将输出动态响应页面。
{% endhighlight %}

#### 如何设定Servlet回应Http请求的回应内容

{% highlight java %}
response.setContentType("text/html");
response.setCharacterEncoding("gb2312");
PrintWriter out=response.getWriter();
void setContentType(String s);
void setCharacterEncoding(String charset);
setContentType("text/html;gb2312");一次设定上面两个方法完成的任务。
PrintWriter getWriter();
ServletOutputStream getOutputStream();这两个方法用来获取输出流，通常需要动态生成网页内容是使用高级的PrintWriter。
void setHeader(String name,String value);//用来设定回应的头信息，通常都不需要调用， 不过当要输出文件时需要设定，如使用Servlet来实现文件下载。
void addHeader(String name,String value);
void sendError(int error_number);
void sendRedirect(String location);//这个方法比较常用，通常当servlet作为控制器时，使用这个方法来转发到显示页面。注意：如果servlet已经输出回应，这时如果在调用sendRedirect()方法将产生异常。
{% endhighlight %}

#### Servlet的生命周期

{% highlight java %}
init(ServletConfig config);
service();
destroy();
{% endhighlight %}

第一个方法是一个servlet实例化时调用的方法，如果需要给它传递初始化参数，则需要在web.xml文件中配置

{% highlight java %}
<servlet><servlet-name>MyServlet</servlet-name><servlet-class>.....</servlet-class>
  <init-param><param-name>paramname</param-name><param-value>....value</param-value></init-param>
{% endhighlight %}

可以配置多个<init-param>标签。第二个service()方法，servlet会跟据http请求的方式调用适当的方法来处理http请求，通常使用的是get和 post方法，所以通常也只需要在servlet中重写这两个方法如：

- public void doGet(HttpServletRequest request,HttpServletResponse response)throws ServletException,IOException{.........//do something} 对应如果使用post方法传递http请求，则重写doPost方法，声明方式和上面的doGet完全一样。
- destroy()方法，这个方法是servlet处理完请求后生命周期结束时调用的，通常都不用重写，也可以在其中释放资源之类的，但是通常都会有更好的方法。

#### 请求转送(request dispatcher)
ServletRequest接口中声明 public RequestDispatcher getRequestDispatcher(String location);//location 为将要接受请求的 jsp或者是servlet的路径。 如

{% highlight java %}
RequestDispatcher dp=request.getRequestDispatcher("index.jsp");
dp.forward(request,response);//完成请求转发。
{% endhighlight %}

ServletContext接口中也声明了一个和上面一样的方法，也可以用来转发，但是必须先获得ServletContext 对象，如：

{% highlight java %}
ServletContext context=getServletContext();
context.getRequestDispatcher(String path);
context.forward(request,response);//完成重定向
{% endhighlight %}

必须注意的是这个接口中的path必须以'/'开头，否则非法，而前面的location则没有限制。
ServletContext接口中还声明了一个方法

{% highlight java %}
public RequestDispatcher getNamedDispatcher(String name);
{% endhighlight %}

这个方法中的形参name接受的是在web.xml文件中定义的servlet实体的<servlet-name>的值。
