---
title: java 基础
categories:
- java
tags:
---

### 数据类型：基础类型、引用类型

> 引用类型存储在`堆`中

java堆、栈、堆栈的区别？

堆(`heap`)与栈(`stack`)都是`Java`用来在`RAM`中存取数据的地方。与`C++`不同，`Java`自动管理堆和栈，程序员不能直接操作堆或栈。

> 1. 堆(`heap`)存取 优点：可动态分配内存大小，生命周期不必先告诉编译器，Java GC会自动回收不再使用的数据。 缺点：由于运行时动态分配内存，故存取速度相对较慢。

> 2. 栈(`stack`) 存储 优点：存取速度快，存储效率仅次于寄存器，栈数据可共享。 缺点：栈中数据大小与生存期都必须要在运行前确定的。

> 3. 堆栈，额其实也就是`stack`栈，栈是堆的一个子集（从内存观点考虑），堆栈堆栈，将其理解为堆中的栈。

参考：

1. [java堆、栈、堆栈的区别](http://www.cnblogs.com/iliuyuet/p/5603618.html)
2. [Java中Heap与Stack的区别](http://www.cnblogs.com/SunDexu/p/3140790.html)
2. [java中数据的5种存储位置(堆与栈)](http://blog.csdn.net/ghost_programmer/article/details/40891735)



### 基础类型：boolean、char、byte、short、int、long、float、double
`int a = 3;` 基础类型定义的变量存储的是字面值，不是类的实力，即不是类的引用，这里并没有类的存在。这里的`a`是指向`int`类型的引用，指向`3`这个字面值。


### 为什么要有Boolean、Char等对基础类型的包装类型呢？
[Java基本类型和包装类型解析](http://www.cnblogs.com/xltcjylove/p/3584386.html)

基础类型与引用类型的存储是不一样的