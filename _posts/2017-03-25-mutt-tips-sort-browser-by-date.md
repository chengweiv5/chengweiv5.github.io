---
layout: post
title: 让 Mutt 邮件客户端将收件箱按时间倒序排序
tags:
    - mutt
    - tips
---

从工作以来就一直使用 [mutt](http://www.mutt.org/) 作为邮件客户端，
借用 mutt 作者的话说："All mail clients suck. This one just sucks less."

由于使用了很多个收件箱来归类邮件，所以对于收件箱的排序来说，
我个人倾向于按照收件箱时间倒序排列，这样，最近有新邮件的收件箱总是排在最前面，
方便查阅。

默认情况下，mutt 对收件箱的排序采用字符序，所以你可以通过类似在邮箱前面加一些排序字符，例如：

```
00-from-boss
01-very-important
02-...
```

这样，重要的收件箱总是排在前面，但是不好的是，这个顺序是固定的。

而以灵活性著称的 mutt，只有想不到，没有做不到，所以，不出所料，
是能够根据时间来排序收件箱的。

查看 mutt 文档，如下：

```
3.269. sort_browser

Type: sort order
Default: alpha

Specifies how to sort entries in the file browser. By default, the entries are
sorted alphabetically. Valid values:

  • alpha (alphabetically)

  • date

  • size

  • unsorted

You may optionally use the “reverse-” prefix to specify reverse sorting order
(example: “set sort_browser=reverse-date”).
```

所以，只需要在 mutt 配置文件中加入下面的配置即可

```
set sort_browser=reverse-date
```

这样，当任何收件箱有新邮件到达时，mutt 总是将最新的收件箱排序在顶部，如下图所示：

![mutt sorted mailboxes](/assets/images/mutt/mutt-sort-browser.png)

如果指向临时改变排序，那么可以按 O 或者 Shift + O 键，前者表示正序，后者表示倒序，
按下键盘后，将出现排序选择，例如：按照时间排序，字符序，大小序等等。
