---
title: redis面试题汇总
date: 2020-03-13 20:49:29
categories: 求职
---
## 常见应用
[参考连接](https://zhuanlan.zhihu.com/p/29665317)

这一题要结合着Redis特性来回答，首先得明确Redis有哪些特性。

> 应用1：读写性能优异可以当作缓存

缓存是Redis最常见的应用场景。

这里面试官可能会问，Redis和memcached比较以及如何保证缓存一致。
<!--more-->
缓存可能会问到Redis缓存实现方式,主要有两种

+ 方案1：读取前会先查询Redis，如果没有命中数据则会去查询数据库比如Mysql，然后拉取数据进Redis

可能会存在缓存击穿以及实时性差的问题，那么如何避免缓存击穿呢？

+ 方案2：插入数据时同时插入Redis，这种实时性强

开发时不便于于统一处理

总结起来就是方案1适合用于对数据实时性要求不高的场合，方案2适合用于数据量不大的场合，如字典表

> 丰富的数据类型

+ String：适合最简单Key-value存储，类似于Memcached存储结构，可以存短信验证码、配置信息等

+ hash：一般key为ID或其他唯一标识，value为详情等

+ list：list适合存储有序且数据相对固定，如省市区表、字典表等 因为list有序，可以根据输入时间排序 如 最新的*** 消息队列等

+ set：可以存放一个人的好友，set的交集、并集、差集操作，例如查找共同好友

+ Sorted Set：是set的增强版本，增加了Score用于排序，可以实现类似Top10而不依赖插入顺序

> 自动过期提高开发效率

如验证码等

> 单线程可以作为分布式锁 setnx

> 分布式和持久化有效应对高并发


## 底层数据结构

[参考文章](https://www.cnblogs.com/jaycekon/p/6227442.html)

+ SDS(Simple Dynamic String)

+ 双端链表
  
```c
typedef struct listNode{
      struct listNode *prev;
      struct listNode * next;
      void * value;  
}
```
可以直接操作`list`来操作链表更方便
```c
typedef struct list{
    //表头节点
    listNode  * head;
    //表尾节点
    listNode  * tail;
    //链表长度
    unsigned long len;
    //节点值复制函数
    void *(*dup) (void *ptr);
    //节点值释放函数
    void (*free) (void *ptr);
    //节点值对比函数
    int (*match)(void *ptr, void *key);
}
```
  
头节点的prev和尾节点的next都指向null

+ 字典 由hashtable实现
```c
typedef struct dictht {
   //哈希表数组
   dictEntry **table;
   //哈希表大小
   unsigned long size;

   //哈希表大小掩码，用于计算索引值
   unsigned long sizemask;
   //该哈希表已有节点的数量
   unsigned long used;
}
```

**SDS、字典、跳跃表、链表、压缩列表**

+ 跳跃表

[Redis内部数据结构详解](https://mp.weixin.qq.com/s?__biz=MzA4NTg1MjM0Mg==&mid=509777776&idx=1&sn=e56f24bdf2de7e25515fe9f25ef57557&mpshare=1&scene=1&srcid=1010HdkIxon3icsWNmTyecI6#rd)

[skiplist](https://juejin.im/post/57fa935b0e3dd90057c50fbc)

`sorted list`底层用到skiplist，那么为什么不用平衡树呢？

内存占用方面skiplist每个节点平均1.33个指针，而平衡树是2

范围查找实现简单

+ [压缩列表](https://mp.weixin.qq.com/s?__biz=MzA4NTg1MjM0Mg==&mid=2657261265&idx=1&sn=e105c4b86a5640c5fc8212cd824f750b&scene=21#wechat_redirect)
  
因为 ziplist 节约内存的性质， 哈希键、列表键和有序集合键初始化的底层实现皆采用 ziplist


## RDB和AOF两种不同方式的比较以及优缺点

> 原理

> 性能

> 稳定性

## 跳表和红黑树之间的比较

> 插入效率

> 实现方式

> 内存消耗

> 特殊条件查询

## SDS和原始字符串比较

[参考文章](https://www.cnblogs.com/jaycekon/p/6227442.html)

+ 查询字符串长度
  
  SDS的结构体如下
```c
/*  
 * 保存字符串对象的结构  
 */  
struct sdshdr {  
      
    // buf 中已占用空间的长度  
    int len;  
  
    // buf 中剩余可用空间的长度  
    int free;  
  
    // 数据空间  
    char buf[];  
};  
```

+ 杜绝内存溢出

+ 减少字符串修改带来的内存重分配次数

+ 惰性空间释放：避免缩短字符串带来的内存重分配，也为将来扩展字符串带来优化

+ 二进制安全

不像c字符串那样通过空字符判断字符串结束，而是通过len属性

## 字典和HashMap的比较

***扩容方式和扩容大小**

## [单线程的Redis为何快](https://www.iminho.me/wiki/blog-25.html)

[一定要看](https://www.iminho.me/wiki/blog-25.html)

## redis同步机制

+ 全量复制
+ 增量复制

[看](https://www.jianshu.com/p/41254dc5cb38)

## 给了一个场景，结合redis设计一个天气数据展示接口，前端传给后台一个城市名参数，后台返回相应的天气数据

