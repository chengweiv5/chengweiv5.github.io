---
layout: post
title: Struts 中使用 <html:cancel>
tags:
  - java
---

Struts中使用<html:cancel>操作，以前看书上都有说的，但是都没有说出最关键的一点，必须在可以处理cancel的action中配置一个属性：<set-property property="cancellable" value="true"/>因为默认为false所以即使你使用了取消，一样没有用。
