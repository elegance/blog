---
title: Thread类源码——join(long)方法的实现
categories:
- java
tags:
- 源码阅读

---

#### Thread 线程类 `join`方法实现如下
都知道`join()` 经常用于等待前一个线程执行完毕，再执行后面的任务，那`join()`方法是如何做到这个"等待"的呢？ 
```java
  1 public final synchronized void join(long millis)                                                                                                                        
  2 throws InterruptedException {
  3 |   long base = System.currentTimeMillis();
  4 |   long now = 0;
  5 
  6 |   if (millis < 0) {
  7 |   |   throw new IllegalArgumentException("timeout value is negative");
  8 |   }
  9 
 10 |   if (millis == 0) { 
 11 |   |   while (isAlive()) {
 12 |   |   |   wait(0);
 13 |   |   }
 14 |   } else {
 15 |   |   while (isAlive()) {
 16 |   |   |   long delay = millis - now;
 17 |   |   |   if (delay <= 0) {
 18 |   |   |   |   break;
 19 |   |   |   }
 20 |   |   |   wait(delay);
 21 |   |   |   now = System.currentTimeMillis() - base;
 22 |   |   }
 23 |   }
 24 }
```
分析如下：
1. 方法无`static`修饰，说明其为**线程实例**的方法
2. 方法有`syncrhonized`修饰，意味着可能需要竞争锁才能进入方法，进入这个方法后将锁住**线程实例**这个对象
3. 第`10`if代码块内的即是`join()`的实现
4. 逻辑：`判断 线程实例 isAlive()返回的是 true，则 wait(0)， 然后再次进入前面的判断，否则退出`，也就是在目标线程执行完毕前，这里会不断循环取状态判断、wait(0)， 这里的`不断`即起到了`当前线程等待目标线程执行完毕的作用了`

这里的做法看起来就像这个思路，`比如需要等某人做完了一些事，我们再去做另外一件事，那么我们可不断的去问这个人他是不是做完了那些事`

#### 那么我们能将`wait(0)`用`sleep(0)`替代吗？
实验一下吧：
```java
  1 public class TestSyncMethodLockAlive {                                                                                                                                  
  2     public static void main(String[] args) throws InterruptedException {
  3         Thread targetThread = new Thread(() -> {
  4             System.out.println("target begin run state=" + Thread.currentThread().getState());
  5             sleep(1000);
  6             System.out.println("target end  run state=" + Thread.currentThread().getState());
  7         }, "target-thread");
  8 
  9         // 定时输出 目标线程的状态 与 isAlive 值
 10         new Thread(() -> {
 11             while (true) {
 12                 System.out.printf("### log ### targetThread: state=%s, isAlvie=%s\n", targetThread.getState(), targetThread.isAlive());
 13                 if (!targetThread.isAlive() && !targetThread.getState().equals(Thread.State.NEW)) {
 14                     break;
 15                 }
 16                 sleep(500);
 17             }
 18             System.out.printf("### log ### targetThread: state=%s, isAlvie=%s\n", targetThread.getState(), targetThread.isAlive());
 19         }).start();
 20         targetThread.start();
 21 
 22         // 模拟 Thread 的 join 方法
 23         synchronized (targetThread) {
 24             while (targetThread.isAlive()) { // 取线程状态
 25                 targetThread.wait(0);
 26             }
 27             System.out.println("synchronized anlog join over.");
 28         }   
 29     }   
 30     
 31     public static void sleep(long millis) {
 32         try {
 33             Thread.sleep(millis);
 34         } catch (InterruptedException e) {
 35             e.printStackTrace();
 36         }   
 37     }   
 38 }
```
这里我们用`23-28`行`wait(0)`的模式模拟了`Thread`的`join`方法，运行测试下，没有问题。 下面我们尝试把`25`行替换为`Thread.sleep(0);`测试下，发现控制台打印没有停止,输出以下信息：
```
target begin run state=RUNNABLE
### log ### targetThread: state=RUNNABLE, isAlvie=true
### log ### targetThread: state=TIMED_WAITING, isAlvie=true
target end  run state=RUNNABLE
### log ### targetThread: state=BLOCKED, isAlvie=true
### log ### targetThread: state=BLOCKED, isAlvie=true
....
```
分析下：
1. `taget end run` 的输出意味着线程内部的代码已经运行结束了，但是目标线程状态的`isAlvie()`返回的true，所以我们期盼的`27`行也不会打印
2. 为什么线程内部代码执行完了，还是alive的呢？ 我们发现：线程状态由`RUNNABLE`转为`BLOCKED`状态，而不是期望值`TERMINATED`
3. 状态为`BLOCKED` 意味着线程阻塞，线程等待某个锁 ，死锁了？ 我们一起来分析下堆栈内存

##### 分析堆栈内存
```bash
$ jps # 拿到进程 pid
$ jstack -l pid # 直接查看下 堆栈信息，可以发现："target-thread" BLOCKED ，但是好像没有发现 "deadlock" 的信息，没有发现死锁那为什么还会一直等待呢？
# jconsole.exe 也可以查看
# $ jmap -dump:live,format=b,file=heap.bin pid # 将堆栈信息导出至文件，离线分析下
```
可以看以下信息：
```
  1 "target-thread" #10 prio=5 os_prio=0 tid=0x000000001d5f4000 nid=0x24140 waiting for monitor entry [0x0000000000000000]                                                  
  2    java.lang.Thread.State: BLOCKED (on object monitor)
  3 
  4 "Service Thread" #9 daemon prio=9 os_prio=0 tid=0x000000001d38d800 nid=0x256e8 runnable [0x0000000000000000]
  5    java.lang.Thread.State: RUNNABLE
```
TODO: 第`1`行 "waiting fro monitor entry[0x0000000000000000]" 意味着等待这个锁， 第`4`行 runnable [0x0000000000000000] 意味着在这个对象上运行着
`synchronized` 不是一直持有 线程对象的锁么？为何 "target-thread"线程 在等待这个。。。恍然大悟， target-thread 线程本身 需要这个锁，而我们`synchronized`是main方法持有了锁
所以就BLOCKED的了。



#### 技能点：
1. 线程状态 - [帮助理解线程状态的例子](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ThreadStateTest.java)
    * `NEW`
2. `wait()`、`wait(long)`
    * `wait()`释放锁，线程进入`WAITING`状态，无限期等待另一个线程执行某一操作，如在锁对象上执行`notify()`/`notifyAll()`
    * `wait(long)`释放锁，线程进入`TIMED_WAITING`，等待指定的时间**自动**结束指定

