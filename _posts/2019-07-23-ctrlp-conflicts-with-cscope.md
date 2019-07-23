---
layout: post
title: vim ctrlp 插件和 cscope 插件冲突
tags:
  - vim
---

最近遇到一个奇怪的事情，到最后解决办法也是瞎猫碰见死耗子，不知道真正是什么原因。
事情是这样的，使用 spf13-vim 发行版时，同时开启了 cscope 和 ctrlp 插件，
然后发现 cscope 跳转到函数定义的时候会提示找不到文件，用 `:pwd` 在 vim 里显示，
当前目录切换了。

由于工作目录切换了，所以 cscope
自然找不到文件，因为是用的相对目录，那么为什么会自动切换目录呢？

一系列 Google 之后，发现主要有几个地方可疑：

1. vim 本身有一个 autochdir 的功能，可以关闭，在 `~/.vimrc` 里设置 `set noautochdir` 即可
2. spf13-vim 有一个配置 `g:spf13_no_autochdir`，设置为 1，`let g:spf13_no_autochdir=1`
3. ctrlp 有一个 `ctrlp_working_path_mode` 配置，默认是 *'ra'*

尝试了一些列各种组合配置之后，发现还是不行，使用 ctrlp
打开一个文件之后，目录会切换到该文件所在的目录。

最后，通过删除 `~/.vimviews` 和 `~/.viminfo` 目录解决了问题。
