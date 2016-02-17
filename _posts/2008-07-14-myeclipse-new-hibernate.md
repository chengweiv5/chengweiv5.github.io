---
layout: post
title: 在 MyEclipse 中添加 Hibernate 应用
tags:
  - java
---

在MyEclipse中添加Hibernate应用是很方便的，这里需要注意的有以下几点：
- 使用MySQL5数据库。
- MySQL方言有三种：MySQLDialect,MySQLInnoDBDialect,MySQLMyISAMDialect分别对应各种表格。
- 在使用MyEclipse创建数据库表的时候注意选择表的类型。通常选择InnoDB，它功能更强大。
- 在配置hibernate.cfg.xml文件的时候，一定要配置正确的dialect，否则将出现异常。
- 通常将映射文件的id设置成increment。
