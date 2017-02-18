---
layout: post
title: Mutt 邮件客户端显示的邮件时间为 UTC
tags:
    - linux
    - mutt
---

偶尔会看到 mutt 邮件客户端把刚刚收到的邮件显示成了看起来已经收到几个小时了，所以会无意识的被吓一跳：我擦，怎么刚刚才看到。

打开收到的邮件，在邮件头部的 Date 里可以看到类似如下字样

![email date](/assets/images/mutt/email-header-date.png)

可以看到 Date 显示的时区为 +0000，那么这个 +0000 是否代表发件人的时区呢？

根据 [RFC2822 3.6.1 节](https://tools.ietf.org/html/rfc2822#section-3.6.1) 中的定义：

![rfc2822-3.6.1](/assets/images/mutt/rfc2822-3.6.1-origination-date.png)

所以，origination date 指的是发件人发送邮件的时间，并不一定是开始传输的时间，也可能是在离线环境下，
将邮件放入发送队列的时间。

下面来看下 orig-date 的定义 `orig-date       =       "Date:" date-time CRLF`

其中 `Date:` 就是我们在 mutt 中看到的关键字，CRLF 分别表示：Carriage Return (ASCII 13, \r) Line Feed (ASCII 10, \n)）。

接着看下 date-time 的定义，参考 [RFC2822 3.3 节](https://tools.ietf.org/html/rfc2822#section-3.3)。

![rfc2822 3.3](/assets/images/mutt/rfc2822-3.3-date-time-definition.png)

注意其中的 time 的定义，包含了时区（zone）这个概念。

从而，也就能够明白为什么刚刚收到的邮件，mutt 在邮件列表中显示时会把它显示得靠前了，因为发件人不正确的发件客户端时区设置导致。
而上面理解的那封凌晨 2 点多发的邮件，其实是本地时间上午 10 点多发送的。
