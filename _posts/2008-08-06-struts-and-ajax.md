---
layout: post
title: Struts 集成 Ajax
tags:
  - java 笔记
---

看了一本书讲解了Struts集成Ajax，主要是在表单验证方面，这方面Ajax也很有优势。但是并没有使用到DWR。

大概思路：

首先当然得有一个输入页面registe.jsp，它包含一个表单

{% highlight java %}
<html:form action="registe.do">
  <html:text property="name" id="nameId"/>
  ...
</html:form>
{% endhighlight %}

然后使用两个action来处理这个表单，因为要使用一个来处理常规提交，另一个来处理Ajax验证。它们都关联一个form-bean;

{% highlight java %}
<form-bean name="registeBean" type="forms.RegisteForm"/>
<action path="/registe" name="registeBean" scope="request" input="/jsp/registe.jsp" validate="true" type="actions.RegisteAction">
    <forward name="suc" path="/jsp/suc.jsp"/>
</action>
<!--注意下面的input属性值-->
<action path="/validate" name="registeBean" scope="request" input="/jsp/error.jsp" validate="true" type="actions.RegisteValidateAction">
    <forward name="valid" path="/jsp/valid.jsp"/>
    <forward name="invalid" path="/jsp/error.jsp"/>
</action>
{% endhighlight %}

使用validator框架来验证，注意RegisteForm继承自ValidatorActionForm而非ValidatorForm这样在validation.xml文件中的form的name属性值应该和路径匹配，如下

{% highlight java %}
<form name="/validate">
  ...
</form>
<form name="/registe">
  ...
</form>
{% endhighlight %}
这样就可以了。
