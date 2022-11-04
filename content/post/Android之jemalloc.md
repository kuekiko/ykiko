---
title: "Android之jemalloc"
date: 2020-04-05T18:18:59+08:00
draft: false
keywords: ["Android","jemalloc"]
description: "Android之jemalloc"
tags: [
    "android安全",
    "jemalloc原理"
]
categories: ["android安全","jemalloc"]
author: "Vorblock"
---

### 0x00 简单介绍

想调一个CVE、发现对jemalloc 了解太少。重新复习复习jemalloc，做个记录。

jemalloc最初是2005年 Jason Evans开发的新一代内存分配器， 之后没多久被添加到FreeBSD的libc中的默认内存分配器，用来替代原来的phkmalloc。2007年 Firefox Mozilla项目的独立版本也将jemalloc作为主要的分配器。2009年，Facebook 的后端项目也广泛使用jemalloc。2014年，Android 5 开始采用jemalloc作为主要的内存分配器，不过部分Android5/6依然能看到dlmalloc和jemalloc两者并存。

jemalloc的一些特性与设计原则：

- 强大的多核/多线程分配能力.
- 最小化的元数据开销
- 基于每个线程进行缓存，避免了同步问题。
- 避免了连续分配内存的碎片化问题。
- 简洁高效

### 0x01 结构

![结构图](https://my-md-1253484710.file.myqcloud.com/20200403011045.png)

> jemalloc对内存划分按照如下**由高到低**的顺序:

> 1. 内存是由一定数量的arenas进行管理.
> 2. 一个arena被分割成若干chunks, 后者主要负责记录bookkeeping（记录信息）.
> 3. chunk内部又包含着若干runs, 作为分配小块内存的基本单元.
> 4. run由pages组成, 最终被划分成一定数量的regions
> 5. 对于small size的分配请求来说, 这些region就相当于user memory.

##### arenas

对于Android来说：

限制了只使用两个arenas,每个带有一个lock。这意味着，不同线程尝试分配内存时，会循环、平均分配至两个arena，确保两个arena有大致相等的进程数量。只有在相同的arena中分配内存时才需要获取lock。

```c
#/android.bp
android_product_variables = {
    // Only enable the tcache on non-svelte configurations, to save PSS.
    malloc_not_svelte: {
        cflags: [
            "-UANDROID_MAX_ARENAS",
            "-DANDROID_MAX_ARENAS=2",
            "-DJEMALLOC_TCACHE",
            "-DANDROID_TCACHE_NSLOTS_SMALL_MAX=8",
            "-DANDROID_TCACHE_NSLOTS_LARGE=16",
        ],
    },
}
```

用shadow查看arenas

![](https://my-md-1253484710.file.myqcloud.com/20200403015253.png)



##### chunk

一个arena下会有若干个chunk，Android 7之前chunk为256k，之后32位系统改为512k，64位系统改为2MB。

![](https://my-md-1253484710.file.myqcloud.com/20200403012241.png)

每个chunk都有一个chunk head 包含着这个chunk的元数据（metadata）.Android 7之后元数据增加了mapbias与mapbits flags。

chunk是存放run的容器，大小固定相同，操作系统返回的内存被划分到chunk中管理

chunk中的元数据结构，mapbit[0]与mapmisc[0]指向chunk中的第一个run：

![](https://my-md-1253484710.file.myqcloud.com/20200403012851.png)

chunk元数据中mapmisc中的bitmap结构管理着run中的region的分配使用：

![chunk](https://my-md-1253484710.file.myqcloud.com/20200403013355.png)

![](https://my-md-1253484710.file.myqcloud.com/20200403015412.png)

##### run

run是存放连续的大小相同的region的容器，每个chunk中会包含若干个run，而run的metadata会存放在chunk的header当中，这样region里只存放数据本身，不再有内存属性说明。

##### region

region是最小的存储单元，每个run里面的region大小完全相同，也没有元数据，malloc实际返回的是region的地址。

##### bins

jemalloc也用bin来管理内存，共有39个bins。bin的metadata存放于arena的header中，39个bin还会存放当前正在使用的run。所有带有空闲region的run和闲置的chunk信息会被放置在红黑树结构当中，这样寻找空闲内存的复杂度可以控制在o(log(n))。

##### tcache

为了优化多线程性能，jemalloc还采用了LIFO结构的tcache，存放近期被释放的region，每个线程的每个bin都对应一个tcache，存放在tcache中的内存并不会设置free标记位，并且由于tache附着于线程本身，使得大部分情况下从tcache分配内存时完全无需lock。

当jemalloc新分配一块内存是发现tcache为空，会触发prefill事件，此时jemalloc会将单前的arena上lock,并从当前run中取出一定数量的region存入tcache，保证tcache不为空。

当tcache满了（small bin是8，larger bin是20）的时候，会触发flush 事件，会释放部分region，并且才会被标记为已释放。这时这些region才能被其他线程自由分配。

此外，jemalloc也实现是GC机制。会有一个计数器统计申请和释放，达到阈值之后会触发特别的事件，目标bin里的tcache的四分之三的region会被释放掉。下次GC时会轮到下一个bin。这是可以从tcache中删除region并使其恢复常规可用性的另一种方法。

##### 分配流程

![](https://my-md-1253484710.file.myqcloud.com/20200403022314.png)

> - 计算申请内存大小
> - 从当前线程的tcache中找到合适的bin
> - 如果tcache为空，就从当前的run中prefill一些region进来
> - 如果当前run耗尽，就从低地址开始找到第一个非空run
> - 如果现有run里没有足够的内存就分配一个新run
> - 如果chunk里没有空间了就分配一个新chunk，同时分配新run并prefill一些region到tcache

### 0x02 shadow

使用shadow查看Android中的内存布局，简单学习下shadow的使用

#### 查看arenas

![](https://my-md-1253484710.file.myqcloud.com/20200403022557.png)

可以看到一共两个arenas，每个arena有36个bin，一共2个chunk。

#### 查看chunks

![](https://my-md-1253484710.file.myqcloud.com/20200403023042.png)

单个chunk，查看chunk中的run

![](https://my-md-1253484710.file.myqcloud.com/20200403023253.png)

查看runs，会列出单前所有的run的详情 run_siez = region_size*no_regions

![](https://my-md-1253484710.file.myqcloud.com/20200403023401.png)

只显示单前运行中的run 

是否是allocated状态是根据arena_chunk_map_bits_s  对应 bits的 第 [0] bit 来确定 这里jemalloc5 和 jemalloc4 3不一样。

![](https://my-md-1253484710.file.myqcloud.com/20200403023454.png)

查看单个run的详情：

![image-20200403023549875](D:\Project\BLOG\blog\content\post\img\image-20200403023549875.png)

run的布局如下：

![](https://my-md-1253484710.file.myqcloud.com/20200405030548.png)

源代码arena.h中有很多关于bits之类的注释。能够帮助理解。

查看bins:

``` c
struct arena_bin_s {
    malloc_mutex_t         lock;    
    arena_run_t            *runcur;
    arena_run_heap_t       runs; //4之前版本为arena_run_tree_t 类型
    malloc_bin_stats_t     stats; //统计信息
}
```

**runcur:** 当前可用于分配的run, 一般情况下指向地址最低的non-full run, 同一时间一个bin只有一个current run用于分配.

![](https://my-md-1253484710.file.myqcloud.com/20200403023802.png)

看别人的文章说是除去0号bin以外没4个bin为一组，组内size差一样，但是在这里可以看到每8个为一组，01-8号bin的size差值都为0x10,算是第一组，那第二组就为9-12号，只有4个bin size差值为0x20,但是有的为空，算第二组。没两组之间的差值2倍。以此类推，后面每4个为一组。

划分为{0}、{1-8}、{9-12}、{13-16}····· 可能不同版本会有区别。

查看regions [换了一个进程]

![](https://my-md-1253484710.file.myqcloud.com/20200405031010.png)

大小都是0x8

按大小查找：第4个：

![](https://my-md-1253484710.file.myqcloud.com/20200405031343.png)

tchche查看：

tcache的定义：

```c
struct tcache_bin_info_s {
	unsigned	ncached_max;	/* Upper limit on ncached. */
};

struct tcache_bin_s {
	tcache_bin_stats_t tstats;
	int		low_water;	/* Min # cached since last GC. */
	unsigned	lg_fill_div;	/* Fill (ncached_max >> lg_fill_div). */
	unsigned	ncached;	/* # of cached objects. */
	/*
	 * To make use of adjacent cacheline prefetch, the items in the avail
	 * stack goes to higher address for newer allocations.  avail points
	 * just above the available space, which means that
	 * avail[-ncached, ... -1] are available items and the lowest item will
	 * be allocated first.
	 */
	void		**avail;	/* Stack of available objects. */
};
struct tcache_s {
	ql_elm(tcache_t) link;		/* Used for aggregating stats. */
	uint64_t	prof_accumbytes;/* Cleared after arena_prof_accum(). */
	ticker_t	gc_ticker;	/* Drives incremental GC. */
	szind_t		next_gc_bin;	/* Next bin to GC. */
	tcache_bin_t	tbins[1];	/* Dynamically sized. */
	/*
	 * The pointer stacks associated with tbins follow as a contiguous
	 * array.  During tcache initialization, the avail pointer in each
	 * element of tbins is initialized to point to the proper offset within
	 * this array.
	 */
};
struct tcaches_s {
	union {
		tcache_t	*tcache;
		tcaches_t	*next;
	};
};
```

![](https://my-md-1253484710.file.myqcloud.com/20200405034722.png)

![](https://my-md-1253484710.file.myqcloud.com/20200405034659.png)

![](https://my-md-1253484710.file.myqcloud.com/20200405190426.png)

### 0x03 利用

 ##### 堆溢出

一般先利用gadget 绕过ASLR，再利用gadget拿到代码执行的权限，只要能执行代码就能逃出sandboxing或者摆脱selinux。

- Small region overflow

![](https://my-md-1253484710.file.myqcloud.com/20200405190147.png)

- Run overflow

![](https://my-md-1253484710.file.myqcloud.com/20200405190224.png)

- Chunk overflow

![](https://my-md-1253484710.file.myqcloud.com/20200405190305.png)







### 总结

后面还是得使用shadow工具具体调试CVE加深理解。

jemalloc新版与旧版有挺多区别，之后想要深入了解jemalloc的细节以及一些实现还是得看看源码。

#### 参考

- https://github.com/jemalloc/jemalloc
- https://blog.csdn.net/txx_683/article/details/53468211
- https://blog.nsogroup.com/a-tale-of-two-mallocs-on-android-libc-allocators-part-2-jemalloc/
- https://www.anquanke.com/post/id/149132#h3-5
- https://www.anquanke.com/post/id/85982
- dlmalloc 的一个tools: [shade](https://github.com/s1341/shade) 