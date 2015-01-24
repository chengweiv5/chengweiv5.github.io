---
layout: post
title: 一些来自 Linux Kernel 的 C 代码技巧
tags:
  - c
---

#### 一种定义函数指针的方法

{% highlight c %}
<linux/proc_fs.h>
typedef int (read_proc_t)(char *page, char **start, off_t offset, int count, int *eof, void *data);
read_proc_t *proc_read;
{% endhighlight %}

这种定义方法有一个优势就是，即使你没有看到read_proc_t的定义，也能够清楚的知道proc_read是一个指针，如下面2中的的create_proc_read_entry中的参数中显示的那样。另一种常用方法是：

{% highlight c %}
typedef int (*read_proc_t)(....);定义函数指针。
read_proc_t p;
{% endhighlight %}

但是这种方法没有上面那种可读性好，因为不能一眼就看出p是一个函数指针。

#### 一种定义宏的方法

{% highlight c %}
<linux/proc_fs.h>
static inline struct proc_dir_entry *create_proc_read_entry(const char *name, mode_t mode,
struct proc_dir_entry *base, read_proc_t *read_proc, void *data);
{% endhighlight %}

inline关键字是GNU C扩展的，专门为设计内核而扩展的，它能够将函数编译成宏，如果它是函数，那么由于有static关键字修饰，显然你不能在本文件之外定义及使用它。

#### 枚举变量的数量

{% highlight c %}
<linux/interrupt.h>
enum{
    HI_SOFTIRQ,
    TIMER_SOFTIRQ,
    ...
    ...
    NR_SOFTIRQS,
};
{% endhighlight %}

这个枚举定义了软中断类型，最后一个NR_SOFTIRQS理所当然的表示了枚举类型的最大值。

#### 联合和结构的嵌套

{% highlight c %}
<linux/mm_types.h>
struct page{
    unsigned long flags;
    union {
        atomic_t _mapcount;
        struct {
            u16 inuse;
            u16 objects;
        };
    };
}
{% endhighlight %}

结构page中的成员怎样访问呢？初始化怎样初始化呢？含有匿名的联合和结构。测试以后可以发现最内层的成员还是当成第一层来访问，但是不能这样初始化。如有

{% highlight c %}
struct page p;
p.flags=1;
atomic_set(&p._mapcount,1);
p.inuse=1;
p.objects=1;
{% endhighlight %}

而不能如下这样初始化

{% highlight c %}
struct page p={
    .flags=1,
    .inuse=1,
    .objects=1,
};
{% endhighlight %}

当然atomic_t成员_mapcount不要单独赋值，而是使用相应的原子操作函数。

#### 宏连接符##

{% highlight c %}
<asm-generic/kmap_types.h>
#define D(n)    __KM_FENCE_##n ,
enum km_type{
D(0)        KM_BOUNCE_READ,
/* 扩展后为
 * __KM_FENCE_0,        KM_BOUNCE_READ,
 */
...        ...
};
{% endhighlight %}
