---
title: java线程池实现原理分析
tags: 多线程
categories: java
date: 2017-06-15 10:48:12
---


> 沉下心，才会远离烦恼。

`java`提供了多线程，用户只要继承`Thread`类或实现`Runnable`接口就能轻松达到多线程的目的。简单的应用时，我们硬编码固定的线程数可能就能满足要求，但是涉及到线程资源的重复利用、管理、响应性能等，我们就需要线程池来协助了。类似数据库连接池，线程池主要有以下优点：
1. 创建线程也需要消耗，池中线程可重复利用，降低资源消耗
2. 线程提前创建，提高响应速度
3. 提高线程可管理性

`Java 1.5`中引入了`Executor`框架把任务的**提交**和**执行**进行了解耦。只需要**定义好任务**，然后**提交**给线程池。而不用关心任务如何被执行、被哪个线程执行、以及什么时候执行等。

`Executor`接口：
```java
public interface Executor {
    void execute(Runnable command);
}
```
`Executor`只是一个简单的接口，但它为灵活而强大的框架创造了基础，`Executor` 基于 **生产者-消费者模式**。如果你在程序中实现一个生产者-消费者的设计，使用`Executor`通常是最简单的方式。

##### Demo 1
```java
public class ExecutorCase {

    private static Executor executor = Executors.newFixedThreadPool(10); // 2. 创建一个包含10个线程的线程池 executor

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            executor.execute(new Task()); // 3. 20个任务提交给 线程池 executor 来执行
        }
    }

    static class Task implements Runnable { // 1. 定义任务
        public void run() {
            System.out.println(Thread.currentThread().getName());
        }
    }
}
```

#### ThreadPoolExecutor
`Executors`是java线程池的工厂类，通过它可以快速初始化一个符合业务需求的线程池，如`Executors.newFixedThreadPool`方法产生一个拥有固定数量的线程池。
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
    }
```
其中`ExecutorService`接口继承接口`Executor`，方法内本质是通过不同参数初始化`ThreadPoolExecutor`，下面看下这个方法是怎么定义的：
```java
    public ThreadPoolExecutor(int corePoolSize,
                                int maximumPoolSize,
                                long keepAliveTime,
                                TimeUnit unit,
                                BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), defaultHandler);
    }
```
最准确的注释，你还可以查看`jdk`源码中的英文注释。

##### corePoolSize
要保存在池中的线程数，包括空闲的。除非`allowCoreThreadTimeOut`参数被设置。如果执行了`prestartAllCoreThreads()`方法，将提前创建并启动所有核心线程。

##### maximumPoolSize
线程池允许的最大线程数，超出的提交将进入`BlockingQueue`阻塞队列，故`executor.execute(xxTask)`之后的代码不会因线程数量的限定而阻塞。

##### keepAliveTime
线程的空闲存活时间。该参数只在线程数大于核心线程数时起作用，结合`corePoolSize`的注释理解。

##### unit
`keepAliveTime`的单位。

##### workQueue
保存任务的阻塞队列，限定了队列中只能存储实现了`Runnable`接口的任务。`BlockingQueue<Runnable>`接口在`JDK`中有以下实现：
* `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列。
* `LinkedBlockingQueue`: 基于链表机构的阻塞队列。
* `SynchronusQueue`: 一个不存储元素的阻塞队列，每个插入操作必须等另一个线程调用移除操作，否则插入一直处于阻塞状态。
* `PriorityBlockingQueue`: 具有优先级的无界阻塞队列。

前两者的味道类似于`ArrayList`与`LinkedList`，主要是具有数据结构`Array`、`链表`的特点。而`SynchronusQueue`则类似于`CSP`场景中，一个没有`buffer`缓冲的`channel`，《七周七并发模型》中一书中的`CSP`模型中提到`新手往往会认为有缓存的channel会比无缓存的channel应用更广泛，但实际情况却恰恰相反。`，虽然这不一定对，但是这提醒了我们一定要根据场景去选择使用。`PriorityBlockingQueue`则是更接近场景需求优先级的解决办法。

##### threadFactory
创建线程的工厂，具有名称前缀`pool-`,主要实现如下：
```java
DefaultThreadFactory() {
    SecurityManager s = System.getSecurityManager();
    group = (s != null) ? s.getThreadGroup() :
                            Thread.currentThread().getThreadGroup();
    namePrefix = "pool-" +
                    poolNumber.getAndIncrement() +
                    "-thread-";
}
```

##### handler
任务队列达到限制的饱和处理策略。线程池提供了4中策略：
* `AbortPolicy`: 直接抛出异常，默认策略
* `CallerRunsPolicy`: 用调用者所在线程来执行任务
* `DiscardOldesPolicy`: 丢弃队列最前面的任务，执行新的任务。类似于`CSP`模型中的`sliding`方式
* `DiscardPolicy`: 直接丢弃任务。类似于`CSP`模型中的`dropping`方式
如果以上都不满足你的需求，你还可以自己实现`RejectedExecutionHandler`接口，自定义饱和处理策略，比如日志记录、邮件提醒等。

#### Executors
`Executors`工厂类提供了线程的初始化接口，主要有如下几种：

##### newFixedThreadPool
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
    }
```
功能如其名，入参只有一个数字。指定固定的线程个数，其中 `corePoolSize == maximumPoolSize`，`0L`代表不会释放`core`线程，使用`LinkedBlocingQueue`作为任务队列。

##### newCachedThreadPool
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
初始化一个缓存限定时间线程的线程池，默认缓存60s，线程空闲超过60s时会自动释放线程，不会保留`core`线程。

##### newSingleThreadExecutor
```java
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```
创建单个工作线程的`Executor`，等同于`newFixedThreadPool(1, threadFactory)`，返回的`Executor`不可再重新配置。

##### newScheduledThreadPool
```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```
创建的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期同步数据。

##### newWorkStealingPool
```java
    public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```
`jdk 1.8`中出现，创建一个`work-stealing`的线程池，内部`ForkJoinPool`使用一个并行因子来创建，默认为主机CPU的可用核心数。

#### 实现原理
可以从方法内部的实例化代码看出，前三者都是`ThreadPoolExecutor`类实现的，`newScheduledThreadPool`返回类型都发生了变化，其实现是`ScheduledThreadPoolExecutor`，另外`newWorkStealingPool`返回值没有变化，说明暴露给外部的使用上没有变，内部使用`ForkJoinPool`来做了优化。

#### ThreadPoolExecutor 线程池内部状态
```java
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0)); // ctl 包含了两个概念，因为两个的关联关系，巧妙的组合在一起；高3位表示线程池状态； 低29位 表示workerCount 
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS; // 高3位为111
    private static final int SHUTDOWN   =  0 << COUNT_BITS; // 高3位为000
    private static final int STOP       =  1 << COUNT_BITS; // 高3位为001
    private static final int TIDYING    =  2 << COUNT_BITS; // 高3位为010
    private static final int TERMINATED =  3 << COUNT_BITS; // 高3为为011
```
* `RUNNING` : 线程池会接收新任务，并处理排队的任务 
* `SHUTDOWN`: 线程池不接收新任务，但会处理队列中的任务
* `STOP`: 线程池不接收新人无，不处理队列中的任务，并中断正在运行的任务
* `TIDYING`:  所有任务已经终止，workCount为零，线程过渡到TIDYING状态
* `TERMINATED`: terminated() 钩子方法运行完毕

#### 任务提交
线程池框架提供了两种方式提交任务：
* `Executor.execute(Runnable command)`  返回void, 不关心返回值
    ```java
    void execute(Runnable command);
    ```
* `ExecutorService.submit(Callable<T> task)` 返回`Future<T>`
    ```java
    <T> Future<T> submit(Callable<T> task)
    ```

##### ThreadPoolExecutor.execute 的实现
```java
    int c = ctl.get(); // 获取原子变量的值
    if (workerCountOf(c) < corePoolSize) { // 统计 workerCount 如果小于 corePoolSize
        if (addWorker(command, true)) // 则 addWorker 来创建线程来执行任务
            return;
        c = ctl.get(); // 如果上面的 command没有被执行，则再获取一次
    }
    if (isRunning(c) && workQueue.offer(command)) { // 判断线程池当前状态为运行，并且成功将任务插入到 阻塞队列中
        int recheck = ctl.get(); // 再次取新值判断
        if (! isRunning(recheck) && remove(command)) // 线程池停止了，并且任务被成功移除
            reject(command); // reject handler
        else if (workerCountOf(recheck) == 0) // workerCount 为 0 
            addWorker(null, false); // ？经过上面内部的层层判断，逻辑不太会走到这里，不太理解此处的 null，在我看来这是一种“弃疗”的表现。
    }
    else if (!addWorker(command, false)) // 再次使用 maximun 作为限定数尝试添加
        reject(command); // 线程池饱和处理 reject handler
```

##### addWorker的实现
`addWorker`方法主要是创建线程，执行任务
```java
 private boolean addWorker(Runnable firstTask, boolean core) {
        retry: // 重试标记层级
        for (;;) { // 无限定条件的for
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false; // 线程池状态不满足则直接返回 false

            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY || // 如果 workerCount 大于 容量
                    wc >= (core ? corePoolSize : maximumPoolSize)) 
                    // 如果 workerCount 大于 核心线程数（外部以核心线程数作为判断依据时） 或 workerCount 大于 最大线程数(外部以最大线程数作为判断依据时)
                    return false; // 容量受限，返回false
                if (compareAndIncrementWorkerCount(c)) // 通过上面的检测，并更新数值成功
                    break retry; // 跳出 多层循环，往方法的下半部分继续执行
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs) // 上面未设值成功，状态 没有变化
                    continue retry; // 继续外部循环
                // else CAS failed due to workerCount change; retry inner loop
            }
        }
```
以上是这个方法的前半部分，主要是线程池状态检测、线程池数量限制检测、线程池相关数量与状态的更新。以下是下半部分代码，主要是创建线程，执行任务:
```java
        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask); // 线程池的工作 通过 Worker 类，Worker 类继承了 AQS （AbstractQueuedSynchronizer）
            final Thread t = w.thread; // 线程是取的 worker中的线程，而worker中的线程是线程池初始化的 线程工厂创建的
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock; // ReentrantLock 锁的保证下，插入到 workers(HashSet结构)中
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w); // 加入 hashSet 中
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    t.start(); // 添加成功 开始 运行
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
```

##### Worker
```java
    Worker(Runnable firstTask) { // 
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this); // 初始化线程池的 线程工厂来创建线程
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this); // runnable 接口的实现，启动线程 运行任务
    }
```

##### runWorker实现
```java
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts // 线程启动后通过unlock释放锁
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) { // 首次进入会执行 firstTask，后面则主要通过getTask()方法取队列中的任务
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task); // 前置任务
                    Throwable thrown = null;
                    try {
                        task.run(); // 开始执行任务
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown); // 后置任务
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
```

##### getTask 实现
```java
    private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : // 在指定时间内，阻塞队列没有新的任务，则会返回null
                    workQueue.take(); // 如果阻塞队列为空，当前线程被挂起；队列中有任务加入时，线程被唤醒，返回任务
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
```

#### Future 和 Callable 的实现
##### Demo 2
```java
public class ExecutorCase2 {
    private static ExecutorService executor = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        Future<String> future = executor.submit(new Task()); // 提交异步任务
        System.out.println("do other things");
        
        String result = future.get(); // 线程阻塞
        System.out.println("asynchronus result:" + result); // 后面跟随的代码，等待上面的阻塞解除执行完后，才会执行
    }

    static class Task implements Callable<String> {

        public String call() throws Exception {
            TimeUnit.SECONDS.sleep(2); // 睡眠2s，模拟异步耗时
            return "this is future case";
        }
    }
}
```
在实际业务场景中，`Future`与`Callable`一般是成对出现的，`Callable`负责执行任务产生结果，`Future`则是负责获取结果
1. `Callable`接口类似`Runnable`接口，只是`Runnable`没有返回值。所以如果你关心你每个任务的执行返回结果，就可以采用`Callable`，否则你就直接使用`Runnable`就好了。
2. `Callable`执行的任务如果发生异常，该异常也会被返回，即`Future`可以拿到异步执行任务的各种结果。
3. `Future.get`方法是阻塞的，直到`Callable`任务执行完成

##### submit实现
```java
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
```
`Callable`任务通过`submit()`方法被封装为一个`RunnableFuture`的`FutureTask`.

##### FutureTask
```java
    /*
     * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
     */
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6

    //...
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

```
* state 存储 `FutureTask`的状态，
* 构造初始状态为`NEW`，构造函数使用`callable`成员变量存储了入参`callable`任务
* `FutureTask`实现了`Runnable`接口，最终实际执行的是`FutureTask`中的`run`方法

##### FutureTask.get实现
```java
    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING) // 状态检查
            s = awaitDone(false, 0L);
        return report(s);
    }
```
内部通过`awaitDone`方法阻塞，代码如下：
```java
    private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        WaitNode q = null;
        boolean queued = false;
        for (;;) {
            if (Thread.interrupted()) {
                removeWaiter(q);
                throw new InterruptedException();
            }

            int s = state;
            if (s > COMPLETING) {
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING) // cannot time out yet
                Thread.yield();
            else if (q == null)
                q = new WaitNode();
            else if (!queued)
                queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                     q.next = waiters, q);
            else if (timed) {
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L) {
                    removeWaiter(q);
                    return state;
                }
                LockSupport.parkNanos(this, nanos);
            }
            else
                LockSupport.park(this);
        }
    }
```
* 如果线程被中断，则抛出异常
* 如果状态大于`COMPLETING`说明已经完成，直接返回状态即可
* 如果状态等于`COMPLETING`说明已经完成，使用`yield`让渡一下`cpu`，`state`则会过度到`NORMAL`了
* 通过`WaitNode`简单链表封装当前线程，并通过`UNSAFE`添加到`waiters`链表
* 最终通过`LockSupport`的`park`或`parkNanos`来挂起线程，另外`finishCompletion`方法中会`unpark`

##### FutureTask.run 实现
```java
    public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```
* `run` 方法是线程池中的线程来执行的，而非主线程
* 执行`callable.call`方法来运行任务
* `call`通过时用`set`方法来保存结果
* `call`出现异常时用`setException`方法来保持异常信息

##### set/setException
```java
    protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }
    protected void setException(Throwable t) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = t;
            UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
            finishCompletion();
        }
    }
```
通过`UNSAFE`修改了`FutureTask`的状态，最终都通过调用`finishCompletion`方法通知主线程任务完成。

##### finishCompletion
```java
    private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }
```
* 更新`waiters`的值
* `LockSupport.unpark(t)`唤醒主线程

---
主要参考：
* [占小狼-深入分析java线程池的实现原理](http://www.jianshu.com/p/87bff5cc8d8c) 
* 《JAVA并发编程实践》
