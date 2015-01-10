---
layout: post
title: Java Servlet 基础（六）：线程安全
tags:
  - java 笔记
---

Servlet运行的原理：每个Servlet类只有一个实体，当收到http请求时，容器会启动一个相应的Servlet线程来处理，这个线程主要会执行Servlet的service方法，从而调用相应的方法来回应请求。

线程安全的变量：
- 局部变量（如在doGet或doPost等方法中声明的变量）
- HttpServletRequest变量（因为每个请求都将请求封装成HttpServletRequest 对象，而传递给doGet或doPost方法处理，所以也就相当于局部变量。

非线程安全的变量：
- HttpSession变量
- 成员变量
- ServletContext变量

处理非线程安全的变量时，使用synchronized来同步对变量的操作。synchronized可以用来修饰方法或者代码块。
