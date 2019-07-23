---
layout: post
title: 在 kernel 代码中用 ctrlp 插件
tags:
  - vim
---

ctrlp vim 插件是一个可以快速打开文件的工具，输入部分文件名，就能自动找到，然后快速选择打开，
不用退出 vim 去找文件然后打开，相比 NERD Tree 这种文件浏览的插件也更方便。

但是，默认情况下 ctrlp 可能找不到你想要的文件，因为默认只会展示 10
个搜索结果，而想要的文件可能不在这 10 个里。

例如，要打开 fs/aio.c，默认的结果如下：

![ctrlp result](/assets/images/vim/ctrlp-search.png)

很显然不是想要的结果。

这里有两个办法：

1. 增加搜索的结果，默认是 10，增加到 100 就能满足需求，如果在增加，会感觉到明显变慢，因为每次搜索的量更大了

    ```bash
    let g:ctrlp_match_window = 'bottom,order:btt,min:1,max:10,results:100'
    ```

2. 使用正则匹配，可以添加配置

    ```bash
    let g:ctrlp_regexp = 1
    ```

    或者，在 ctrlp 界面，按 `c-r` 组合键来切换。
