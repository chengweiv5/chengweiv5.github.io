---
layout: post
title: Btrfs 文件系统基本概念（二）
tags:
  - btrfs
---

使用如下命令来获得源码

{% highlight bash %}
$git clone http://git.kernel.org/pub/scm/linux/kernel/git/mason/btrfs-progs-unstable.git
$cd btrfs-progs-unstable
$git checkout v0.2~90 -b 90
{% endhighlight %}

对于v0.2~106来说，v0.2~90的改变主要集中在增加了超级块以及对extent的支持，现在有两棵树，一棵root tree, 一棵extent tree，后者负责所有磁盘空间的管理。

#### 磁盘结构的改变

{% highlight bash %}
0-------------------------16------------------------17-----------------------18------------------------
| reserved    | super block               | root tree                    | extent tree
----------------------------------------------------------------------------------------------------------
{% endhighlight %}

首先，mkfs.c是创建文件系统的文件，它将初始化数据写入磁盘指定位置，如上图所示，现在extent tree中需要插入3个item来管理已经分配了的空间(0, 17), (17, 1), (18, 1)。空间二元组（起始地址，大小）。第一个extent实际上只有block16被超级块使用，前面的块保留。

#### 数据结构的改变

super block数据结构使用ctree_super_block来表示，它实际上只包括了两棵树，都使用结构ctree_root_info来表示，因为现在还没有实现索引节点对象等，所以超级块中的成员非常简单。

由于引入了extent支持，所以引入了新的数据结构extent_item。

{% highlight c %}
struct extent_item {
u32 refs; /* 被引用的次数，由于现在没有支持快照，子卷等，所以总是为1 */
u64 owner; /* owner的objectid */
};
{% endhighlight %}

#### extent的支持

现在集中在extent的实现上，首先引入了extent tree，它管理所有的磁盘空间，每个extent的分配都必须在extent tree中插入一个extent_item。包括它自己使用空间，当前分配的extent的大小都统一为1，除了第一个item(0,17)大小为17外。代码中最大的改变就是增加了extent-tree.c文件。包含以下几个函数

{% highlight c %}
static int del_pending_extents(struct ctree_root *extent_root);
{% endhighlight %}

将extent tree中标记为CTREE_EXTENT_PENDING的extent_item删除，并且清楚radix tree中相应tree_buffer的PENDING标记，从而释放空间。

{%  highlight c %}
int free_extent(struct ctree_root *root, u64 blocknr, u64 num_blocks);
{% endhighlight %}

函数中有两条逻辑，一条是对于其它树要释放空间，那么直接将extent tree中相应的extent_item删除即可，另一条是如果extent tree自己需要释放空间（删除节点），那么只将相应的extent在radix tree中标记为PEDING即可。稍后详细解释。

{% highlight bash %}
int find_free_extent(struct ctree_root *orig_root, u64 num_blocks, u64 search_start, u64 search_end, struct key *ins);
{% endhighlight %}

在extent tree中查找地址空间，是否能找到空闲空间。这里同样检查是否是extent tree需要分配空间。

{% highlight bash %}
static int insert_pending_extents(struct extent_root *extent_root);
{% endhighlight %}

将extent tree中标记为PENDING的tree_buffer对应的extent_item插入到extent tree中（磁盘中）。

{% highlight bash %}
int alloc_extent(struct ctree_root *root, u64 num_blocks, u64 search_start, u64 search_end, u64 owner, struct key *ins);
{% endhighlight %}

分配extent，调用find_free_extent()来搜索extent tree找到剩余空间，然后插入，也要判断是不是为extent tree自己分配空间。

{% highlight bash %}
struct tree_buffer *alloc_free_block(struct ctree_root *root);
{% endhighlight %}

为树分配tree_buffer，首先查找radix tree，如果不存在，则分配。

理解extent tree实现的难点在于：为什么有两条逻辑？一条是为其它树分配空间，一条是为extent tree分配空间；为前者分配直接将extent_item插入extent_tree即可，而为后者分配则需要PENDING。

实际情况是：extent tree的空间分配总是被动的，因为它不存储其它数据，只存储extent_item用来管理磁盘空间，所以只有在为其它树分配空间时或删除空间时，它才会被动的增长或缩小。现在只讨论一种情况：增长。

假设有磁盘如下，adcd都空闲。

{% highlight bash %}
---------------------------------------------------------------------------------------------------

... | a | b | c | d | ...

---------------------------------------------------------------------------------------------------
{% endhighlight %}

假设现在extent tree中没有任何PENDING的item.假设root tree要增长，那么它需要分配节点空间，假设它调用split_leaf()来新增节点。

a. split_leaf()，分裂节点，需要申请新的节点空间。

- -->alloc_free_block()，申请新的节点空间。
- -->alloc_extent()
- -->find_free_extent()，搜索extent tree。

c. -->判断是否为extent tree，这里为false，然后调用insert_item -->insert_item()，将相应的extent_item插入extent tree中

-->search_slot()，在extent tree中查找合适的插入点（叶子中）

b. -->split_leaf()，假设extent tree需要增长。

-->判断是否为extent tree, 这里为false

-->返回分配节点的tree_buffer对象

a点：tree root调用split_leaf()，b点：extent tree调用split_leaf()，现在沿着调用路径到了c点alloc_extent()函数中判断是否为extent tree时，为真。

-->判断是否为extent tree，为真

-->find_tree_block()，查找是否已经分配，为否

-->alloc_tree_block()，分配一个tree_buffer对象并插入radix_tree中

-->将相应的tree_buffer标记为PENDING。也就是实际上为extent tree分配的空间实际上没有在磁盘上执行，只是在内存中标记相应的磁盘空间为PENDING了。那么为什么不执行呢？因为现在extent tree的增长没有完成，它的增长是由于需要插入extent_item而引起的，如果你现在要落实到磁盘，那么又需要插入新的extent_item，如此将陷入无限递归！

现在，已经在内存中为extent tree分配了一个tree_buffer并被标记为PENDING插入到了radix_tree中，返回到split_leaf()，继续往下

d. split_leaf()

-->alloc_free_block()，返回tree_buffer，然后连接到树中，并且移动一些extent_item到其中。

-->write_tree_block()，将数据写入磁盘中。

完了吗？如果觉得完了，那就忽略了最重要的一件事了，我们为extent tree分配的那个节点现在已经通过write_tree_block()真正的写入到磁盘中了，但是我们在extent tree中还没有插入相应的extent_item,对吧？我们只是在radix tree中标记了磁盘块为PENDING，所以这就是insert_pending_extents()函数的作用，它把radix tree中所有标记为PENDING的extent对应的extent_item插入到磁盘中。那么，它是什么时候调用的呢？显然，它不能自己调用，否则插入extent_item时还是会递归，它是在alloc_extent()函数中，当为其它树分配空间时调用的。

现在，就真正完成了，删除操作和插入类似。
