---
layout: post
title: 解决 Java JSP 不能显示中文
tags:
  - java
---

在学习 JSP 的过程中，和大多数人一样，遇到一些中文乱码问题，这里记录了一些解决办法。

### JSP 页面中的中文乱码
解决办法很简单，只需要修改`<%@page/>`标签，修改为

{% highlight java %}
<%@page language="java" pageEncoding="gb2312"%>
{% endhighlight %}

### JSP 中传递 URL 参数时中文乱码
这个问题需要修改 tomcat 的配置文件 `$TOMCAT_HOME/conf/server.xml`，
在这个文件中找到如下行：

{% highlight java %}
<Connector port="8080" ......>
{% endhighlight %}

为 Connector 添加一个属性 URLEncoding="gb2312" 就可以了，因为默认 URL 的编码方式是 `ISO-8859-1`，
而这个编码方式不支持中文。

### 表单中的中文不能正常显示
这种情况是虽然你的几个页面都是 `gb2312` 或者 `utf-8` 但是当使用
`request.getParameter(String name)` 方法获得表单数据时一样不能正常显示中文。
这时可以使用 String 的一个构造方法 new String(request.getParameter(String name).getBytes("ISO-8859-1"), "gb2312")
来转换编码，这里的 `ISO-8859-1` 参数为提交表单的编码方式。

这种方法的弊端在于：如果表单很大时，就显得有点笨重，可以使用过滤器来实现对 Request 的统一编码，方法如下：

#### 编写一个过滤器类
下面是一个简单的转换类

{% highlight java %}
import javax.servlet.*;
import java.io.Exception;

public class MyFilter implements Filter {
    private FilterConfig config;
    private String charset = "gb2312";
    private isEncode = false;
   
    public void init(FilterConfig config) { //这个方法必须重写
        this.config = config;
        charset = this.config.getInitParameter("charset"); //获得在web.xml部署描述文件中自定义的字符集

        if(this.config.getInitParameter("convert").equalsIgnoreCase("true")) {
            isEncode = true; //如果配置成需要转换，则把标志变量置位
        } else {
            isEncode = false;
        }
    }

    public void doFilt(ServletRequest request,ServletResponse response,FilterChain chain)
    throws IOException,ServletException { //这个方法也是重写的，因为Filter是个abstract类
        if(isEncode) {
            request.setCharacterEncoding(charset);
            chain.doFilter(request,response);
        }
    }

    public void destroy() { //重写的方法，虽然没有任何实现

    }
} 
{% endhighlight %}

#### 修改 web.xml 文件，添加过滤器
下面是 `web.xml` 中配置的 `filter`, 它和 `<servlet>` 在一个层次。

{% highlight xml %}
<filter>
    <filter-name>Filter</filter-name>
    <filter-class>MyFilter</filter-class>
    <init-param>
        <param-name>charset</param-name>
        <param-value>"gb2312"</param-value>
    </init-param>
    <init-param>
        <param-name>convert</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

需要注意的是：两个 `<filter-name>` 的值必须一样，`<filter-class>` 是你的过滤类的完全类名。
`<init-param>` 标签定义初始参数；`<filter-mapping>` 中的 `<url-pattern>` 设置对于哪些 request
进行过滤。
