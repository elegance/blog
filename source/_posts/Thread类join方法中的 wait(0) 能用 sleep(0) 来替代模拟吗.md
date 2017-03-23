---
title: Thread类join方法中的 wait(0) 能用 sleep(0) 来替代模拟吗
tags:
  - 多线程
  - 源码阅读
categories:
  - java
date: 2017-03-23 13:41:15
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
3. 第`10`行`if`代码块内的即是`join()`的实现
4. 逻辑：`判断 线程实例 isAlive()返回的是 true，则 wait(0)， 然后再次进入前面的判断，否则退出`，也就是在目标线程执行完毕前，这里会不断循环取状态判断、wait(0)， 这里的`不断`即起到了`当前线程等待目标线程执行完毕的作用了`

这里的做法看起来就像这个思路，`比如需要等某人做完了一些事，我们再去做另外一件事，那么我们可不断的去问这个人他是不是做完了那些事`

#### 那么我们能将`join`方法中的`wait(0)`用`sleep(0)`替代吗？
实验一下吧：
```java
  1 public class TestSyncMethodLockAlive {                                                                                                                                  
  2     public static void main(String[] args) throws InterruptedException {
  3         Thread targetThread = new Thread(() -> {
  4             System.out.println("target begin run state=" + Thread.currentThread().getState());
  5             sleep(1000); // 模拟目标线程做某些事情，比如查询数据库
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
可以看以下信息(截取了部分)：
```
  1 "target-thread" #10 prio=5 os_prio=0 tid=0x000000001d5f4000 nid=0x24140 waiting for monitor entry [0x0000000000000000]                                                  
  2    java.lang.Thread.State: BLOCKED (on object monitor)
  3 "Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000001bf06800 nid=0x21a24 in Object.wait() [0x000000001d15f000]
  4    java.lang.Thread.State: WAITING (on object monitor)
  5         at java.lang.Object.wait(Native Method)
  6         - waiting on <0x000000076b406f58> (a java.lang.ref.ReferenceQueue$Lock)
  7         at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
  8             - locked <0x000000076b406f58> (a java.lang.ref.ReferenceQueue$Lock)
  9         at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
 10         at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)
 11 "main" #1 prio=5 os_prio=0 tid=0x0000000002458000 nid=0x244ec runnable [0x000000000293f000]
 12    java.lang.Thread.State: RUNNABLE
 13         at java.lang.Thread.sleep(Native Method)
 14         at commu.TestSyncMethodLockAlive.main(TestSyncMethodLockAlive.java:31)
 15         - locked <0x000000076b4b1c40> (a java.lang.Thread)
```
内容分析
1. 第`1-2`行: `target-thread` waiting for monitor entry ` 意味“在等待进入一个临界区， ”，这里的信息没有期望的"-waiting to lock<0xx>" 不能看出来到底是被哪个锁阻塞。 (jconsole中都看不到此线程了)
2. 第`11-15`行: `main`方法运行的主线程`，主线程一直处于`RUNNABLE`状态，并且持有锁 `- locked <0x764b1c40>`
3. 第`3-10`行：daemon类型，一般为jvm守护线程，这里的`Finalizer`线程主要是给执行完`run`的线程处理一些`身后事`，比如将线程移除引用队列。可以看出这里等待的锁，不是直接`main`线程持有的锁，而是线程本身在此前`locked`的一个锁，只是这里在等待外部调用某个方法来`notify`唤醒线程，而间接的在外部依赖了`main`一直持有的锁，而这个外部可能是在`native`中而无法跟踪了。

思考弯路：
1. `synchronized` 不是一直持有 线程对象的锁么， 为何 "target-thread"线程在等待锁？  ---->   `synchronized`是main方法持有了锁, "target-thread" 是线程内部需要这个锁

总结：
1. 创建的线程运行完`run`方法后，线程调度器对其还有`身后事`要处理的，并且间接同步的使用到“线程实例”，并且这个"间接"不好追踪，所以最好不要再线程实例外部来将线程实例作为同步条件来使用。
2. 线程内做的操作影响线程本身的状态
3. 将“线程实例”作为锁与“普通对象”作为锁本质一样，将其普通看待不要受到干扰，所以不要把`targetThread.wait()`方法看的特殊了，理解为`lock.wait()`就好

#### 回到之前的问题“我们能将`wait(0)`用`sleep(0)`替代吗？”
其实从控制台打印信息，可以看出`isAlive返回的一直为true，因为"身后事"的处理被阻塞，所以线程还是alive的，但是线程状态已经由"RUNNABLE"转变为"BLOCKED"了`.

![](/images/xbq/idea.gif)那么我们可以将我们的`while (targetThread.isAlive())` 的判断条件修改下`while (!targetThread.getState().equals(Thread.State.BLOCKED))` 修改好了接着测试看下控制台的输出吧：
```
### log ### targetThread: state=TIMED_WAITING, isAlvie=true
target end  run state=RUNNABLE
synchronized anlog join over.
```
bingo! 按我们期盼的顺序输出了 ![](/images/xbq/big-laugh.gif)

高兴的有些早了，多运行几次发现有时候会是这样：
```
synchronized anlog join over.BLOCKED
target begin run state=RUNNABLE
```
main先结束，target才开始，再次证明多线程问题的偶发性，一不小心就以为万事大吉，其实已经暗藏危机了。其实我们这里的根本原因是`忽略了除了NEW/TERMINATED状态，其它的几个线程状态都有可能随意切换`。 我们这里单纯认为`BLOCKED`就是由处理“身后事”造成的，是不严谨的，这里多种其他的情况情况要处理比如:
1. `targetThread.start()`是`synchronized`方法，如果它比main`synchronized`块后拿到锁，会造成线程`BLOCKED`
2. `run`方法内部的`System.out.println`方法为同步方法也有可能造成线程 `BLOCKED` [这也是为什么在打包的代码里最好不要出现`System.out.println`的原因]
3. `run`方法内部的`Thread.sleep(1000)` 是`native`的，经测试这里暂时`不会造成BLOCKED`，但是这里是模拟的业务，如果真实业务访问数据库、读写文件等操作其他资源都有可能造成`BLOCKED`

现有情况的解决方式：
1. 基于`1`的问题，可以在`targetThread.start()后再 sleep(50)`
2. `2`的问题，将打印信息的方式改为`logger`方式输出，比如`java.util.logging.Logger.getGlobal().info`非阻塞来替代
3. `3`如果只是基于我们现有的模拟任务，不用修改可以暂时满足。

结论：**基于上面的测试代码**，可以用`sleep(0)`代替`wait(0)`的方式来达到`join()`的效果。

我们根据日志结果做了不科学的事情：利用判断 thread.getState() 尝试代替 thread.isAlvie() 。只有状态`NEW`与`TERMINATED`时isAlive()为false，其他状态对应的都是true;

**不科学的使用，将给你带来各种意外，需要步步跟踪测试才能做出严谨的判断，除非你只是本着探索的心在学习，否则不建议这样使用**


#### 技能点：
1. 线程状态 - [帮助理解线程状态的例子](https://github.com/elegance/dev-demo/blob/master/java-demo/thread/ThreadStateTest.java)
    * `NEW` 创建了未 start
    * `RUNNABLE` 有可能正在运行(RUNNING)，也有可能在就绪队列中(Ready)，这个取决于 线程调度器
    * `TIMED_WAITING` 挂起，等待指定的时间后自动恢复
    * `WATING` 挂起，被动恢复，依赖其他线程的操作才会唤醒
    * `BLOCKED` 阻塞，被动恢复执行，依赖其他线程释放相关资源的锁
    * `TERMINATED` 执行完毕的线程
2. `wait()`、`wait(long)`
    * `wait()`释放锁，线程进入`WAITING`状态，无限期等待另一个线程执行某一操作，如在锁对象上执行`notify()`/`notifyAll()`
    * `wait(long)`释放锁，线程进入`TIMED_WAITING`，等待指定的时间**自动**结束指定
3. `sleep(long)`
    * `sleep(long)` 如果在同步块内，线程将不会释放锁，一直持有，线程进入`TIMED_WAITING`，等待指定的时间**自动**往下执行
3. java dump 分析：
    * [JAVA Thread Dump 分析综述](http://www.blogjava.net/freeman1984/archive/2015/12/14/428645.html)
    * [各种 Java Thread State 第一分析法则](http://www.cnblogs.com/zhengyun_ustc/archive/2013/03/18/tda.html)
    * [三个实例演示 Java Thread Dump 日志分析](http://www.cnblogs.com/zhengyun_ustc/archive/2013/01/06/dumpanalysis.html)