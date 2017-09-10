---
layout: post
title: 配置 mutt 显示 html 邮件时的换行
tags:
  - mutt
---

Mutt 是一个纯文本的邮件客户端，但是通过 mailcap 的配置，也能通过一些其它工具来显示 html，
将 html 转换成纯文本显示，通常来说，这个其它工具就是 w3m。

安装了 w3m 之后，它会提供 mime 能力，在 debian 上，w3m mime pakcage 位于
`/usr/lib/mime/packages/w3m`，文件内容如下：

```
text/html; /usr/bin/w3m -T text/html %s; needsterminal; description=HTML Text; nametemplate=%s.html; priority=4
text/html; /usr/bin/w3m -I %{charset} -dump -T text/html %s; copiousoutput; description=HTML Text; nametemplate=%s.html; priority=3
```

`update-mime` 命令会根据各个 mime package 以及一些其它配置，生成 `/etc/mailcap` 文件，
所以，通常来说，不要直接修改 `/etc/mailcap` 文件，否则会被再次生成文件的时候冲掉，
当然，`/etc/mailcap` 文件也提供了一个 user section，可以直接编辑并且不会被冲掉，
但是并不推荐这样用。

对于用户的 mailcap 配置来说，推荐是编辑 `~/.mailcap` 文件，这个文件具有更高的优先级。

这里，为了使 w3m 在 mutt 邮件客户端中部使用默认的 80 字节宽度，而是使用整个 terminal 的宽度，
需要修改下 mailcap 中的 w3m 命令，查看 w3m 帮助文档可以找到这个参数：

```
    -cols width      specify column width (used with -dump)
```

现在，需要查看 terminal 的宽度，可以使用 tput 命令：

```
$ tput cols
151
```

所以，在 `~/.mailcap` 中添加一行 w3m 的配置即可。

```
text/html; /usr/bin/w3m -I %{charset} -cols 151 -dump -T text/html %s; copiousoutput; description=HTML Text; nametemplate=%s.html
```
