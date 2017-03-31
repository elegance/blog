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

#### 2.2.11 同步sychronized方法无限等待与解决
同步方法容易造成死循环，让其他线程得不到运行机会。

测试类：[TwoStop.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/TwoStop.java)

这个其实还得与具体的业务分析，不同的同步方法，是否涉及同一个资源访问的读写，如果不涉及则可以使用不同的"监控锁对象"（以上举例的测试类，在真正的业务场景中基本不会这样）

比如按业务不同，定义不同的锁对象，测试类：[TwoStopMultiLockObj.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/TwoStopMultiLockObj.java)

#### 2.2.12 多线程的死锁
`java`线程死锁是一个经典的问题，造成的原因是不同的线程在等待不可能被释放的锁。

这里我自己脑补了一个来帮助理解死锁的例子:

测试类：[DeadLockTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/DeadLockTest.java)

死锁是因为`阿毛`：`先锁坑位，再锁纸`不走寻常人的路，`阿毛他爹`、`阿毛麻麻`是：`先锁纸，再锁坑位`，导致`阿毛全家`都不能正常上厕所了。 如果注释掉`阿毛`，其他人都能正常排队的上厕所，反之注释掉其他人，`阿毛`一个人也玩的转，他们同时来就会发生死锁。

避免死锁：**对多个资源(数据库表、对象)同时加锁时，需要保持一致的加锁顺序**

运行程序后可以看到应用`假死`了，这是我们可以用`jps` 查看下运行的`java`进程的信息，得到`pid`，然后使用 `jstack -l pid`，可以看出`线程`在等待哪个对象的锁，而这把锁现在正被哪个线程锁持有，可以看出这里的死锁就是两个线程互相持有了对方等待的锁。

```shell
$ jps 
    -q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数
    -m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
    -l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
    -v : 详细信息

$ jstack
    -l 
```

#### 2.2.13 内置类与静态内置类
内部类依赖其外部类实例来实例化：`new External().new InnerClass()`， 静态内部类则可以直接：`new InnerStaticClass()`，实例化时不依赖外部类实例

测试类：[ExternalClass.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ExternalClass.java) 、[RunTestExternalClass.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/RunTestExternalClass.java)

```
// 内部类， 把类比喻成一个鸡蛋，内部类是蛋黄，外部类是蛋壳
// 普通内部类(非static) 那相当于生鸡蛋，没有鸡蛋壳（外部类没有实例化），蛋黄也就不复存在 ----- 生鸡蛋： 壳之不存，黄之焉附
// 内部静态类 , 相当于熟鸡蛋，没有鸡蛋壳，蛋黄也可以是完好的（可以实例化）， -----  熟鸡蛋：唇亡齿寒 ，照样可以嚼东西（没有蛋壳，蛋黄可以用来做卤蛋呀...）

// 内部类没有 `public`标识时，只有在同一个包的其他类可访问、实例化
```

#### 2.2.14 内置类与同步：实验1
测试类：[OutClzSyncTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/OutClzSyncTest.java) 、[RunOutClzSyncTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/RunOutClzSyncTest.java)

这里很好理解，由于`method1`与`method2`持有不同的“对象监视器”，所以他们是异步非阻塞，打印结果是乱序的。

#### 2.2.15 内置类与同步：实验2
测试类：[OutClzSyncTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/OutClzSyncTest2.java) 、[RunOutClzSyncTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/RunOutClzSyncTest2.java)

这里用我们上面学到知识就可以理解了，A1、A2 使用了不同锁，非阻塞异步执行，A1、B1争抢一把锁，A1释放锁后B1才可拿锁执行。

#### 2.2.16 锁对象的改变
测试类：[ChangeLockString.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ChangeLockString.java)

从测试类发现：A、B线程锁对象`lock`，就算值发生了改变，他们持有的锁都是“123”，还是起到了同步的效果，C、D线程进一步验证了只要对象不变，即使对象的属性被改变，其运行结果还是同步的。

### 2.3 volatile 关键字
`volatile`关键字的主要作用是使变量在多个线程间可见。强制从**公共堆栈**中取得变量值，而不是从**线程私有数据**取得变量的值

测试类：[VolatileCompare.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/VolatileCompare.java)

测试类输出结果可以看出，没有`volatile`标识的变量，`threadA`根本不理会主线程对这个变量的修改，线程会一直运行；而依赖`volatile`修饰变量运行的线程，可以得到主线程的修改，线程得以正常退出。

![-server为了线程效率，从私有堆栈中取值](/images/java-thread/java-thread-memory.jpg)

![volate强制从公共堆栈中取值](/images/java-thread/volatile-force-main-memory.jpg)

#### 2.3.1 关键字volatile与死循环--单线程死循环，下一步的停止标识设置没有机会执行
#### 2.3.2 解决同步死循环--(多线程解决，新启线程来设置停止标识)
#### 2.3.3 解决异步死循环--(-server 不读取主堆栈问题，使用volatile解决，强制读主堆栈内存)

#### 2.3.4 volatile 非原子的特性
虽然`volatile`虽然实现了共享资源在多个线程间的可见性，但它却不具备同步性，那么也就不具备原子性(个人理解：变量本身没有原子性，对变量的操作才有原子性一说)。

测试类：[CounterVolatileUnsafe.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/CounterVolatileUnsafe.java)

`volatile`只是增加了多线程间共享资源的透明度，上面的执行结果有可能出现的是你的期望值`10000`，这只是提高了他出现的几率，这也体现了线程安全问题的难以测试和问题偶发性。

#### 2.3.5 使用原子类进行i++ 操作
从jdk1.5起开始提供了`AtomicXX`的一些原子类，这些类是乐观锁的一种CAS(Compare And Swap)的实现，其利用JNI调用CPU指令实现。

主要提供了：`AtomicInteger`、`AtomicBoolean`、`AtomicLong`供基础数据类型的操作，`AtomicReference<V>`对象数据操作，`AtomicStampedRefrence<V>`来操作对象并解决`ABA`的问题

`AtomicInteger`完成i++, 测试类：[AtomicIntegerTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/AtomicIntegerTest.java)
`AtomicReference<V>`模拟栈，测试类：[AtomicStack.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/statc-atomicVsSync/AtomicStack.java)
`AtomicStampedRefrence<V>`解决ABA问题，测试类：[ABA.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/CAS/ABA.java)

#### 2.3.6 原子类也并不完全安全
这里其实主要说明的是多个原子类方法间是不安全的，单个原子类方法没有问题。

#### 2.3.7 synchronized 代码块有volatile同步的功能
关键字`synchronized`可以使多个线程访问同一个资源具有同步性，而且还具有将线程工作内存中的私有变量与公共内存中的变量同步的功能。

## 线程间通信
Skills:
* 使用`wait/notify` 实现线程间的通信
* 生产者/消费者模式的实现
* 方法`join`的使用
* `ThreadLocal`的使用

### 3.1 等待/通知机制
线程与线程之间不是独立的个体，他们彼此之间可以互相通信和协作。

#### 3.1.1 不使用等待/通知机制实现线程间通信
测试类：[TwoThreadTransData.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/TwoThreadTransData.java)

线程`thread-b`循环中不断检测一个条件，轮循时间小，会造成浪费CPU资源，轮循间隔大时，响应不会实时。 所以要出现了`wait/notify`机制。

#### 3.1.2 什么是等待/通知机制
比如：`厨师`完成一道菜的时间不确定，`服务员`需要将这道菜，送给就餐者。

如果不是“等待/通知”机制：`服务员`不断的询问`厨师`菜完成了没...

有了“等待/通知”机制择时：`服务员`坐等(wait)，`厨师`完成菜品即告诉(notify)`服务员`。

#### 3.1.3 等待/通知机制的实现
**`wait`使线程暂停执行，而`notify`唤醒其他线程继续执行**

`wait()`方法使当前执行代码的线程进行等待，`wait()`方法是`Object`类的方法，该方法将当前线程置入“预执行队列中”，并且在`wait()`所在代码处停止，直到接到通知或被中断。
```
调用`wait()`方法前必须获得该对象实例的锁，即只能再锁对象实例的同步代码内中调用`wait()`方法，否则将抛出`IllegalMonitorStateException`，`wait()`方法后，当前线程释放该对象实例的锁。
```
`notify()`方法也需要或得对象实例的锁，该方法用来通知那些可能等待该对象的对象锁的其他线程，如果有多个线程等待，则有线程规划器随机挑出一个呈`wait`状态的线程，对其发出通知notify。
```
执行`notify()`方法后，当前线程不会马上释放该对象的锁，呈wait状态的锁不会马上获得锁，而是要等执行`notify()`方法所在同步块执行完。
```

简单体现`wait/notify`, 测试类：[TestWaitNotify.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/TestWaitNotify.java)

实现之前提到的`当公共变量==5时退出一个线程`, 测试类：[WaitNotifyWhen5.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/WaitNotifyWhen5.java)

**`运行--就绪--等待**  （用单核cpu的方式去立即：同一时刻只有一个线程被执行，所以有了就绪队列）`:
**每个锁对象都有两个队列，一个是就绪队列(竞争锁)，一个是阻塞队列(wait待唤醒)。就绪队列存储了将要获得锁的线程，阻塞队列存储了被阻塞的线程。一个线程被唤醒后，才会进入就绪队列，等待CPU调度；反之，一个线程被wait后，就会进入阻塞队列，等待下一次被唤醒**

关于`就绪队列`与`阻塞队列`顺序相关
1. 就绪队列，进入方法的顺序与竞争得到锁的顺序，测试类：[CompeteOrder.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/CompeteOrder.java)
2. 阻塞队列，wait触发的顺序与被唤醒的顺序，测试类：[WaitOrder.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/WaitOrder.java)

#### 3.1.4 方法wait()锁释放与notify()方法锁不释放
当方法`wait()`被执行后，锁自动释放，但执行完`notify()`方法，锁是不自动释放的，还有在同步代码块内`sleep()`方法也是不会释放锁的。 这些其实在上面的例子中都已经有体现了。

#### 3.1.5 当interrup方法遇到wait方法
当线程呈`wait`状态时，调用线程对象的`interrupt()`方法会出现`InterruptedException`异常。

#### 3.1.6 只通知一个线程
调用`notify()`方法一次只~~随机~~(wait队列poll出一个)通知一个线程进行唤醒。

测试类：[NotifyOne.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/NotifyOne.java)

#### 3.1.7 唤醒所有线程
调用`notifyAll()`方法将唤醒wait队列中的所有线程。

#### 3.1.8 方法wait(long)的使用
带一个参数`wait(long)`方法的功能是等待指定的时间，如果指定时间内没有被`notify`将自动苏醒。

无人唤醒，自动苏醒，测试类：[WaitHasParamMethod.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/WaitHasParamMethod.java)

#### 3.1.9 通知过早
"服务员"还没过来等待，“厨师”做完菜就发出了通知。也就是 一个线程notify 发生在另外一个线程wait之前。解决这种问题是在调用wait方法前判断，如果先通知了，则wait方法就没必要执行了。

#### 3.1.10 等待wait的条件发生变化
要注意wait所依赖的条件变化，多个线程在wait，有可能条件检验已经过期，测试类：[WaitOld.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/WaitOld.java)

#### 3.1.11 生产者/消费者模式实现
`生产者消费者：互相通知，互相等待`

1. 一生产者与一消费者：操作值，测试类：[ProducerConsumerTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ProducerConsumerTest.java)
2. 多生产者与多消费：操作值-假死，测试类：[ProducerConsumerAllWait.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ProducerConsumerAllWait.java)
3. 多生产者与多消费：操作值，将上面中的`notify`改为`notifyAll`，唤醒所有。
4. 一生产与一消费：操作栈，测试类：[ProducerConsumerStack.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ProducerConsumerStack.java)
5. 一生产与多消费：操作栈，测试类：[ProducerMulConsumerStack.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ProducerMulConsumerStack.java)

#### 3.1.12 通过管道进行线程间通信：字节流
`Java`语言中提供了各种各样的`输入/输出` 流`Stream`，能方便的对数据进行操作，其中`管道流`(`pipeStream`)是一种特殊的流，用于在不同线程间直接传送数据。

字节流通信，测试类：[PipedInputOutput.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/PipedInputOutput.java)

#### 3.1.13 通过管道进行线程间通信：字符流
字符流通信，测试类：[PipedReaderWriter.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/PipedReaderWriter.java)

#### 3.1.14 实战：等待/通知之交叉备份
创建20个线程，其中10个线程将数据备份至A数据库，另外10个线程将数据备份至B数据库，并且备份A数据库和备份B数据库交叉进行。

测试类：[WaitNotifyInsert.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/WaitNotifyInsert.java)

### 3.2 方法join的使用
在很多情况下，主线程创建并启动子线程，如果子线程耗时较久，主线程往往将早于子线程结束。这这时如果主线程想等待子线程完成之后做操作，就可以使用`join()`方法了。

#### 3.2.1 学习方法join前的铺垫
比如我们需要 T1、T2、T3 三个线程按先后顺序执行，没有join方法可以尝试这么做：
```java
T1.start();  T1.sleep(xxx);
T2.start();  T2.sleep(xxx);
T3.start();
```
问题就在于 上面的xxx时间我们无法确定，因为每个线程运行多久我们不能确定， 设值长了，程序会浪费等待时间，设值短了，可能出现顺序不对。

#### 3.2.2 用join方法解决
```java
T1.start();  T1.join();
T2.start();  T2.join();
T3.start();
```
这样就能保证三个线程依次执行了。

另外注意：**如果`T1`内部启动了新的线程，`T1.join()`方法后面的代码不会等待`T1`新启的线程**

#### 3.2.3 方法join与异常
#### 3.2.4 方法join(long)的使用
```java
childThread.join(1000); //执行这个方法所在的线程最多等待 1000ms，
    Thread.sleep(1000); //这个方法会无论如何等待 1000ms
如果子线程执行的时间超过1000ms，那么他们所看起来的效果是一致的。    
```

#### 3.2.5 方法 join(long) 与 sleep(long)的异同
`join(long)`方法内部是使用`wait(long)`实现的，所以`join(long)`方法也具有释放锁的特点， 而`sleep(long)`方法不会释放锁。

相同：
1. 调用`sleep`与`join`方法来达到阻塞当前线程的目的

不同：
1. `sleep(long)`为`Thread类static`方法，`join(long)`为`Thread实例的方法`，故需要注意他们作用于的线程区别
2. 作用于普通的非同步方法中区别就是：`sleep(long)`等待固定时间、`join(long)`最多等待这么久的时间

具体深入对比可以查看: [Thread类join方法中的 wait(0) 能用 sleep(0) 来替代模拟吗](http://blog.ouronghui.com/2017/03/23/Thread%E7%B1%BBjoin%E6%96%B9%E6%B3%95%E4%B8%AD%E7%9A%84%20wait(0)%20%E8%83%BD%E7%94%A8%20sleep(0)%20%E6%9D%A5%E6%9B%BF%E4%BB%A3%E6%A8%A1%E6%8B%9F%E5%90%97/)

### 3.3 类ThreadLocal的使用
类变量的共享可以采用`public static`形式，所有线程都使用同一个`public static`变量。 如果想要实现每个线程都有自己的共享变量呢？ `JDK`提供的类`ThreadLocal`正是为了解决这个问题的。

* 局部变量：方法内，不同享，与实例和线程都无关。
* 全局变量：类内，共享实例变量，在不同的线程、或方法间达到共享
* 全局静态：类内，共享静态变量，不同线程间访问达到共享，`静态`与实例无关。

#### 3.3.1 方法 get() 与 null
#### 3.3.2 验证变量的隔离性
#### 3.3.3 解决 get() 返回 null问题
#### 3.3.4 再次验证线程变量的隔离性
测试类：[VerifyIsolation.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/VerifyIsolation.java)

### 3.4 类 InheritableThreadLocal 的使用
使用`InheritableThreadLocal`可以在子线程取得父线程继承下来的值。

#### 3.4.1 值继承
#### 3.4.2 值继承再修改
测试类：[InheritableThreadLocalTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/InheritableThreadLocalTest.java)

## 4. Lock的使用
Skills:
* `ReentrantLock`类的使用
* `ReentrantReadWriteLock`类的使用

### 4.1 使用 ReentrantLock类
在`java`多线程中，可以使用`synchronized`关键字来实现线程之间同步互斥，但在`jdk1.5`中新增了`ReentrantLock`类也能达到同样的效果，并且在扩展功能上也更加强大，比如有嗅探锁定、多路分支通知等功能，使用上比`synchronized`更加灵活。

#### 4.1.1 使用 ReentrantLock类实现同步
测试类：[ReentrantLockTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ReentrantLockTest.java)

#### 4.1.2 使用 ReentrantLock类实现同步: 测试2
测试类：[ReentrantLockTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ReentrantLockTest2.java)

#### 4.1.3 使用Condition 实现等待/通知：错误用法与解决
关键字 `synchronized`与`wait()`和`notify()/notifyAll()`方法结合可以实现等待/通知模式，类`ReentrantLock`实现同样的功能是借助于`Condition`对象。`Condition`是JDK5中出现的技术，使用它有更好的灵活性，比如实现多路通知的功能，也就是在一个`Lock`对象中可以创建多个`Condition`对象实例，线程对象可以注册在指定的`Condition`中，从而可以有选择性地进行线程通知，在调度线程上更加灵活。

测试类：[UseConditionWaitNotifyError.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/UseConditionWaitNotifyError.java)

#### 4.1.4 正确使用Condition实现等待/通知
测试类：[UseConditionWaitNotifyOK.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/UseConditionWaitNotifyOK.java)

成功的实现了等待/通知模式。
* `Object`类中的`wait()`相当于`Condition`类中的`await()`方法， 线程进入`WAITING`状态。
* `Object`类中的`wait(long timeout)`相当于`Condition`类中的`await(long time, TimeUnit unit)`方法， 线程进入`TIMED_WAITING`状态。
* `Object`类中的`notify()`相当于`Condition`类中的`signal()`方法
* `Object`类中的`notifyAll()`相当于`Condition`类中的`signalAll()`方法

#### 4.1.5 使用Condition实现通知部分线程：错误用法
测试类：[MustUseMoreConditionError.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/MustUseMoreConditionError.java)

两个方法共用一个`Condition`，不能体现区别唤醒，`thread-A`、`thread-B`两个线程启动分别调用了同一个`condition`的`await()`方法，线程都进入了`WAITING`状态，最后主线程同时唤醒的是两个线程。

#### 4.1.6 使用多个Condition实现通知部分线程：正确用法
测试类：[MustUseMoreConditionOK.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/MustUseMoreConditionOK.java)

此时两个方法分别使用了`conditionA`、`conditionB`，主线程调用了`conditoinA.signalAll()`达到了只唤醒`thread-A`的效果，`thread-B`继续`WAITING`

#### 4.1.7 实现生产者/消费者：一对一交替打印
测试类：[ConditionTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ConditionTest.java)

#### 4.1.8 实现生产者/消费者：多对多交替打印
测试类：[ConditionTestManyToMany.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ConditionTestManyToMany.java)

类似`Object`的`notify()`方法，`signal()`方法同样也会造成假死的现象，这是因为`生产者`与`消费者`使用的是同一个`Condition`，`signal()`方法在我们期望通知`消费者`时，可能通知到的是`另一个消费者`，反之消费者发出的通知也是一样的。所以这里也采取了`signalAll()`方法发出信号一并唤醒。

#### 4.1.9 公平锁与非公平锁
公平与非公平锁：锁`Lock`分为`公平锁`和`非公平锁`, 公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的`FIFO`先进先出的顺序。 而非公平锁就是一种获取锁的抢占机制，是随机获得锁的。

测试类：[FairNoFairTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/FairNoFairTest.java)
* 公平锁 ，开始运行与得锁顺序呈有序
* 非公平锁， 开始运行与得锁顺序基本上是乱序的

#### 4.1.10 方法getHoldCount()、getQueueLength()、getWaitQueueLength()
* `int getHoldCount()` 方法是**查询当前线程保持此锁的个数**，也就是调用 `lock()`方法的次数。测试类：[LockMethodHoldCount.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodHoldCount.java)
* `int getQueueLength()` 方法是返回**等待获取此锁的线程估计数** 测试类：[LockMethodQueueLength.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodQueueLength.java)
* `int getWaitQueueLength(Condition condition)` 方法是返回**等待此锁相关条件Condition的线程估计数** 测试类：[LockMethodWaitQueueLength.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodWaitQueueLength.java)

这里的体现其实与之前`synchronized`内部锁达一致：锁的两个队列，getQueueLength() 取的是针对此锁 在`BLOCKED` 就绪阻塞线程，`getWaitQueueLength(Condition condition)` 则是`WAITING`睡眠等待唤醒的线程

#### 4.1.11 方法 hasQueuedThread()、hasQueuedThreads() 和 hasWaiters()的测试
* `boolean hasQueuedThread(Thread thread)` 查询指定的线程是否等待获取此锁。
* `boolean hasQueuedThreads()` 查询是否有线程正在等待获取此锁。

测试类：[LockMethodHasQueued.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodHasQueued.java)

* `boolean hasWaiters(Condition condition)` 查询是否有线程正在等待此锁有关的`condition`条件,测试类：[LockMethodHasWaiters.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodHasWaiters.java)

#### 4.1.12 方法 isFair()、isHeldByCurrentThread() 和 isLocked() 的测试
* `boolean isFair()` 判断锁是不是公平锁，默认情况下`ReentrantLock`是`非公平锁`
* `boolean isHeldByCurrentThread()` 查询当前线程是否保持此锁定。
* `boolean isLocked()` 查询此锁是否有线程保持锁定。

测试类：[LockMethodIsHeldByCurrentThread.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodIsHeldByCurrentThread.java)

#### 4.1.13 方法 lockInterruptibly()、tryLock() 和 tryLock(long timeout, TiemUnit unit) 的测试
* `void lockInterruptibly()` ：取锁，如果当前线程未被中断，则获取锁定，如果已经被终端则出现异常。测试类：[LockMethodInterruptiblyTest.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodInterruptiblyTest.java)
* `boolean tryLock()` : 取锁，尝试取锁，如未取得则返回false。
* `boolean tryLock(long timeout, TimeUnit unit)`：取锁，指定时间内如未取得锁，取锁失败返回false

测试类：[LockMethodTryLock.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodTryLock.java)

#### 4.1.14 方法 awaitUninterruptibly() 的使用
测试类：[LockMethodAwaitUninterruptibly.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodAwaitUninterruptibly.java)

`awaitUninterruptibly()`方法不同于`await()`，前者将不理会`interrupt()`动作，继续执行，而`await()`在线程触发`interrupt()`动作时将正常抛出异常。

#### 4.1.15 方法 awaitUntil() 的使用
测试类：[LockMethodAwaitUntil.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/LockMethodAwaitUntil.java)

#### 4.1.16 使用 Condition 实现顺序执行
测试类：[ConditionABC.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ConditionABC.java)


### 4.2 使用 ReentrantReadWriteLock 类
类`ReentrantLock`具有完全互斥排他的效果，即同一时间只有一个线程在执行`ReentrantLock.lock()`方法后的任务。这样虽然保证了实例变量的线程安全性，但效率确实低下的。所以`JDK`提供了一种读写锁`ReentrantReadWriteLock`类，使他可以加快运行效率，在某些不需要操作实例变量的方法中，完全可以使用读写锁`ReentrantReadWriteLock`来提升该方法的代码运行速度。

读写锁表示有两个锁，一个是读操作相关的锁，也称为共享锁；另一个是写操作相关的锁，也叫排他锁。

多个读锁之间不互斥，读写与写锁互斥，写锁与写锁互斥。 在没有线程进行写操作时，进行读取操作的多个线程都可以获取读锁，而写操作的线程只有在获取写锁后才能进行写操作。

即多个线程可以同时进行读取操作，但是同一时刻只允许一个线程进行写操作。

### 4.2.1 类 ReentrantLockReadWriteLock 的使用：读读共享
可以看出多个线程都获得了读锁`readLock`, 测试类：[ReadWriteLockBegin1.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ReadWriteLockBegin1.java)

### 4.2.1 类 ReentrantLockReadWriteLock 的使用：写写互斥
同意时刻，只有一个线程获得锁，写锁阻塞等待前一个线程释放锁, 测试类：[ReadWriteLockBegin2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ReadWriteLockBegin2.java)

### 4.2.3 类 ReentrantLockReadWriteLock 的使用：读写互斥
### 4.2.4 类 ReentrantLockReadWriteLock 的使用：写读互斥
**获得读锁未释放，写锁也会被阻塞，获得写锁未释放，读锁也会被阻塞**：测试类：[ReadWriteLockBegin3.java](https://github.com/elegance/dev-demo/blob/master/java-demo/lock/reentrantLock/ReadWriteLockBegin3.java)

### 4.3 Lock 本章总结 
完全可以使用`Lock`对象将`synchronized`关键字替换掉，而且其具有的功能是`synchronized`不具有的，`Lock`是`synchronized`的进阶。


## 5 定时器 Timer
Skills:
* 如何实现指定时间执行任务
* 如何实现按指定周期执行任务

### 5.1 定时器 Timer 的使用
`JDK`中`Timer`类主要负责计划任务的功能，也就是在指定时间开始执行某一任务。`Timer`的作用是设置计划任务，但是封装任务的类是`TimerTask`抽象类，所以具体要计划执行的任务继承`TimerTask`类即可。

#### 5.1.1 方法 schedule(TimeTask task, Date time)
指定的日期的时间执行一次任务。

1. 执行任务的时间晚于当前时间：在未来的某个时间点执行 --- timer内部的TimerThread 在实例化时start，默认非守护线程，意味任务完成后，即使没有其他线程，程序不会结束，如果设定为守护线程，如果任务运行之前，其他非守护都已经结束，那么有可能任务还未执行，程序就已经结束。
2. 计划时间早于当前时间：设定的时间点是已经过去 -- 时间点是过去，则会立即执行task任务

    1,2 测试类：[TimerTest1.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTest1.java)

3. 一个`Timer`中多个`TimerTask` 任务及延时的测试 -- 以计划执行的时间排队成队列，前者执行完后者再执行，当前者执行时间较长时会阻塞后面队列中的任务。
    3 测试类：[TimerMultiTask.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerMultiTask.java)


#### 5.1.2 方法 schedule(TimerTask task, Date firstTime, long period)
指定日期的时间后，按指定间隔周期性的执行某一任务。
1. 计划时间晚于当前时间：在未来某个时间点开始
2. 计划时间早于当前时间：设定的开始时间是已经过去的 -- 立即开始周期任务
3. 任务执行时间被延迟 -- 当间隔时间小于任务单次执行所要的时间时，后面的任务都被延时堆压，会越积越多，但还是一个一个顺序执行

    1,2,3 测试类：[TimerTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTest2.java)

4. `TimerTask`类的`cancel`方法 -- 将自身任务从`Timer`任务队列中移除，其他任务不受影响
    测试类：[TimerTaskCancel.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTaskCancel.java)
5. `Timer`类的`cancel`方法 -- timer中的任务全部清除，timer内部线程销毁，程序退出
    测试类：[TimerCancel.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerCancel.java)
6. `Timer`的`cancel()` 方法注意事项 -- 调用`cancel()` 方法不一定会停止任务，当`cancel()`方法没有竞争到内部的`queue`锁时。

#### 5.1.3 方法 schedule(TimerTask task, long delay)
以当前时间为基准，延迟指定的毫秒数执行某一任务。
    测试类：[TimerTest3.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTest3.java)

#### 5.1.4 方法 schedule(TimerTask task, long delay, long period)
以当前时间为基准，延迟指定的毫秒数开始周期性的执行某一任务。
    当前时间往后推3秒开始执行， 每3秒 执行一次myTask ,测试类：[TimerTest4.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTest4.java)

#### 5.1.5 方法 scheduleAtFixedRate(TimerTask task, Date firstTime, long period)
**`scheduleAtFixedRate`：如果任务的执行时间点已经过去，任务会在上次任务完成后立即执行** [基础点固定-立马] -- 具有追赶执行性

**`schedule`: 如果任务的执行时间点已经过去，任务将在上次完成的时间基础上加上周期时间执行，首次任务的时间已经过去则会立即执行**  [基础点按周期顺延] -- 按周期、不具追赶执行性

测试类：[TimerTest4.java](https://github.com/elegance/dev-demo/blob/master/java-demo/timer/TimerTest4.java)

## 6. 单例模式与多线程
通过单例与多线程技术的结合，在这个过程中发现很多以前为考虑的问题，学习如何使单例模式遇到多线程时安全的。

### 6.1 立即加载/"饿汉模式"
在使用类之前类的对象就已经创建好了，中文语境来看，就是饿的迫不及待，有着急迫切的含义。实现的一般做法是：私有化构造方法，声明类全局静态变量，并实例化
```java
private static MyObject myObject = new MyObject();
```
测试类：[SigletonTest1.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest1.java)

注意看下上面例子不单是单例相关的演示，而且包括：静态资源初始化的问题，**会有一个奇怪的现象，多线程访问类的普通静态方法，不是立马返回结果，而是线程被“阻塞了”**

* 静态资源初始化本身就是 单线程的(同步阻塞)，在类内部资源被初次访问时，触发静态初始化,初始化的顺序 是从上往下.
* 被静态初始化“阻塞”的方法，不是阻塞"BLOCKD"状态，而是 "RUNNABLE" 状态

其实这里应该也是`JVM`的一个编译优化，类如果被使用到，其静态的资源也不会被初始化加载。

### 6.2 延迟加载/懒汉模式
只有在`get()`时才被创建，从语境上看是“缓慢”、“不急迫”的含义。

测试类：[SigletonTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest2.java)

以上代码帮助理解"懒汉模式" + "DCL" 方式实现的单例，应对的绝大多数场景：`高并发取单例，低并发初始化实例`，巧妙的避免了`synchronized`的阻塞，又使用`synchronized`来保证单次实例化。

进阶理解，对比测试：[SyncDclMethodCompare.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SyncDclMethodCompare.java)

`剧情再次反转，在我的上面的测试环境，就用 synchronized 方法就好，不会有什么性能影响`

### 6.3 静态内置类实现单例
静态内部类实现 [SigletonTest3.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest3.java)

### 6.4 序列化反序列化实现单例
反序列化生成对象时，不通过对象的构造方法，所以会造成有另外的实例被生成，出现非单例的情况，但是反序列化内部会判断对象是否有`readResolve`方法，有就会自动调用，来达到单例的目的。

测试类： [SigletonTest4.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest4.java)

### 6.5 使用 static 代码块实现单例
其实这种和`饿汉模式`类似，都类被访问，静态资源自动初始化。

测试类： [SigletonTest5.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest5.java)

### 6.6 使用枚举 enum 数据类型实现单例
利用枚举类中枚举元素自动实例化的特点，定义几个枚举元素，产生几个实例，定义一个枚举元素，就是单个实例。

测试类： [SigletonTest6.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest6.java)

### 6.7 完善使用 enum 枚举实现单例模式
上面`6.6`的例子中违反了“职责单一原则”，完善测试类： [SigletonTest7.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest7.java)

## 7. 拾遗增补
Skills:
* 线程组的使用
* 如何切换线程状态
* SimpleDateFormat 类与多线程的解决办法
* 如何处理线程异常

### 7.1 线程的状态
线程状态枚举类：`Thread.State`

#### 7.1.1 验证 NEW、RUNNABLE、TERMINATED
#### 7.1.2 验证 TIMED_WAITING
#### 7.1.3 验证 BLOCKED
#### 7.1.4 验证 WAITING

[演示以上1-4状态的例子](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ThreadStateTest.java)

### 7.2 线程组
可以把线程归属到某一个线程组中，线程组中可以有线程对象、也可以有线程组，组中还可以有线程。就类似于一颗节点树，树分支是线程组，叶子节点就是线程，树分支上可以有更小的树分支。

线程组的作用是批量管理线程或线程组对象，有效的对线程或线程组对象进行组织。

#### 7.2.1 线程对象管理线程组： 1级关联
测试类： [GroupAddThread.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GroupAddThread.java)

#### 7.2.2 线程对象管理线程组： 多级关联
多级分组, 测试类：[GroupAddThreadMoreLevel.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GroupAddThreadMoreLevel.java)

#### 7.2.3 线程组自动归属特性
自动归属就是自动归到当前线程组中。
测试类：[AutoAddGroup.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/AutoAddGroup.java)

#### 7.2.4 获取根线程组
通过线程`getThreadGroup().getParent()`获取线程所在组的父级线程组，得到为null时则已经是最根级别的组了。

测试类：[GetParentGroup.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GetParentGroup.java)

#### 7.2.5 线程组里加线程组
利用`ThreadGroup`构造函数显示指定父线程组，测试类：[GroupAddGroup.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GroupAddGroup.java)

#### 7.2.6 组内的线程批量停止
测试类：[GroupInnerStop.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GroupInnerStop.java)

#### 7.2.7 递归与非递归取的组内对象
* `getThreadGroup().enumerate(putList, isRecurse)` 可以指定是否递归子孙组
* `activeGroupCount()` 取的数量是包括子孙组的

测试类：[GroupRecurse.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/group/GroupRecurse.java)

### 7.3 使线程具有有序性
正常情况下，多个线程执行任务的时机是无序的。可通过改造代码使他们具有有序性。

测试类：[ThreadRunSync.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ThreadRunSync.java)

这里的顺序控制逻辑其实可以利用其它方式，如指定`nextFlag`，或者使用`ReentrantLock`的多个`Condition`来指定唤醒执行。

### 7.4 SimpleDateFormat 非线程安全
`SimpleDateFormat`主要负责日期的转换和格式化，但在多线程环境中容易误用，比如全局静态化实例、全局实例多线程访问造成转换不准确。

#### 7.4.1 出现异常
发生异常的原因：跟踪`SimpleDateFormat` 源码可以发现 内部 存储了全局变量： `Calendar`，也就是单个实例，多线程 都会访问操作 这个`Calendar`，造成混乱，最终转换错误或出现转换异常
测试类：[FormatError.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/FormatError.java)


#### TODO 大总结： 
* 锁存在的意义
* 相关类 方法一览,截图
* `synchronized` ,内部锁， 锁对象， Object() 类方法：截图  wait()/wait(long)、notify()、notifyAll()
* `LOck`接口方法，`ReentrantLock`类方法，`ReentrantReadWriteLock`类方法 读、写锁特性
* `Timer`类方法
* 补充 ThreadGroup、Thread.enumerate 等方法
* 补充 管理类：ThreadPoolExcutor 等

#### TODO 想到的几个问题
1. 类内部的`public static xxMethod() {}` 有可能被阻塞吗？
> 静态资源初始化造成“阻塞”，但是

2. `thead.join()`方法内的`wait(0)`可以用`sleep(0)`代替实现吗？

3. `thread.join()` 如果`thread`内部启动了新的线程，那么`thread.join()`后的代码会等待`thread`内部线程再执行吗？

4. 线程基础 `Object.wait()`与 `Object.notify()` 都是干什么用的，怎么用的？

5. 怎么看待单例模式中懒汉模式结合`DCL`的方式解决线程安全问题？ 
> 初期理解，以高并发初次实例化的场景看待问题，觉得存在几处浪费工作，代码结构更加不清晰。 
不过后来发现，其实应该“乐观”看待，`高并发初始化`场景少，`高并发取实例`场景多
理解代码：[SigletonTest2.java](https://github.com/elegance/dev-demo/blob/master/java-demo/sigleton/SigletonTest2.java)

6. 发生线程安全问题一般有哪几种情况？ 方法内定义的变量，可能引起线程安全问题吗？
> 容易出现问题的一般有这几中情况：
1. 全局的静态变量， 问题的发生点在于`全部线程都能访问这个静态变量`，少有的web下应用也常见的场景：定义一个全局静态的`map`来缓存数据  `public static HashMap<String, String> cacheMap = new HashMap<String, String>();`、定义一个全局
2. 全局的实例变量， 问题的发生点就在于`单个实例会被多个线程访问到`，web下其实遇到的较少，一般不同的线程会建立不同的实例，但是也有单个实例被多个线程访问到的情况，比如网上看到的一个利用`i++`全局变量作为`sessionId` [链接](http://www.blogjava.net/aoxj/archive/2012/06/16/380926.html)。
3. 方法内的"局部"变量，这种在实际中发生的较少，问题发生在：`方法内定义变量，方法内部再启动多个Thread，此时"局部"变量就成了新启动的几个线程的共享变量了`，测试类：[MethodVaribleSecurity.java](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/MethodVaribleSecurity.java)

7. 变量到底怎样的规则存储堆、栈中？
> 一般的说法：基础值类型存栈中，对象在堆中。 不管怎样 基础类型、对象不都是定义在class内么，整个实例化后也是对象，那岂不是都在堆中了？
比如:`Person`类，有属性：`int age`、`String name`、`Array<Person> friends`，分析下这个类的实例时如何存储的吧。