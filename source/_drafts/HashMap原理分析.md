---
title: HashMap原理分析
categories:
    - java
    - 基础
tags:
    - 源码阅读
---
## HashMap
Hash: 散列，将一个任意的长度通过某种算法转换成一个固定的值。

Key,Value

## 源码分析
数据结构：数组 + 链表

hash(key) ===> hash

indexFor(hash) ===> 数组索引 , O(1)

链表遍历：k == key && k.equals(key) ===> 匹配 key ，O(N), N取决于 hash碰撞程度

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

static final int MAXIMUN_CAPACITY = 1 << 30;

static final float DEFAULT_LOAD_FACTOR = 0.75f; // 扩容因子，当容量占用达到 3/4 时触发扩容

```

resize() ==> 两倍增长，transfer， rehash 准确来讲是 reindex 根据 新的 capacity 与 hash ==> h & (capacity-1)

1.8 新，当链表长度大于8时，转换为红黑树，O(N) => O(logN)

ConcurrentHashMap: Segment extends ReentrantLock ，分段锁

整理了一张理解图：

![hashMap原理](http://wx1.sinaimg.cn/large/929194b4gy1fkrz9urmloj21in0i0myt.jpg)