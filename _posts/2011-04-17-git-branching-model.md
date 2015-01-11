---
layout: post
title: 一种典型的 git 分支使用方法
tags:
  - git
---

以下是在根据网上一片 好文章 [A successful git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
而来的笔记。

#### decentralized but centralized
有一个origin repo，这个repo做为中心repo，其它开发者从它pull/push，而开发者之间可以组成小组，它们之间可以相互pull/push，例如几个人开发一个feature。

#### the main branches
origin/master, 发行版本，HEAD总是指向一个稳定的发行版本commit。

origin/develop, 当前开发分支，HEAD总是指向当前最新完成的开发，为下次的发行做准备。每次当从develop合并回master时，都必须严格控制，因为master总是代表产品，所以可以在合并点设置git hook。

#### supporting branches

其它分支可能包括：
- feature branches, 开发新特性
- release branches, 准备发行
- hotfix branches, bugfix，往往需要合并回develop, master等分支。

#### feature branches

从develop分支并合并回develop，feature branches的生命期是有限的，当开发完成后就合并回develop分支。feature branches往往不存在于origin repo，而是存在于开发者repo中。

{% highlight bash %}
$git checkout develop -b myfeature
... develop new features
$git checkout develop
$git merge --no-ff myfeature
$git branch -d myfeature
$git push origin develop
{% endhighlight %}

使用--no-ff表示no fast forward，这样总是生产一个commit对象，这样就能够知道这个feature是由那些commit带来的，而不是要一个个的检查develop分支中的commit来猜测。

#### release branches

从develop分支，合并回develop和master。release分支在准备发行新的版本时创建，这是所有为这个发行版本准备的feature都已经合并到了develop中，而为将来发行版本的feature可能还在其它feature branches中，当生产release分支后，develop就可以开始接受将来发行版的feature了。

{% highlight bash %}
$git checkout develop -b release-1.2
{% endhighlight %}

... 现在准备发行新版本，这时，新的feature不允许加入release分支中，而只能合并到develop中作为下一个发行版的feature。而只能接受一些bugfix。

当版本准备好发行时，就可以合并回origin master分支，然后在origin master分支中打一个tag；然后合并回develop分支，以使develop分支也有在release branch中的bugfix。

{% highlight bash %}
$git checkout master
$git merge --no-ff release-1.2
$git tag -a 1.2
$git checkout develop
$git merge --no-ff release-1.2
$git branch -d release-1.2
{% endhighlight %}

#### hotfix branches

从master分支，合并回master和develop分支。例如当前发行版本为1.2，发现了重要bug。

{% highlight bash %}
$git checkout master -b hotfix-1.2.1
... 现在修复这些bug，然后提交
$git commit -a -m "fixed bugs..."
$git checkout master
$git merge --no-ff hotfix-1.2.1
$git tag -a 1.2.1
{% endhighlight %}

当修复了这些bug后，发行版本的版本号一定要改变，不然怎么区别bugfix之前和之后的版本。然后将这些bugfix合并回develop分支

{% highlight bash %}
$git checkout develop
$git merge --no-ff hotfix-1.2.1
$git branch -d hotfix-1.2.1
{% endhighlight %}

另外，为了每次在合并时都是用--no-ff选项，可以在分支上设置合并选项

{% highlight bash %}
[branch "master"]
mergeoptions = --no-ff
{% endhighlight %}

或

{% highlight bash %}
$ git config branch.developer.mergeoptions "--no-ff"
{% endhighlight %}
