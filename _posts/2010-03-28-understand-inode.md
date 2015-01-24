---
layout: post
title: 理解 inode 中的 i_dentry
tags:
  - kernel
---

#### 索引节点对象struct inode<linux/fs.h>
索引节点对象包含了内核在操作文件或目录时需要的全部元信息（meta data），注意，文件名不包含在inode中。对于Unix/Linux系统来说，这些信息可以从磁盘索引节点直接读入，而对于没有磁盘索引节点的文件系统，如FAT, NTFS；那么在内存中也必须现场组建索引节点对象。

#### 目录项对象struct dentry<linux/dcache.h>
路径中的每个组成部分都由一个索引节点对象表示，例如：/home/user/.vimrc，上面这个路径包含了4个目录项对象：/, home, user, .vimrc；需要注意的是：文件也是目录项对象。为了方便查找操作，VFS引入了目录项概念。每个dentry代表路径中的一个特定部分。目录项也可以包括安装点，VFS在执行目录操作时——如果需要的话——会现场创建目录项对象。

不同于超级块和inode，目录项对象没有对应的磁盘数据结构，VFS根据字符串形式的路径名现场创建它；所以也不需要回写目录项对象。对于目录项的理解，有一点曾经迷惑过我。如下：

{% highlight c %}
/* linux/fs.h */  
struct inode {  
    ...  
    struct list_head i_dentry;  
    ...  
};  
/* linux/dcache.h */  
struct dentry {  
    ...  
    struct inode *d_inode;  
    ...  
};  
{% endhighlight %} 

结论：在inode结构中有一个双向链表struct list_head i_dentry结构，这个链表链接“被使用的”目录项，当然，这些目录项的d_inode指针都指向同一个inode结构。这代表什么呢？表示一个索引节点可以对应多个目录项对象。

分析：显然，要让一个索引节点表示多个目录项对象，肯定会使用文件链接，文件链接有两种类型，硬链接和符号链接（软链接）。使用ls -i可以查看文件的inode。执行一些简单操作如下：

{% highlight bash %}
$mkdir test_dir
{% endhighlight %}

然后执行

{% highlight bash %}
$ln $PWD/test_dir $PWD/test_dir_hardlink
{% endhighlight %}

显然，上面的命令会失败，因为硬链接不能指向目录，否则在文件系统中会形成环，另外，硬链接还不能跨文件系统，为什么呢？看了下面的命令就知道。

{% highlight bash %}
$touch test_file 建立一个文件 
$ln $PWD/test_file $PWD/test_file_hardlink 建立文件的硬链接
$ln -s $PWD/test_file $PWD/test_file_symlink 建立文件的符号链接
$ls -i $PWD 查看当前目录下文件的inode 
{% endhighlight %}

从最后一条命令可以看出，符号链接有不同的inode号，而硬链接文件test_file_hardlink和文件test_file的inode号一样，表示它们执行同一个inode结构。由于inode是对于文件系统来说的，所以硬链接不能跨越文件系统，不同文件系统中的相同inode并不是同一个inode。而符号链接没有硬链接的如上两个限制。

inode结构中的i_dentry链表结构，把属于同一个inode的被使用的（dentry结构中的d_count大于0）目录项连接起来。显然，这里的目录项肯定是使用硬链接的方式来表示的，但是硬链接不能指向目录！这里是最容迷惑的地方：其实，不仅是目录才是目录项，文件也是目录项，看看前面对目录项的定义中的举例就知道了。

另外，在inode结构中嵌入的双向链表成员struct list_head i_dentry，并不是用来链接inode结构的，而是链接dentry结构的，所以这里需要理解Linux kernel中的struct list_head结构，它是一种嵌入式的链表结构，和数据无关，只有嵌入了struct list_head的结构都可以链接在一起，内核中定义了操作这种链表的宏及函数。详见深入理解Linux内核中的链表。

在结构dentry中也嵌入了struct list_head d_lru成员，所以就可以通过它来链接了。
