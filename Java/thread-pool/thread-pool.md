---
title: 线程池整理
date: 2022-09-03
tags: [Java,多线程,线程池]
toc: true
---

# 1、池化思想

**背景：经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。**

**思路：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。 可以避免频繁创建销毁、实现重复利用。类似生活中的公共交通工具。**

**池化思想：程序运行本质就是占用系统资源，要优化资源的使用！就要用到池化技术，池化技术就是事先准备好一些资源，要用就来这里拿，用完之后还回来。线程池、连接池、内存池、对象池...**

<!--more-->

<font size=4 style="font-weight:bold;background:yellow;">线程池的优点</font>

- 线程可以复用，可以控制最大并发数

- 提高响应速度（减少了创建新线程的时间）；
- 降低资源消耗（重复利用线程池中线程，不需要每次都创建）；
- 便于线程管理，核心参数如下：
  - corePoolSize：核心池的大小
  - maximumPoolSize：最大线程数
  - keepAliveTime：线程没有任务时最多保持多长时间后会终止

JDK 5.0 起提供了线程池相关 API：**`ExecutorService`** 和  **`Executors`**

- ExecutorService：真正的线程池接口，常见子类 `ThreadPoolExecutor`。
  - `void execute(Runnable command)`：执行任务/命令，没有返回值，一般用来执行 Runnable；
  - `<T>Future<T> submit(Callable<T> task)`：执行任务，有返回值，一般用来执行 Callable；
  - `void shutdown()`：关闭连接池 
- Executors：工具类、线程池的工厂类，用于创建并返回不同类型的线程池。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolTest {
    public static void main(String[] args) {
        // 1、创建线程池
        // newFixedThreadPool参数为线程池大小
        ExecutorService service = Executors.newFixedThreadPool(5);
        // 2、执行
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        // 3、关闭连接
        service.shutdown();
    }
}

class MyThread implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```

<font size=4 style="font-weight:bold;background:yellow;">线程池的种类</font> 

Executors：工具类，线程池的工厂类，用于创建并返回不同类型的线程池，JDK 5.0后开始提供。

```java
Executors.newCachedThreadPool()      // 创建一个可根据需要创建新线程的线程池
Executors.newFixedThreadPool(n)      // 创建一个可重用固定线程数的线程池
Executors.newSingleThreadExecutor()  // 创建一个只有一个线程的线程池
Executors.newScheduledThreadPool(n)  // 创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
```

<font size=4 style="font-weight:bold;background:yellow;">线程池的创建</font>

- Executors
- ThreadPoolExecutor

# 2、三大方法

三大方法指的是创建线程池的三种方法，以下是这三种线程池的实现，可以发现底层使用的都是 ThreadPoolExecutor。

**① newSingleThreadExecutor：**<font color='#0000ff' style="font-weight:bold;">线程池中只有一个线程</font>，执行多个线程时需要重复执行这一个线程；

```java
ExecutorService threadPool = Executors.newSingleThreadExecutor();   // 只有一个线程的线程池

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,									// 核心数量和最大数量都是1
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));	// 工作队列使用链表实现的阻塞队列
}
```

**② newCachedThreadPool：**<font color='#0000ff' style="font-weight:bold;">线程池中的线程数不固定</font>，依据所执行的任务进行动态调节。 

```java
ExecutorService threadPool = Executors.newCachedThreadPool();   	// 大小可变的线程池，遇强则强遇弱则弱

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,				// 核心数量是0，最大数量没有上限
                                  60L, TimeUnit.SECONDS,			// 某个线程空闲超过60s就会被销毁
                                  new SynchronousQueue<Runnable>());	// 工作队列是一个长度为0的队列
}
```

**③ newFixedThreadPool：**<font color='#0000ff' style="font-weight:bold;">需要指定线程池中线程的数量</font>，执行多线程时这些线程轮番执行；

```java
ExecutorService threadPool = Executors.newFixedThreadPool(5);   	// 创建一个固定大小的线程池

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,				// 线程数量固定
                                  0L, TimeUnit.MILLISECONDS,		// 由于线程数量固定，所以这两个参数无用
                                  new LinkedBlockingQueue<Runnable>());
}
```

<font size=4 style="font-weight:bold;background:yellow;">关于线程池的创建</font>（阿里 Java 编程规范手册）

【强制】线程池不允许用 `Executors` 去创建，而是要通过 **`ThreadPoolExecutor`** 的方式自己指定参数创建，这样的处理方式可以让写的同学更加明确线程池的运行规则，避免资源耗尽的风险。

**说明**：`Executors` 返回的线程池对象的弊端如下：

- `FixedThreadPool` 和 `SingleThreadPool`

  允许的请求队列长度为 `Integr.MAX_VALUE`，可能会堆积大量的请求，从而导致 `OOM`（Out Of Memory）。

- `CachedThreadPool` 和 `ScheduledThreadPool`

  允许的创建线程数量为 `Integr.MAX_VALUE`，可能会创建大量的线程，从而导致 `OOM`。

<font size=4 style="font-weight:bold;background:yellow;">Demo</font>

{% spoiler 展开查看折叠代码 %}

```java
public class Demo01 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();     // 单个线程
//        ExecutorService threadPool = Executors.newFixedThreadPool(5);   // 创建一个固定大小的线程池
        ExecutorService threadPool = Executors.newCachedThreadPool();   // 大小可伸缩的线程池
        
        try {
            for (int i = 0; i < 9; i++) {
                // 使用线程池创建线程
                threadPool.execute(() -> {
                    // 可采用不同的线程池观察线程的执行情况
                    System.out.println(Thread.currentThread().getName() + " OK");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();	// 程序结束，线程池用完，要关闭
        }
    }
}
```

{% endspoiler %}

# 3、七大参数

通过观察可知，以上三个方法的底层都是由 `ThreadPoolExecutor` 创建线程池的，查看构造方法发现有七个参数，这就是七大参数。

```java
public ThreadPoolExecutor(int corePoolSize,		// 核心线程数
                          int maximumPoolSize,	// 最大线程数
                          long keepAliveTime,	// 超时时间，超时了没有线程调用就会释放
                          TimeUnit unit,		// 超时时间单位
                          BlockingQueue<Runnable> workQueue,	// 阻塞队列
                          ThreadFactory threadFactory,			// 线程工厂，创建线程
                          RejectedExecutionHandler handler) {	// 拒绝策略
    ...
}
```

可以用去银行柜台办理业务对 `ThreadPoolExecutor` 七大参数进行解释说明：

- `int corePoolSize`：银行的部分窗口数量，一直开放。（核心线程池大小）
- `int maximumPoolSize`：银行的全部窗口数量，不会一直开放；候客区满了，又新来了客人，银行装不下了才会开辟新窗口办理业务。（最大线程池大小）
- `long keepAliveTime`：如果一段时间内新开辟的窗口一直空闲，就会关闭。（超时了没有线程调用就会释放）
- `TimeUnit unit`：超时单位
- `BlockingQueue<Runnable> workQueue` ：候客区，银行窗口有人，新来的客人会在候客区等候。（阻塞队列）
- `ThreadFactory threadFactory`：线程工厂，创建线程
- `RejectedExecutionHandler handler`：银行所有窗口、候客区都满了，又新来了客人，对新客人的处理的方式：等待/离开。（拒绝策略）

<img src="https://haining820-bucket.oss-cn-beijing.aliyuncs.com/typora_img/线程池参数示意.png" style="zoom: 67%;" />

<font size=4 style="font-weight:bold;background:yellow;">手动创建一个线程池</font>

```java
// 自定义线程池，最大承载量：queue+max=3+5=8
ExecutorService threadPool = 
    new ThreadPoolExecutor(2,	// corePoolSize
                           5,	// maximumPoolSize
                           3,	// keepAliveTime
                           TimeUnit.SECONDS,
                           new LinkedBlockingQueue<>(3),
                           Executors.defaultThreadFactory(),	// 默认的线程工厂
                           new ThreadPoolExecutor.DiscardOldestPolicy());// 拒绝策略
```

<font size=4 style="font-weight:bold;background:yellow;">最大线程数 maximumPoolSize 该如何设置？</font>

CPU 密集型和 IO 密集型

- CPU 密集型：使用电脑的核数作为线程数，可以保持 CPU 效率最高。

  使用以下代码可以获取当前电脑的 CPU 核数，在设置线程池参数时将本机 CPU 的核数直接设置为线程池的最大线程数，可以避免因服务器型号不同而产生的性能损失。

  ```java
  System.out.println(Runtime.getRuntime().availableProcessors());	// 获取CPU核数
  ```

- IO 密集型：判断程序中十分消耗 IO 的线程有多少个，设置的最大线程数要大于这个数量。

  比如：一个程序中15个大型任务，IO 十分占用资源，要设置出额外的线程数量供其他线程使用。

# 4、四种拒绝策略

```java
ThreadPoolExecutor.AbortPolicy()         // 银行（窗口、候客区）满了，还有人进来，不处理这个人，抛出异常
ThreadPoolExecutor.CallerRunsPolicy()    // 哪来的去哪里
ThreadPoolExecutor.DiscardPolicy()       // 队列满了，丢掉任务，不会抛出异常
ThreadPoolExecutor.DiscardOldestPolicy() // 队列满了，会将最早进入队列的任务删除，然后将其加入队列
```

这里的线程池延续之前创建的线程池的配置，<font color='red' style="font-weight:bold;">核心线程池大小为 2，最大线程池的大小为 5，阻塞队列长度为 3，</font>线程的执行内容和运行结果如下：

```java
try {
    // n 为线程数，一个线程池的最大承载量：queue+max
    for (int i = 1; i <= 9; i++) {
        threadPool.execute(() -> {	// 使用线程池创建线程
            System.out.println(Thread.currentThread().getName() + " OK");
        });
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    threadPool.shutdown();	// 程序结束，线程池用完，要关闭
}
```

- 对于线程数 `n`，当 `n <= 2` 时，线程池核心数够用，可以直接同步运行；

  ```java
  pool-1-thread-2 OK
  pool-1-thread-1 OK	// n=2
  ```

- 当 `3 <= n <= 5` 时，线程池核心数只有 2，新开启的线程会在阻塞队列中排队等候，仍由核心线程池中线程依次运行；

  ```java
  pool-1-thread-2 OK
  pool-1-thread-1 OK
  pool-1-thread-2 OK
  pool-1-thread-1 OK
  pool-1-thread-2 OK	// n=5 
  ```

- 当 `6 <= n <= 8` 时，此时线程较多，线程池核心以及阻塞队列都满了，就会开启额外的线来缓解压力；

  ```java
  pool-1-thread-2 OK
  ...
  pool-1-thread-5 OK	// 3/4/5 都是额外的线程
  pool-1-thread-1 OK
  pool-1-thread-2 OK
  pool-1-thread-3 OK	// n=8
  ```
  
- 当 `n >= 9` 时，**一个线程池的最大承载量就是 `maximumPoolSize + queueSize`** ，根据之前线程池的配置，该线程池的最大容量为 `5 + 3 = 8`，超过了 8 之后，线程池就需要拒绝策略来处理新开启的线程了。

  - 当拒绝策略为 `AbortPolicy()` 时，<font color='red' style="font-weight:bold;">新开启的线程会抛出异常</font>；

    ```java
    pool-1-thread-1 OK
    ...
    pool-1-thread-1 OK
    java.util.concurrent.RejectedExecutionException...	// n=9,前8个线程正常运行,第九个会出现异常
    ```
    
  - 当拒绝策略为 `CallerRunsPolicy()` 时，<font color='red' style="font-weight:bold;">新开启的线程会被返回至原线程</font>；
  
    ```java
    pool-1-thread-1 OK
  pool-1-thread-4 OK
    main OK				// 被返回至原线程
  pool-1-thread-3 OK
    ...
    pool-1-thread-1 OK	// n=9
    ```
    
  - 当拒绝策略为 `DiscardPolicy()` 时，<font color='red' style="font-weight:bold;">队列满了任务会被直接抛弃，不会抛出异常</font>；
  
  - 当拒绝策略为 `DiscardOldestPolicy()` 时，<font color='red' style="font-weight:bold;">会尝试和阻塞队列中最早的任务竞争，抛弃该任务，再将新任务加入队列，也不会抛出异常</font>。
  

