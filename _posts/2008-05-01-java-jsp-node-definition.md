---
layout: post
title: Java JSP 自定义标签
tags:
  - java 笔记
---

#### JSP自定义标签库（javax.servlet.jsp.tagext包）
- 撰写标签处理类，通常都是extends TagSupport类或者BodyTagSupport类
- 撰写TLD(自定义标签库描述文件.tld）

#### Tag接口

{% highlight java %}
方法：                                 返回值
doStartTag()                           int : EVAL_BODY
                                             SKIP_BODY
doEndTag()                             int : SKIP_PAGE
                                             EVAL_PAGE
release()                              void
{% endhighlight %}

#### IterationTag接口
extends Tag，添加的方法

{% highlight java %}
int doAfterBody()
{% endhighlight %}

只要运行了自定义标签主体就会运行doAfterBody()方法。

#### TagSupport类
implements IterationTag，生命周期：
- doStartTag()
- doAfterBody()//这个方法都不用，因为如果要实现有标签体的标签则应该实现，BodyTagSupport类，而不是TagSupport类。
- doEndTag()
- release()

#### BodyTagSupport类
extends TagSupport implements BodyTag，生命周期：
- doInitBody()//初始化BodyContent对象
- doStartTag()//这里由于实现了BodyTag接口所以有三个返回值：SKIP_BODY, EVAL_BODY_INCLUDE, EVAL_BODY_BUFFERED，用标签处理结果建立一个BodyContent对象
- doAfterBody()
- doEndTag()
- release()

#### 撰写一个标签处理类的要求：
- import javax.servlet.jsp.tagext.*;
- 必须为每个属性书写设置器。
- doStartTag()方法，如果定义的是空标签则必须返回SKIP_BODY。
- 如果不是则应在doAfterBody()中撰写相应的代码。                                                      
- doEndTag()方法通常都让它返回EVAL_PAGE。
- release()方法中通常撰写回复属性初始值的代码。

撰写中可能使用到的有用的类：JspWriter out =pageContext.getOut(); pageContext是TagSupport类中定义的一个属性，它是PageContext的对象javax.servlet.jsp.PageContext

PageContext类中的常用的方法：
- findAttribute()//在page,request,session,application作用域中查找
- getAttribute()//在当前JSP网页中寻找。
- getServletContext()
- getSession()
- getRequest()

#### 标签库描述文件（TLD)
TLD是标准的xml文件

{% highlight java %}
  <taglib>
   <tlib-version>1.0</tlib-version>
   <jsp-version>2.0</jsp-version>
   <description>this si....</description>
   <short-name>myT</short-name>
   <uri>http://leisure/taglib</uri>
   <tag> 
     <name>FirstTag</name>
     <tag-class>leisure.taglib.FirstTag</tag-class>
     <body-content>empty</body-content>
     <description>this is....</description>
     <attribute>
       <name>
       <type>
       <requried>
       <rtexprvale>
     </attribute>
   </tag>
  </taglib>
{% endhighlight %}

注：body-content可以取empty / jsp  /  tagdependent三个值//jsp表示可以是任何合法的jsp代码，tagdependent表示容器不会处理它，只会将它交给标签处理类 <tag>子元素： <required>可以为true或false <rtexprvalue>表明属性值是否可以在运行期间产生true或false

#### 使用自定义标签库
- 将标签处理类打包成jar文件，放在lib目录下。
- 将tld描述文件放在WEB-INF目录下。
- 在web.xml文件中声明tld。
{% highlight java %}
     <taglib>
        <taglib-uri>http://leisure/taglib</taglib-uri>
        <taglib-location>/WEB-INF/firstTag.tld</taglib-location>
     </taglib>
{% endhighlight %}
- 在JSP页面中使用。 <%@ taglib uri="http://leisure/taglib" prefix="myTag" %> 使用如下： <myTag:FirstTag ...../>空标签
