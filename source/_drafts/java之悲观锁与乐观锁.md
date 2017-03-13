---
title: java之悲观锁与乐观锁
tags:
---
参考：
1. [乐观锁的一种实现方式——CAS](http://www.tuicool.com/articles/yiyy6bI)
2. [Java 理论与实践: 非阻塞算法简介](https://www.ibm.com/developerworks/cn/java/j-jtp04186/)
3. [深入理解Java虚拟机笔记---原子性、可见性、有序性](http://www.tuicool.com/articles/ru6vUvn)
4. [Java 理论与实践: 流行的原子](https://www.ibm.com/developerworks/cn/java/j-jtp11234/index.html)
<hr>

### 锁的由来：共享资源，并发访问没有问题，操作则会带来线程安全问题，所以需要锁的存在。

### 常见误用：static HashMap、static SimpleDateFormat

### 内存模型围绕着并发过程中如何处理原子性、可见性、有序性来建立的
1. 原子性(Atomicity)
2. 可见性(Visibility)
3. 有序性(Ordering)

### 悲观锁
锁解决原子性问题，1.5之前synchronized,悲观锁

线程==>锁（共享资源）

### 悲观锁问题：
竞争～[线程等待挂起]～加锁(独占)～释放，上下文切换调度延时，性能问题

### volatile 更轻量级的同步机制
普通：主堆内存 ==load==> 线程工作内存==save==>主堆内存（有点类似于缓存）

volatile：标志其不稳定、易变，从而不缓存，保证线程间的数据是可见的

### 乐观锁
乐观锁思想：资源一般情况下不会冲突，资源提交更新时才检测冲突。(冲突概率小于20%  适用场景)

### 乐观锁与悲观锁的实现
乐观锁实现：原子类（AtomicXX--- CAS： JNI CPU指令）、数据库(version字段-读取、更新检查)
悲观锁实现：synchronized锁、数据 for updat

### 适用场景
悲观锁认为的场景是：总有人和我抢资源
乐观锁认为的场景是：基本没人和我抢资源