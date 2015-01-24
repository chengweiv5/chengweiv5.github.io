---
layout: post
title: "GitHub: 学习使用 Hub"
tags:
  - github
---

对于大多数使用 Git 作为 VCS 的程序员来说，应该都接触过 [GitHub](https://github.com)，
GitHub 就像程序员的淘宝一样，里面充满了好东西，时时刻刻都可能给你惊喜！

很多人可能不仅在 GitHub 寻找合适的车轮子，还可能会为造车轮子贡献自己的力量，
往往会使用一些基本操作来完成，典型的为：

- fork
- PR(pull request)

当然，如果是项目的维护者，还会使用 `merge` 等。

但是，我想很少人会使用过 GitHub 的命令行接口 [Hub](https://github.com/github/hub)，
通常的操作都可以通过友好的 Web 界面，点几个 button 来完成，简单实用！
所以很少有需求会迫切需要一个命令行工具来完成这些操作，我也是在想要清理一下自己的
repositories 的时候，发现一个一个在 Web 上来清理确实不够高效，如果有命令行接口的话，
可以很快进行批量操作，或许需要加几行 shell 命令等，那也是极好的。

所以，在 Google 之后（不要问我怎样翻墙，首选 [GoAgent](https://github.com/goagent/goagent)），
我发现真的有 GitHub 的命令行工具 [Hub](https://github.com/github/hub)，主页上是这样介绍的：

> git + hub = github

Hub 是对 git 的一层封装，以便和 GitHub 完美结合，而且号称对 git 完全兼容，而且推荐直接将
hub 设置为 git 的别名，所以在执行 git 的时候实际上是在执行 hub。别名设置如下：

{% highlight bash %}
eval "$(hub alias -s)"
{% endhighlight %}

Hub 主页上有完善的文档教用户怎样安装、设置、使用命令详解等等，这里不再赘述。
为了勾起读者的兴趣，简单介绍一下怎样使用 hub 来进行最常见的在 Github 上的开发。

这里我们直接进入正题，介绍安装和设置比较乏味！

### Fork 一个项目
要在 GitHub 上进行开发，往往会基于一个已有的开源项目，所以首先需要 fork 这个项目。

{% highlight bash %}
$ git clone github/hub
Cloning into 'hub'...
remote: Counting objects: 10646, done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 10646 (delta 4), reused 0 (delta 0)
Receiving objects: 100% (10646/10646), 3.25 MiB | 58.00 KiB/s, done.
Resolving deltas: 100% (6302/6302), done.
Checking connectivity... done.
$ cd hub/
$ git fork
Updating chengweiv5
From git://github.com/github/hub
 * [new branch]      1.11-stable -> chengweiv5/1.11-stable
 * [new branch]      1.12-stable -> chengweiv5/1.12-stable
 * [new branch]      gh-pages   -> chengweiv5/gh-pages
 * [new branch]      master     -> chengweiv5/master
 * [new branch]      skip_completion_script_for_windows -> chengweiv5/skip_completion_script_for_windows
new remote: chengweiv5
{% endhighlight %}

这里和 Web 上的操作有点不同，从 Web 上是首先找到一个项目，然后点击一下 `Fork`，
然后会在自己的空间内创建这个项目。

而使用 hub, 则首先是 clone 下来原有的项目（以 hub 项目为例，git clone github/hub），然后再执行
`fork` 子命令，完成后，可以看到本地添加了一个 remote，而且通过 web
页面也可以看到自己的空间里已经添加了一个叫 hub 的项目，fork 自 github/hub。

### PR(Pull Request)
在本地完成一些开发后，可能想要将 patch 提交给 upstream 项目，在 GitHub 中，向上游提交 patch
通过 PR 来完成。

当然，这里我不会向 hub 提交 PR，因为我只是在写博客，在吃过午饭后，找到一个很不错的项目，
而且发现一个非常小的 bug，可以作为博客素材了。如下：

{% highlight bash %}
$ git clone sb2nov/mac-setup
Cloning into 'mac-setup'...
remote: Counting objects: 1635, done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 1635 (delta 33), reused 0 (delta 0)
Receiving objects: 100% (1635/1635), 3.69 MiB | 59.00 KiB/s, done.
Resolving deltas: 100% (941/941), done.
Checking connectivity... done.
$ cd mac-setup
$ git fork
{% endhighlight %}

完成 fork 后，我会修改一处文档，diff 如下：

{% highlight bash %}
$ git diff
diff --git a/SystemPreferences/README.md b/SystemPreferences/README.md
index a148d74..a7ff953 100644
--- a/SystemPreferences/README.md
+++ b/SystemPreferences/README.md
@@ -1,7 +1,7 @@
 # System Preferences
 
 First thing you need to do, on any OS actually, is update the system! For that: **Apple Icon > Software Update.**
-Also upgrade your OS incase you want to work on the latest OS. Mavericks is a free upgrade so please check that.
+Also upgrade your OS incase you want to work on the latest OS. Yosemite is a free upgrade so please check that.
 
 If this is a new computer, there are a couple tweaks you would like to make to the System Preferences. Feel free to follow these, or to ignore them, depending on your personal preferences.
 
{% endhighlight %}

`git pull-request` 会检查你在 GitHub 上的自己的项目和上游项目相应的 branch 是否有不同，所以，
首先将这个修改提交到自己的项目中，push 就行。

{% highlight bash %}
$ git commit -asm "Yosemite is the latest Mac OS X now"
$ git push chengweiv5
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 391 bytes | 0 bytes/s, done.
Total 4 (delta 2), reused 0 (delta 0)
To git@github.com:chengweiv5/mac-setup.git
   16df764..e25031f  master -> master
{% endhighlight %}

然后，提交 PR，如下：

{% highlight bash %}
$ git pull-request 
https://github.com/sb2nov/mac-setup/pull/27
{% endhighlight %}

`hub` 还有许多有用的命令，例如打开浏览器查看项目，merge PR，新建 repo 等等，这里不再介绍，
感兴趣的读者可以参考 Hub 文档。

最后，学习了 hub 的使用，但是却没能满足最初的需求：从命令行删除 repository，
不过最后还是找到办法了，需要使用 github 的 API 来操作，参考：
[stackoverflow](https://stackoverflow.com/questions/19319516/how-to-delete-a-github-repo-using-the-api)。
