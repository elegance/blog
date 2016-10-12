---
title: java多线程编程核心技术
categories:
- java
tags:
- 多线程
---

## 一、Java多线程技能
Skills:
. 线程的启动
. 如何使线程暂停
. 如何是线程停止
. 线程的优先级
. 线程安全相关的问题

### 1.1 进程和多线程的概念及线程的优点
进程是操作系统结构的基础，它是系统进行资源分配和调度的一个独立单位。可以将windows任务管理器的`.exe`理解成一个进程。
线程：线程就是进程中独立的子任务，比如QQ.exe运行时，有视频的线程、下载文件线程、传输数据线程等。
优点：在多任务操作系统中效率大大提升。

### 1.2 使用多线程
一个进程中至少会有一个线程在运行，比如常见的`main`函数:
```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getName());
}
```

#### 1.2.1 继承Thread类
实现多线程的主要方式有两种：一种是继承`Thread`类，另一种是实现`Runnable`接口。
前者与后者工作时的性质是一样的，继承`Thread`最大的局限就是不支持多继承.
继承实现的方式就是继承`Thread`类，重写`run`方法，然后创建线程类实例，调用其`start`方法即启动了一个线程
`如果多次调用start()方法，则会出现异常 Exception in thread ..`

#### 1.2.2 实现Runnable接口
实现方式：实现`Runnable`接口并实现`run`方法

#### 1.2.3 实例变量与线程安全
线程类：
```java
public class MyThread extends Thread {

    private int count = 5; //实例变量(非static)
    
    public MyThread() {
        super();
    }

    public MyThread(String name) {
        this.setName(name);
    }
    
    @Override
    public void run() {
        count--;
        System.out.println("由 " + Thread.currentThread().getName() + " 计算， count=" + count);
    }
}
```
运行方式一，不共享数据，运行类Run:
```java
public class Run {
    public static void main(String[] args) {
        MyThread a = new MyThread("A");
        MyThread b = new MyThread("B");
        MyThread c = new MyThread("C");
        a.start();
        b.start();
        c.start();
    }
}
```
这里共创建了3个线程，每个线程都有自己的count变量，不存在访问同一个实例变量的情况。

运行方式二，共享数据，运行了ShareRun:
```java
public class ShareRun {
    
    public static void main(String[] args) {
        MyThread threadRun = new MyThread(); // 使用同一个Runnable实现类的实例
        
        Thread a = new Thread(threadRun, "A"); // 再通过Thread包装
        Thread b = new Thread(threadRun, "B");
        Thread c = new Thread(threadRun, "C");
        Thread d = new Thread(threadRun, "D");
        Thread e = new Thread(threadRun, "E");
        
        a.start();
        b.start();
        c.start();
        d.start();
        e.start();
    }
}
```
jvm的i--分3个步骤：
> 1. 取i值
> 2. 计算i-1
> 3. 赋值i

此时多个线程访问`threadRun`实例的变量出现非线程安全问题，这就是典型的线程问题了。这时通过在`run`方法前添加`synchronized`可以解决这种问题。
`synchronized`可以在任意对象及方法上加锁。
多个线程执行`run`时，以排队的方式执行，先判断`run`方法有没有上锁，如果有上锁说明其他线程正在执行，等待其他线程执行。线程尝试拿锁，如果没有
拿所锁，这个线程就会不断的尝试拿这把锁，直到拿到为止。

#### 1.2.4 注意count--与System.out.println()异常
将上面代码里面`count--` 放置到`System.out.println()`中，如下：
```java
System.out.println("由 " + Thread.currentThread().getName() + " 计算， count=" + (count--));
```
我们注意下在jdk的源码中`System.out.println()`这个方法内部是有`synchronized`的，说明这个方法是线程安全。
但是上面的代码也会发生非线程安全问题，因为`count--`这个计算是在`println`方法前执行的，这个有点类似`scala`的求值策略的`Call by Value`，
先会将形参计算出来 ，再进入方法中。

### 1.3 currentThread()方法
`currentThread()`方法可以返回代的段正被哪个线程所执行，可以在线程类的run方法国打印执行线程的名称，然后在外部分别使用线程的`start`和`run`方法执行对比查看。
```java
System.out.println(Thread.currentThread().getName());
```

### 1.4 isAlive()方法
`isAlive()` 方法是用来判断线程是否属于活动状态，活动状态即`线程已经启动且位尚未终止`

### 1.5 sleep()方法
`sleep()`方法是用来让线程暂停指定的时间。

### 1.6 getId()方法
`getId()`方法用来获取线程的唯一标识。

### 1.7 停止线程
停止线程是在多线程开发时重要的技术点，它不想循环中的break简单粗暴，需要一些技巧性的处理，处理好一些身后事(关闭资源连接、事务)才停止线程。
`Thread.stop()`可以停止一个线程，但不建议使用它，这个方法是不安全的。
大多数停止一个线程的操作使用的是`Thread.interrupt()`，尽管方法名称是终止、停止的意思，但这个方法不会终止一个正在运行的线程，需要加入一些判断才可以完成线程的停止。
3中终止正在运行的线程的方法：
1. 使用退出标志，使程序正常退出，也就是当run方法完成后线程终止
2. 使用stop方法强行终止，但是不推荐使用，stop、suspend、resume都是作废过期的方法，使用它们可能会产生意料不到的结果。
3. 使用interrupt方法中断线程

#### 1.7.1 停止不了的线程
下面的例子演示了`interrupt`停止不了线程的现象：
线程类：
```java
public class MyThread extends Thread {
    
    @Override
    public void run() {
        for (int i = 0; i< 500000; i++) {
            System.out.println("i=" + (i+1));
        }
    }
}
```
运行类：
```java
public class Run {
    public static void main(String[] args) {
        try {
            MyThread  t = new MyThread();
            t.start();
            Thread.sleep(2000);
            t.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
最终控制台还是打印了50万行，没有停止住线程。

#### 1.7.2 判断线程是否是停止状态
我们来看下如何判断线程状态不是停止的。Thread提供了两种方法：
1. this.interrupted: 测试当前线程是否已经中断，执行后将状态标识清除为false的功能。
2. this.isInterrupted: 测试当前线程是否已经中断，但不清除状态标志。

#### 1.7.3 能停止的线程——异常法
线程类：
```java
public class MyThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 500000; i++) {
            if (Thread.interrupted()) { //每次获取标识判断
                System.out.println("已经有了停止标识，我要退出了！");
                break; //手动break
            }
            System.out.println("i=" + (i + 1));
        }
        System.out.println("for 循环后面的代码被输出，线程其实未被停止。");
    }
}
```
上面的线程其实未真正停止，下面在接着改进：
```java
public class MyThread extends Thread {

    @Override
    public void run() {
        try {
            for (int i = 0; i < 500000; i++) {
                if (Thread.interrupted()) { //每次获取标识判断
                    System.out.println("已经有了停止标识，我要退出了！");
                    throw new InterruptedException(); // 改为 抛出异常的方式
                }
                System.out.println("i=" + (i + 1));
            }
            System.out.println("for 循环后面的代码被输出，线程其实未被停止。--不会再输出");
        } catch (InterruptedException e) {
            System.out.println("进入线程类的catch，线程终止");
            e.printStackTrace();
        } 
    }
}
```