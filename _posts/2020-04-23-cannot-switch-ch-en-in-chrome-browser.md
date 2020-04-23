---
layout: post
title: Google Chrome 浏览器中无法使用 shift 键切换 ibus-sunpinyin 中英文输入
tags:
  - chrome
  - ibus
---

之前有遇到过 ibus-pinyin 输入法会导致隐藏光标的[问题]({% post_url 2018-02-22-ibus-pinyin-hides-cursor %})，然后就切换到了
ibus-sunpinyin 输入法，后来又遇到 sunpinyin 输入法在 google chrome
浏览器中无法使用 shift 键切换中英文输入的问题。

在解决了 ibus-pinyin 隐藏光标的问题之后，果断切回 ibus-pinyin，因为 ibus-pinyin
在 google chrome 中是可以用 shift 切换中英文输入的，所以看起来是 google chrome
和 ibus-sunpinyin 之间有冲突，并且在 google chrome
的帮助网站里，也有不少人反馈这个[问题](https://support.google.com/chrome/thread/24298673?hl=en)，
但是无奈被关闭了，并没有解决。

后续应该可以看下 ibus-pinyin 和 ibus-sunpinyin 的代码，应该可以解决。
