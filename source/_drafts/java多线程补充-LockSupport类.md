---
title: java多线程补充-LockSupport类
categories:
    java
tags:
    多线程
    锁
---

在之前的[java多线程编程核心技术](http://blog.ouronghui.com/2017/04/06/java多线程编程核心技术)文章中，主要是记录了书`《Java多线程编程核心技术》`的一些内容，其中没有介绍到`LockSupport`类，但是这个类在`jdk`源码中也经常会碰到，所以特意拿出来再看一番。

##### 既然已经有了`ReentrantLock`，为什么还需要`LockSupport`呢？
主要区别在于他们面向的对象不同。

我们先简单来回顾下`ReentrantLock`的使用：
```java
    public static void reviewReentrantLock() {
        ReentrantLock lock = new ReentrantLock();

        new Thread(() -> {
            try {
                lock.lock();
                System.out.println("thread-A doXX");
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock(); // 记得 finally 中 释放锁
            }
        }, "thread-A").start();

        new Thread(() -> {
            try {
                lock.lock();
                System.out.println("thread-B doXX");
            } finally {
                lock.unlock();
            }
        }, "thread-B").start();
    }
```
* lock 通常需要在finally中释放
* 其它线程取锁将被阻塞

接下来看下`LockSupport`要怎么使用，看了下`LockSupport`的源码有以下特点：
* 构造函数是私有的，说明不可手动实例化
* 其它的方法都是静态的，说明不用实例化可以直接拿来使用

##### 尝试1
```java
    public static void useLockSupport() {
        LockSupport.park(); // 等待许可，如果许可可用将立即返回，否则线程将进入休眠状态
        System.out.println("block.");
    }
```
* 运行此方法会发现，控制台的Terminate一直是红色，"block"也不会输出，说明当前线程在park后就被阻塞了
* **线程许可默认是被占用的**

可以使用`unpark(thread)`先取的许可，再执行`park`,如下：
```java
    public static void useLockSupport2() {
        Thread thread = Thread.currentThread();
        LockSupport.unpark(thread); // 为给定的线程提供许可；如果线程在park上被阻塞，那么它将被解除阻塞。否则它的下一次 park 执行将不会被阻塞
        LockSupport.park(); // 等待许可，因为上一步已经提供了，所以会直接往下执行
        System.out.println("block");  // 像没事人样，正常输出
        LockSupport.park(); // 等待许可，前面的许可已经被使用了，故此线程将会进入休眠，等待许可
        System.out.println("block 2");  // 阻塞，不会被输出
    }
```
* “block”会正常输出。
* “block 2”不会输出，线程如果重复调用`park`，那么线程将会一直阻塞下去，故**LockSupport是不可重入的** ，对比而言`ReentrantLock`是可重入的，一个线程可以多次获取同一把锁，`lock.getHoldCount()`方法会得到当前线程持有该锁的个数，也就是`lock()`方法的次数。总的来说就是：**`lock.lock()`可以重复调用(线程内执行相应的`lock.unlock()`)，而`LockSupport.park()`则是单次不重复的，只可等待其他线程`LockSupport.unpark(t)`**

##### 线程等待许可时，被打断时会怎样？
下面我们看下线程对应中断会怎样响应：
```java
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            long start = System.currentTimeMillis();
            long end = 0;
            int count = 0;

            while (end - start <= 1000) {
                count++;
                end = System.currentTimeMillis();
            }
            System.out.printf("after 1 second: acount=%s\n", count);
            
            LockSupport.park(); // 等待许可
            System.out.println("thread-child over." + Thread.currentThread().isInterrupted());
        }, "thread-child");
        
        t.start();
        Thread.sleep(2000);
        
        t.interrupt(); // 中断线程
        System.out.println("main over!");
    }
```
控制台输出：
```
after 1 second: acount=85732003
main over!
thread-child over.true
```
* 并没有抛出异常，程序照常往下执行

#### 跳回前面的问题小结下 ReentrantLock 与 LockSupport 
`ReentrantLock` 关注线程内部取锁`lock()`，`unlock()`的问题，都是**线程内部代码**在掌控锁，自己关注方法内的业务逻辑。
`LockSupport` 更倾向线程间的协作，一个线程“LockSupport.park()”等待许可，另外一个线程来“唤醒”等待的线程。

`LockSupport`像是站在线程间的指挥家，可以指定唤醒哪个线程(`LockSupport.unpark(thread)`)、什么时候唤醒等。

更准确的理解可以去查看`LockSupport`的源码注释。

---
主要参考：
* [LockSupport的park和unpark的基本使用,以及对线程中断的响应性](http://blog.csdn.net/aitangyong/article/details/38373137)
* [Java中Lock和LockSupport的区别到底是什么？](https://www.zhihu.com/question/26471972/answer/74773092)