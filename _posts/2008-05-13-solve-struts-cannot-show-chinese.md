---
layout: post
title: Struts 中乱码完全解决方法
tags:
  - java
---

可能很多人都遇到过和我一样的问题：不知道是什么原因，在写Servlet和Jsp的时候也是乱码不能解决，因为写的过滤器根本就没有起作用。现在到了Struts，还是有这个问题。郁闷好久了，上网搜了好久都是使用过滤器看起来最方便，其实也是。但是还是不起作用，我的方法都是post不知道为什么。今天深入了解Struts工作原理后，发现有一个地方可以设定编码，当然不是指的controller里面，在controller里面设定了contentType="text/html;gb2312"结果也没有用。而且我所有页面也统一使用gb2312。

下面大概说一下Struts处理request的过程，当然第一是被ActionServlet控制器拦截请求，然后ActionServlet在doPost和doGet方法中都会调用一个具体干事的方法process(HttpServletRequest request,HttpServletResponse response)throws IOException,ServletException{...............}这个方法是RequestProcessor的方法，这个类是struts1.1中作为ActionServlet的助手类出现的，大部分工作都交给它处理了。在它的process（）方法中会调用16个方法

{% highlight java %}
processMultipart()   //判断是否是multipart/form-data
processPath()   //获得路径
processLocale()  //是否读取请求中的locale信息和处理
processContent()   //设置回应的编码
processNoCache()
processPreprocess()   //预处理没有任何编码，直接返回true
processMapping()
processRoles()
processActionForm()   //处理表单，先调用reset方法，然后组装form
processPopulate()
processValidate()
processForward()
processInclude()
processActionCreate()
processActionPerform()
processActionForward()
{% endhighlight %}

方法一：注意processActionForm()方法，这个方法会先调用ActionForm的reset方法，所以你可以在你的ActionForm中重写reset方法，在其中进行编码设定如：request.setCharacterEncoding("utf-8");这里千万要注意的是，你设置的编码和发送请求的页面的编码必须，注意是必须一致。一下几个方法也是相同的。如果要使用Validator插件，我想使用了Struts都会用上的。那么在Validator中表单是使用ValidatorForm和DynaValidatorForm，他们都是ActionForm的子类，你也可以重写reset方法，当然最方便的还是写一个自己的基类扩展它们，而在其中只用重写reset（）而不做其它操作，然后其他表单继承这个基类。这种方式的缺点是只能解决表单中文乱码问题。

方法二：查看源码可以发现上面RequestProcessor中的方法很多参数都是HttpServletRequest和HttpServletResponse，所以第二种方法是扩张RequestProcessor类，重写某个方法，添加编码的代码。如重写processPreprocessor()方法，这个方法在源码中没有任何逻辑操作，只是返回一个true
重写如下

{% highlight java %}
public boolean processPreprocess(HttpServletRequest request,HttpServletResponse response){
 try{
  request.setCharacterEncoding("utf-8");
 }catch(UnSupportedEncodingException e){
  e.printStackTrace()
 }
 return true;
}
{% endhighlight %}

还可以重写processContent(HttpServletRequest,HttpServletResponse)等方法，但是推荐还是重写上面的processPreprocess方法，因为里面没有任何逻辑代码，所以你不用担心会破环原来的处理过程。现在要做的就是在struts-config.xml中设定控制器如下：
<controller processorClass="leisure.util.MyProcessor"/>重启tomcat验证

方法三：设置过滤器，这个方法是用的最郁闷的，但是不能否认还是很方便1、写一个实现了javax.servlet.Filter接口的类

{% highlight java %}
public class MyFilter implements Filter {
 private FilterConfig config;
 private String charset;
 private boolean isEncode;
 public void init(FilterConfig config){
  this.config=config;
  charset=config.getInitParameter("char");
  if(config.getInitParameter("encode").equalsIgnoreCase("true")){
   isEncode=true;
  }
 }
 public void doFilter(ServletRequest request,ServletResponse response,
   FilterChain chain){
  try{
   ((HttpServletRequest)request).setCharacterEncoding(charset);
  }catch(UnsupportedEncodingException e){
   e.printStackTrace();
  }
  try{
   chain.doFilter(request, response);
  }catch(ServletException e){
   e.printStackTrace();
  }catch(IOException e){
   e.printStackTrace();
  }
 }
 public void destroy(){
  config=null;
  charset=null;
  isEncode=false;
 }
}
{% endhighlight %}

第二是配置这个过滤器，在web.xml文件中加入如下语句

{% highlight java %}
<filter>
  <filter-name>filter</filter-name>
  <filter-class>leisure.util.MyFilter</filter-class>
  <init-param>
    <param-name>char</param-name>
    <param-value>utf-8</param-value>
  </init-param>
  <init-param>
    <param-name>encode</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>myFilter</filter-name>
  <url-pattern>*.do</url-pattern>
</filter-mapping>
{% endhighlight %}

最后千万要注意的是：你所设定的支持中文的编码一定要和页面的编码一致。否则一样是乱码！
