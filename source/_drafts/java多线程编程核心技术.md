---
title: java多线程编程核心技术
categories:
- java
tags:
- 多线程
---
多线程双刃剑：充分利用多核，复杂度提升，操作共享资源处理不好时会带来线程安全问题。
WEB编程普遍缺乏对多线程的理解：web容器实现，屏蔽了复杂的编程细节，多线程处理，（自己实现一款简单的web容器）

## 一、Java多线程技能
Skills:
* 线程的启动
* 如何使线程暂停
* 如何是线程停止
* 线程的优先级
* 线程安全相关的问题

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
`如果多次调用start()方法，则会出现异常 IllegalThreadStateException ..`

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
`currentThread()`方法可以返回代的段正被哪个线程所执行，可以在线程类的run方法打印执行线程的名称，然后在外部分别使用线程的`start`和`run`方法执行对比查看。
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
3种终止正在运行的线程的方法：
1. 使用退出标志，使程序正常退出，也就是当run方法完成后线程终止
2. 使用stop方法强行终止，但是不推荐使用，stop、suspend、resume都是作废过期的方法，使用它们可能会产生意料不到的结果。（强制性停止可能使一些清理工作得不到完成。）
3. 使用interrupt方法，程序判断isInterrupted，使用throw/return来中断线程，推荐“抛异常”方式

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

可以参考: [interrupt、interrupted 、isInterrupted 区别](http://blog.csdn.net/z69183787/article/details/25076033)

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

### 1.8 暂停、恢复线程
使用`suspend()`方法暂停线程，使用`resume()`方法恢复线程执行。

#### 1.8.1 测试
演示功能效果，起到暂停、恢复效果，测试类：[SuspendResumeTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SuspendResumeTest.java)

#### 1.8.2 suspend与resume方法的缺点——独占
线程类内部锁，独占，此类的其他线程实例也被阻塞，测试类：[SuspendResumeDealLock.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SuspendResumeDealLock.java)

公共锁同步对象被独占，造成主线程阻塞，例如`println`方法，测试类：[SuspendResumeLockStop.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SuspendResumeLockStop.java) 
注释掉除线程内部的`println`方法后面的代码即恢复执行。

#### 1.8.3 suspend与resume方法的缺点——不同步
使用`suspend`与`resume`方法时容易出现因为线程的暂停而导致数据不同步的情况。

值不同步的情况，测试类：[SuspendResumeNoSameValue.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SuspendResumeNoSameValue.java) 


### 1.9 yiled 方法
`yield()`方法的作用是放弃当前`CPU`资源，放弃时间不确定，有可能刚放弃，马上又获得了CPU时间片。

去除注释前后对比测试下吧，测试类：[YieldTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/YieldTest.java) 

### 1.10 线程优先级
线程可以划分优先级，优先级高的线程得到的`CPU`资源角度，也就是CPU优先执行优先级较高的线程对象中的任务。`java`中优先级设置中定义的几个常量：`MIN_PRIORITY=1`、`NORM_PRIORITY=5`、`MAX_PRIORITY=10`，设置小于1或者大于10将会抛出`IllegalArgumentException`

#### 1.10.1 优先级的继承性
`java`中，线程的优先级具有有继承性，比如`A线程`~~启动~~**创建了**了`B线程`，则`B线程`的优先级与`A线程`是一样的。

继承特性，上面写道了`创建了`，可以将`创建 t1`调换值至`设置main优先级之前`测试下，测试类：[PriorityInheritanceTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/PriorityInheritanceTest.java) 

#### 1.10.1 优先级具有规则性
高优先级的线程总是大部分先执行完，但不代表高优先级的线程全部执行完。

### 1.11 守护线程
在`Java`进程中有两种线程，一种是用户线程，一种是守护线程。

守护线程的特性有“陪伴”的含义，当进程中不存在非守护线程了，则守护线程自动销毁。典型的守护线程就是垃圾回收线程。

守护进程，测试类: [DaemonTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/DaemonTest.java) , 理解：用户线程 main都结束了，守护线程们已经没有什么可守护的了，就结束了


## 对象及变量的并发访问
Skills:
* `synchronized`监视`Object`的使用
* `synchronized`监视`Class`的使用
* 非线程安全是如何出现的
* 关键字`volatile`的主要作用
* 关键字`volatile`与`synchronized`的区别及使用情况

### 2.1 synchronized同步方法
多线程并发访问共享资源，很可能发生“非线程安全”题，产生的后果就是"脏读"。而“线程安全”就是获得的资源是经过同步处理的，不会出现脏读现象。

#### 2.1.1 方法内的变量为线程安全
方法内的变量不存在线程安全问题，永远都是线程安全的，这是方法内部的变量私有的特性造成的。

#### 2.1.2 实例变量非线程安全
多个线程访问一个实例中变量发生线程安全问题, 测试类: [ThreadSafetyProblem.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ThreadSafetyProblem.java)

#### 2.1.3 多个对象多个锁
**关键字`synchronized`取的锁都是对象锁，而不是把一段代码或方法当作锁**

两个线程访问同一个类的的不同实例的相同同步方法，因为创建了不同的实例，系统将根据实例个数产生锁。测试类: [TwoObjectTwoLock.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/TwoObjectTwoLock.java)

#### 2.1.4 synchronized 方法与锁对象
* `A线程`先持有`object`对象的`Lock`锁，`B线程`可以调用`object`对象中的非`synchronized`类型的方法。
* `A线程`先持有`object`对象的`Lock`锁，`B线程`调用`object`对象中的`synchronized`类型的方法则需等待，也就是同步。

测试类：[SynchronizedMethodLockObject.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SynchronizedMethodLockObject.java)

#### 2.1.5 脏读
脏读出现在不同线程“争抢”实例变量的结果，即`2.1.4`中非同步方法随时可取共享资源就会造成脏读。

#### 2.1.6 synchronized 锁重入
线程进入`synchronized`方法调用本对象的另一个`synchronized`方法时，是永远可以得到锁的。即进入`synchronized`方法后可以无阻塞畅游本实例的所有方法了(当然这里不包括访问其他实例的`synchronized`方法)

#### 2.1.7 出现异常，锁自动释放
当一个线程执行代码出现异常时，其所持有的锁会自动释放。

测试类：[ExceptionAutoReleaseLock.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ExceptionAutoReleaseLock.java)

#### 2.1.8 同步不具有继承性
子类重写父类的`synchronized`方法，如果该方法不添加`synchronized`标识，此方法将不再具有同步的特性。

测试类：[SyncNotExtends.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncNotExtends.java)

### 2.2 synchronized 同步语句块
多个线程访问同一个对象中的`synchronized(this)`同步代码块时，一段时间内只能有一个线程被执行，其他线程需要等待当前线程完成这个代码块的执行。我们来对比下，同步方法与同步块

同步方法，测试类：[SyncMethod.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncMethod.java)

我们分析可以发现`doLongTimeTask`方法里面，只是对`getData1`、`getData2`赋值做了对**共享资源**的操作，之前部分的耗时操作不依赖**共享资源**，这部分代码完全可以`非同步`率先执行，所以修改方法，可以让方法内部达到`一般异步，一半同步`的效果。

同步块，测试类：[SyncBlcok.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncBlock.java)

完成同样的效果，后者花费的时间缩短了50%；

#### 2.2.1 synchronized 方法的弊端
#### 2.2.2 synchronized 同步代码块的使用
#### 2.2.3 用同步方法解决同步方法的弊端
#### 2.2.4 一半异步，一半同步
其实上面提到的几点，上面的例子都已经体现了

同步方法弊端：也就是方法内部的代码，眉毛胡子一把抓，不区分“必须同步”与“可以异步”的代码将其“同步”的方式来执行，这样方法持锁时间更长，耗时更久。

同步块代码的使用：分析代码，区分`必须同步`与`可以非同步`，将同步代码加上`synchronized`。

#### 2.2.5 synchronized 代码块的同步性
使用`synchronized(this)`代码块时，当一个线程访问`object`的一个`synchronized(this)`方法时，其他线程访问对`object`其他所有的`synchronized(this)`方法访问将会阻塞。

#### 2.2.6 synchronized(this) 代码是锁定当前对象的

#### 2.2.7 将任意对象作为对象监视器
`synchronized`方法与`synchronized(this)`都具有：
1. 对其他`synchronized`方法与`synchronized(this)`同步块调用呈阻塞状态
2. 同一时间只有一个线程执行当前`synchronized`方法块或`synchronized(this)`中的代码

`java`中支持对`任意对象`(注意此处是对象，基础的值类型可是不行的哦)作为“对象监视器”来实现同步功能，使用格式 `synchronized(非this)`

测试类：[SyncBlockString.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncBlockString.java)

#### 2.2.8 细化验证3个结论
`synchronized(非this对象x)`即意味着将对象`x`作为`对象监视器`，可以得出以下3个结论：
1. 多个线程同时执行`synchronized(x)`代码块时呈同步效果
2. 其他线程执行`x`对象中的`synchronized`同步方法时呈同步效果
3. 其他线程执行`x`对象中的`synchronized(this)`代码块时呈同步效果 （2、3点是`x`对象中本身还有相关的同步方法）

测试类：[SyncLockObjInsideSyncMethod.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncLockObjInsideSyncMethod.java)

#### 2.2.9 静态同步synchronized方法与synchronized(class)代码块
`synchronized`可以应用在`static`静态方法上，这样表示对当前的`XX.java`文件对应的`Class`类进行持锁，等同于`synchronized(XX.class)`，会从`class`级别全局阻塞`class`锁，但不会阻塞实例的同步方法（非静态同步方法）。

测试类：[SyncStaticMethod.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/SyncStaticMethod.java)

#### 2.2.10 数据类型String的常量池特性
`JVM`具有`String`常量池缓存的功能，所以使用`String`作为监控锁对象不小心时可能会带来一些意外。所以一般`synchronized`代码块不使用`String`,改用其他如`new Object()`。

测试类：[StringConstantTrait.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/StringConstantTrait.java)