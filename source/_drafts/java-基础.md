---
title: java 基础
tags:
---

### 数据类型：基础类型、引用类型

> 引用类型存储在`堆`中

java堆、栈、堆栈的区别？

堆(`stack`)与栈(`heap`)都是`Java`用来在`RAM`中存取数据的地方。与`C++`不同，`Java`自动管理堆和栈，程序员不能直接操作堆或栈。

> 1. 堆(`stack`)存取 优点：可动态分配内存大小，生命周期不必先告诉编译器，Java GC会自动回收不再使用的数据。 缺点：由于运行时动态分配内存，故存取速度相对较慢。

> 2. 栈(`heap`) 

参考：

1. [java堆、栈、堆栈的区别](http://www.cnblogs.com/iliuyuet/p/5603618.html)
2. [java中数据的5种存储位置(堆与栈)](http://blog.csdn.net/ghost_programmer/article/details/40891735)


### 基础类型：boolean、char、byte、short、int、long、float、double

### 为什么要有Boolean、Char等对基础类型的包装类型呢？
比方说

基础类型与引用类型的存储是不一样的