---
layout: post
title: Btrfs 文件系统基本概念理解(三)：持久化 snapshot 的实现
tags:
  - btrfs
---

btrfs-progs-unstable v0.2~68 实现了持久化的snapshot

重点理解btrfs_commit_transaction()

int btrfs_commit_transaction(struct btrfs_root *root, struct btrfs_super_block *s);

这个函数接受一个btrfs_root参数和btrfs_super_block参数，这里有三棵树存在：
- tree root
- extent root
- fs root

从定义层次来看：

tree root是一颗存储btrfs_root_item的树，而后者代表一棵树的完整snapshot。包括extent tree, fs tree，所以宏观上感觉tree root才是所有通用的tree参数，实际不然，通用的tree是fs tree，这也是因为文件操作才是最根本的。

btrfs_commit_transaction()

--> __commit_transaction(root)，将fs tree中的脏块写入到磁盘。

然后保存旧的snapshot key到snap_key，这里需要特别解释的是：fs tree通过COW机制首先会复制树根块的内容，然后在修改相应的内容，例如blocknr修改为新的COW块的块号，而其它的如key, btrfs_root_item等存储在磁盘中的数据并不会修改。保持旧snapshot的key后，将key++，得到标记现在这个snapshot的key。

-->btrfs_set_root_blocknr()，将旧的复制过来的btrfs_root_item对象的blocknr修改为COW块的块号，需要牢记的是：root对象是一个内存对象，在这里修改只会弄脏内存，所以不要认为修改了磁盘中的btrfs_root_item，也就是现在还在tree root中的btrfs_root_item对象。

-->btrfs_insert_root()，现在代表新snapshot的key, btrfs_root_item都已经在内存中生成了，调用此函数将其插入tree root中，同样，对于tree root也会进行写时复制。但是tree root并没有代表其自己的btrfs_root_item，所以不会陷入递归。

-->commit_extent_and_tree_roots()，确保extent tree的btrfs_root_item保持最新，这里注意：并不是像fs tree那样插入一个新的btrfs_root_item。

-->write_ctree_super()，将新的tree root的块号写入超级块中。

-->btrfs_finish_extent_commit()，调用两次分别将pinned的块写入磁盘。

修改fs tree的commit_root指向新的snapshot。

-->btrfs_drop_snapshot()，删除旧的snapshot。

-->btrfs_del_root()，删除tree root中旧snapshot的btrfs_root_item。

到了这里，是否都理解了呢？如果是的那么就错了，从整个流程来看，我们把旧的fs tree的snapshot删除了，但是tree root的呢？还有extent tree呢？这就是调用两次btrfs_finish_extent_commit()的原因，tree root和extent root有些特殊。

btrfs_finish_extent_commit()函数主要刷洗root->pinned_radix中的块到磁盘中。所以要理解tree root, extent root的COW怎样工作，必须从它们的pinned_radix入手。

这里剩下的疑问是：fs tree旧的block会被btrfs_drop_snapshot()以及btrfs_del_root()函数删除，那么root tree和extent tree就的块呢？什么时候删除的，这里需要从COW着手，当执行COW时，btrfs_cow_block()，会执行btrfs_free_extent()和btrfs_block_release()，而btrfs_free_extent()会根据extent的引用计数来决定是否释放块，

btrfs_free_extent()

-->如果是extent tree，则在cache_radix中将相应的extent标记为CTREE_ENTENT_PENDING_DEL

-->__free_extent()，使用COW机制从extent tree中删除要被释放的extent item。如果是extent tree，那么就将要被删除的extent插入pinned_radix树中。

-->run_pending()，删除extent tree中cache_radix中标记为CTREE_ENTENT_PENDING_DEL的extent。

对于root tree和extent tree来说，它们的extent都是不使用引用计数机制的，所以它们的引用计数总是为1，当调用btrfs_free_extent()时，总会被删除。

*上面的理解有一点不到位的地方：root tree和extent tree都不使用reference counting，所以它们的ref_cows都被设置为0，而不是上面说的只有extent tree的ref_cows设置为0. 很明显，root tree和extent tree的结点不会被共享，所以不需要使用ref_cows。* 
