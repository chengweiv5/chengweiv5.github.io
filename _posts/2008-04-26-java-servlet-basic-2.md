---
layout: post
title: Java Servlet 基础（二）
tags:
  - java
---

Java Servlet Web 应用程序的结构和部署

#### WEB-INF
WEB-INF目录有特殊的用处，不能使用url直接访问它下面的资源，即使你把一个jsp或是一个servlet放在WEB-INF目录下，使用http://localhost:8080/MyApp/WEB-INF/MyServlet来访问MyServlet也会找不到资源。WEB-INF下的classes是存放Servlet编译好的class文件，而lib则是存放.jar文件的目录如数据库驱动和其它需要使用的外部.jar文件。

#### web.xml
放在WEB-INF下面，而且也只能放在那里，它是整个应用程序的部署描述文件，通常包含下列元素。<web-app>是根元素，下面是<web-app>下常用的元素；

{% highlight java %}
<servlet>
  <servlet-name>....</servlet-name>
  <servlet-class>.....</servlet-class>
  <!--可以有多个<init-param>元素-->
  <init-param>
    <param-name>......<param-name>
    <param-value>.....</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>必须对等一个<servlet>中的<serlvet-name>
  <url-pattern>

<!--可以有多个context-param-->
<context-param>
  <param-name>Context的起始参数名</param-name>
  <param-value>

<filter>
  <filter-name>
  <filter-class>
  <init-param>
    <param-name>
    <param-value>
  <init-param>....
  <!--可以有多个<init-param>初始化多个参数
<filter-mapping>
  <filter-name>
  <url-pattern>
<welcom-file-list>
  <!--可以有多个welcom-file,容器会使用找到的最前面一个作为欢迎页面-->
  <welcom-file>.......</welcom-file>
  <welcom-file>.......</welcom-file>
{% endhighlight %}

#### ServletConfig
在初始化一个Servlet的时候会向init()方法传递一个ServletConfig参数，在Servlet中使用下面的方法来获取初始参数值：

{% highlight java %}
ServletConfig    sc=getServletConfig();        
String      value=sc.getInitParameter(String     name);
String value=getInitParameter(String name);
{% endhighlight %}

#### 建立war文件
在命令行下切换到应用程序目录下

{% highlight java %}
jar    cvf     MyApp.jar   *.*
{% endhighlight %}

参数：
- C  建立新的jar文件
- T  列出jar文件的目录
- U  更新现有的jar文件
- V  将建立过程信息输出
- F  指定jar文件的名称

#### 访问某个Servlet的三种方式

使用Servlet的全限定名称如有Servlet:leisure.servlet.MyServlet 则可使用：http://localhost:8080/myapp/servlet/leisure.servlet.MyServlet
使用Servlet实例在web.xml文件中的<servlet-name>MyServlet 如：http://localhost:8080/myapp/servlet/MyServlet 1和2中url中的servlet/对应的是WEB-INF/classes/路径
使用web.xml中<servlet-mapping>下的<url-pattern>/Hello 如：http://localhost:8080/myapp/Hello
