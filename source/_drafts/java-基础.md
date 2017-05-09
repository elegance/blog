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

### 有关hasCode、equals、== ，重写对象的 equals方法
`==` 一般用于原始类型相等判断， 非原始类型即`Object`, 默认情况下是比较两个对象的地址，`equals`方法内部直接采用的`==`比较。

`hashCode`是Object类的方法，返回一个离散的int型整数。集合操作中使用，提高查询速度，如HasSet、HashMap、HashTable等。

[OverrideEquals.java](https://github.com/elegance/dev-demo/blob/master/java-demo/basic/OverrideEquals.java)

### 一个中文在java中占几个字节？
这个题目其实是不完整的，就像有数字“42”，你说它占几个字节，那你得用byte、short、int还是long来存它；。

那么我们这里要确定这个中文使用什么类型存储的，比如是`char c = '中';` 还是`String s = "中";` ，一个中文，即字符，**同一字符在不同编码下可能占用不同的字符**，脱离具体的编码讨论某个字符占几个字节没有意义。

| 编码方式 | 'a' | '中' | '𩄀' |
|:-----|:-----|:-----|:-----|
| ASCII | 1 | ? | ? |
| UTF-16 | 2 | 2 | 4 |
| UTF-8 | 1 | 2 | 4 |
| GBK | 2 | 2 | ? |

1. java 内部采用编码`UTF-16BE`, 但是**Java语言规范规定，Java的char类型是UTF-16的code unit，也就是一定是16位（2字节）；**(https://www.zhihu.com/question/27562173)，也就是只能表示1个单元可以表示的字符，**增补字符**占两个单元的字符无法表示。

2. `UTF-16`编码的特点：以2个字节为编码单元，字节序分为UTF-16BE、UTF-16LE(大端法、小端法)；大部分字符使用2个字节，少部分4个字节；


如果"字符"是指`java`中的`char`，那他是16位，2字节。

**不同字符在相同编码下也可能占不同字节**，这种情况发生在**变长编码**上。

固长编码：ASCII、GB2312/GBK、UTF-32，固长编码的问题就是 要么不能支持一部分字符，要么就是浪费空间

变长编码：UTF-8、UTF-16、GB18030(不常见)