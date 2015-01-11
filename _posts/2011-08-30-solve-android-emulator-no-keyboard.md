---
layout: post
title: Android 模拟器没有键盘的解决方法
tags:
  - android
---

刚开始使用android模拟器的时候，发现自己创建的AVD启动后没有出现侧边的键盘，在网上搜索后，发现很多人都有这个问题，也有文章说直接使用PC上的键盘，因为有对应的快捷键。但是，没有键盘，始终不爽！

问题的原因在于自定义AVD时没有选择built-in的skin导致的，编辑相应的AVD，选择built-in的skin。
