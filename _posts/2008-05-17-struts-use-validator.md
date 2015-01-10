---
layout: post
title: Struts 集成 Validator
tags:
  - java 笔记
---

使用validator框架

- 集成Validator(略）
- 使用动态表单org.apache.struts.validator.DynaValidatorForm
- 设置表单
{% highlight java %}
<form-bean name="user" type="org.apache.struts.validator.DynaValidatorForm">
  <form-property name="userId" type="java.lang.String"/>
....
</form-bean>
{% endhighlight %}
- action不用说了
- 注意使用动态验证一样可以在每个输入框后面显示，但是<html:errors>的property必须和表单的property一致
- 注意在写validation.xml时，注意每个validator的msg字段，这是错误的key，如果使用argn
传递参数，想要显示错误，那么在properties文件中必须有和validator中对应的key/value
例如：在validator-rules.xml中required的msg为errors.required，那么在.properties文件中必须有
errors.required={0} is required当然如果你不需要传递参数，可以直接是如：UserName is required
validation.xml中
{% highlight java %}
<form-validation>
 <formset>
  <form name="user">
   <field property="userId" depends="required">
    <arg0 key="leisure.user"/>
   </field>
  </form>
 </formset>
</form-validation>
{% endhighlight %}

你也可以修改validator-rules.xml文件验证器的msg的值，但是必须保证在你的.properties属性文件中有对应的key 这样才能正常显示错误信息。
