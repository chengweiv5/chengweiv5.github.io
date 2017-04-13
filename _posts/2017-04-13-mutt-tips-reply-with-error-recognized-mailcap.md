---
layout: post
title: Mutt：怎样回复邮件中含有被错误识别的附件
tags:
    - mutt
---

才发现 mutt 回复邮件时，如果该邮件中包含了被错误识别的附件，例如：keynote 文件，
那么，keynote 会被转换成乱码，从而回复的时候之前的内容就会乱码了。

如下图所示：

![mutt attachment](/assets/images/mutt/mutt-attachments.png)

附件实际上是一个 keynote 文件，文件名结尾为 `.key`，但是被识别成了 `application/pgp-keys`，
那么，当你回复邮件的时候，mutt 会将其当做 `application/gpg-keys` 来显示，效果如下图所示：

![reply email](/assets/images/mutt/mutt-reply-email.png)

很显然，乱码了。

那么，怎样避免这种问题？

正确的办法当然是让 mutt 能够正确识别所有附件，或者是配置正确的 mailcap。

这里介绍一个绕过的办法：直接在回复邮件的时候剔除不能识别的附件即可。

首先，不要直接回复邮件（r, g 命令），而是直接查看邮件的附件（v），也就是进入如第一个图所示界面
然后，标记（t）能正确识别的附件，这里也就是第一个附件，`text/plain` 格式，实际上就是邮件正文
标记之后，直接使用 tag-reply 功能，也就是按下分号键，在 mutt 命令控制栏出现 `Tag-` 之后，按下 r 或 g 回复即可。
如下图所示：

![mutt tag reply](/assets/images/mutt/mutt-tag-reply.png)
