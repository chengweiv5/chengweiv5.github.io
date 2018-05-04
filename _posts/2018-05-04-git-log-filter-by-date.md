---
layout: post
title: 为什么 Git 根据日期来过滤 commit 的结果看起来是错的？
tags:
  - git
---

今天有个需求，想要查看某个同学在某段时间内的开发工作，从 Gitlab 上的 API
虽然也可以导出，但是出来的结果不够直观，需要做好些过滤的工作才能开到。

所以，尝试直接使用 git log 来过滤 commit，第一直觉就是用

```
$ git log --author=zhangsan --after="2017-12-31" --before="2018-04-01"
```

so easy！

但是，出来的效果却令人大吃一惊！

结果里居然有 **2015** 年的 commit！

这是因为：git 默认显示的是 **author date（%ai）**，包括 tig 这类工具，
所以，我们看到的是真正 author commit 的时间。

而 git log 根据日期时间过滤的时候，却是根据 **commit date（%ci）** 的，
而 commit date 是该 commit 被 merge 的时候，或者 rebase 的时候的时间。

可以通过下面的命令查看

```
$ git log --format=format:"%ai, %ci %aE %s"
```

上面的命令中：

- %ai，author date
- %ci，commit date
- %aE，是 author email 的缩写
- %s，是 commit subject

这是，就可以发现前面的 2015 年的 commit 因为最近一次升级项目做了 rebase，导致其
commit date 实际是 2018 年，所以就被前面的 git log 过滤出来了。

现在还不清楚为什么 git log 默认显示 author date，却在过滤的时候用 commit date，
导致用户看到的感觉是拿到了错误的结果。
