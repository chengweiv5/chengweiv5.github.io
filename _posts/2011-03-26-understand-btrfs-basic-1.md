---
layout: post
title: Btrfs 文件系统基本概念（一）
tags:
  - btrfs
---

执行下面的命令来获得相应的源码

$git clone http://git.kernel.org/pub/scm/linux/kernel/git/mason/btrfs-progs-unstable.git

$cd btrfs-progs-unstable

$git checkout v0.2

$git checkout HEAD~106 -b 106

 

数据结构解析

1. 这里的磁盘使用一个文件来表示，它的结构如下

--------------------------------------------------------------------------

ctree_header   | node/leaf | ... ... | node/leaf | ... ...

--------------------------------------------------------------------------

ctree_header只有一个成员u64 root_block指向树根所在的块，后面即是所有的结点（叶子和非叶子结点）。

2. 结点数据结构

磁盘中的结点有两种：叶子和非叶子，它们有不同的数据结构，都定义在ctree.h中

struct node { struct header header; ... };

struct leaf { struct header header; ... };

node和leaf都有一个header结构，存储了一些关于本结点的元信息，然后node的负载是指针，leaf的负载才是真正的数据。下面是node和leaf的逻辑结构图

----------------------------node--------------------------------------------

header | keys[0] | ... | keys[N-1] | blockptrs[0] | ... | blockptrs[M-1]

------------------------------------------------------------------------------

----------------------------leaf----------------------------------------------

header | item[0] | ... | item[N] | ...free space ... | data[N] | ... | data[0]

-------------------------------------------------------------------------------

这里需要特别说明的是leaf中数据的组织方式，item按照它们的key来排序，而它们的负载data也会相应的移动，所以看起来item和data是一种镜面结构。之所以排序是为了使用二分查找法来搜索。

3. 更进一步分析leaf磁盘结构

leaf中的数据存放格式如上图所示，现在分析item结构。

struct key {

u64 objectid;

u32 flags;

u64 offset;

} __attribute__ ((__packed__));

struct item {

struct key key;

u16 offset;

u16 size;

} __attribute__ ((__packed__));

item中嵌入了key结构，为了对item排序。而item.offset, item.size则表明它的负载data的位置。这往往才是真正的item。

4. 内存数据结构

分析完磁盘数据结构后，现在看看文件系统怎样来管理这些数据，首先介绍概念，然后再分析数据结构。

1). open_ctree() 初始化一个ctree_root对象，通过读取文件（也就是磁盘）

2). insert_ptr()插入一个指针到非叶子节点中。

3). insert_item()插入一个数据到叶子中。

假设我们要存储数据，这可能需要涉及到以上操作。最终将数据保存到正确的leaf中。下面详细分析：

struct tree_buffer {

u64 blocknr; /* 它代表的结点 */

int count; /* 该对象被引用的次数，以免意外释放 */

union {

struct node node;

struct leaf leaf;

};

};

struct ctree_root {

struct tree_buffer *node; /* root_block指向的块 */

int fp; /* 磁盘文件 */

struct radix_tree_root cache_radix; /* 用来组织所有的结点 */

};

open_ctree()打开文件，然后读取文件最前面的ctree_header得到root结点所在的块，然后初始化一个内存对象tree_buffer指向这个块，然后初始化radix tree。这里提到tree_buffer，它是磁盘结点在内存中的表现形式。因为node/leaf都有header，所以一个tree_buffer结构就可以表示它们俩。

当要插入一个数据块时，它是真正的data，这时需要确定将它插入到哪儿，这就需要使用它的关键字初始化一个item，而item又会初始化一个key，这个item的[offset, size]就指向data。相当然的可能在想这里的data的关键字是什么呢？对于用户来说文件是透明管理的，没有什么关键字，但是对于文件系统来说，就要inode存在，所以实际上这里的data是一个文件元数据结构，而不是真正的文件数据。元数据结构就有管理其的方式，例如inode使用inode号来作为关键字。

当插入数据时，有可能叶子结点空间不够，那么还需要分裂，增长存储空间，这里表现为truncate文件的大小。
