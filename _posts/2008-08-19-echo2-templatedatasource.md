---
layout: post
title: Echo2 中的 TemplateDataSource
tags:
  - java 笔记
---

最近在学习Echo2，使用模板时总有不顺，但是也有一点点心得。

TemplateDataSource接口由一个实现类是AbstractTemplateDataSource，后者有四个直接子类：

- FileTemplateDataSource//使用的为文件路径
- JspTemplateDataSource //使用jsp文件在类路径下，但是至今没有解析成功
- ResourceTemplateDataSource//使用类路径下的文件，默认格式为xhtml
- StringTemplateDataSource //使用字符串。

中文支持，当在html模板文件中使用中文时，会显示乱码，尽管你在html模板文件中设置的编码支持中文，但是解析器的默认编码为iso-8859-1,所以会是乱码，可以使用以上一个类的方法setEncoding(String encode)来设置编码，注意和html模板文件中的编码一致。

发现在使用html模板文件时，添加了一个<table>后，居然解析不成功，它只是把table的内容给解析出来了，但是完全没有按照table的布局。

使用jsp文件的时候老是出错org.xml.sax.SAXParseException: Premature end of file.或者
White spaces are required between publicId and systemId.异常。
