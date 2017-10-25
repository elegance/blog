---
title: '关于那道Integer: 1000 == 1000 返回false，100 == 100 返回 true的题目'
tags:
  - 源码
categories:
  - java
date: 2017-10-25 11:10:56
---

前段时间看到了那篇[为什么1000 == 1000返回为False，而100 == 100会返回为True?](http://mp.weixin.qq.com/s?__biz=MzUxOTMyMzE2Mg==&mid=2247492934&amp;idx=1&amp;sn=81e40f81110d4239bd1bdfe28d51bc09&source=41&ascene=2&devicetype=android-23&version=26051035&nettype=WIFI&abtest_cookie=AgABAAgADAADAJ6GHgAKiB4AJIgeAAAA&wx_header=1)

#### 一、现象：主要代码如下，请先猜测 1、2的结果输出
```
Integer a = 1000, b = 1000; 
System.out.println(a == b);//1
Integer c = 100, d = 100; 
System.out.println(c == d);//2
```

**一般**你会拿到结果：
```
false
true
```
这里我强调了是在**一般情况下**，原文中并没有写到是一般，我们不更改代码，第一个`a == b`结果也可能返回`true`，修改`jvm`参数配置就可以达到这种效果，文章的最后我提供了测试代码。

其实出现不同的原因是在这里：
* `Integer a = 100` ==> 创建了`Integer`对象 `Integer.valueOf(i)`，方法经过判断会是下面两种情况的一种：
    1. `IntegerCache.cache[n]` 
    2. `new Integer(i)`

会有两种情况：一种会新建对象，一种返回一个缓存的对象

`==` 用来比较值的，对象的值是否相等请使用 `obj.equals(o)`，这里两个`Integer`对象使用`==`来比较本身就是一种**“不正当使用”**，对象`==`取的是对象内存地址。

下面我们看下这个`IntegerCache.cache`的缓存区间：
* `cache[]`: `final int low = -128` ， `final int high`: `sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");` or `127`

#### 二、为什么默认是`[-128...127]`呢？
个人认为可能主要有以下两个原因：
* 日常生活经常使用到的数字，比如年龄，成绩，物品个数等等
* “科学边界” 一个字节，即 `2^8`，256个数字，带上符号就是 [-128...127]

#### 三、总结强调下
使用科学的方法来判断相等：
* 值类型相等比较没得选 用 `==`
* 对象相等比较使用`.equals`

另外，如果你的程序比较特殊，有大量的 `< 128` 或 `> 127` 的数字存在，你可以尝试考虑调节下参数`java.lang.Integer.IntegerCache.high`，当然这是一种锱铢必较的手段，会带来一些交付上的问题，就是每个环境可能都要加上此配置，这里只是一种优化上的思路假设。

#### 四、附加-测试
**测试 场景：1 千万的 Integer数组，将每个设值为[0, 1000]的随机整数，对比初始时间、各代空间占比**
总的对比结果如下（前者代表默认，后者代表--XX:AutoBoxCacheMax=1000）：
 1. 申请1千万数组空间时间无差异，都是在 10ms 左右
 2. 遍历设值初始值，前者会隐式的使用 new，耗时 3000ms左右，后者会隐式命中 cache，耗时 300ms, 相差10倍
 3. 各空间使用对比
    * 前者 eden 空间 使用比后者 eden 空间 大出 1倍
    * 前者 old 空间使用了 126M，后者没有使用 old空间

源码请看[这里](https://github.com/elegance/dev-demo/blob/master/java-demo/src/main/java/org/orh/basic/IntegerCacheTest.java)