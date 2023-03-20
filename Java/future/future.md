---
title: Future学习
date: 2022-08-14
tags: [异步,future]
toc: true
---

# 0、同步和异步

> Callable 和 Future 在 JDK1.5 之后开始提供

- 同步：同步是指一个进程在执行某个请求的时候，如果该请求需要一段时间才能返回信息，那么这个进程会一直等待下去，直到收到返回信息才继续执行下去。
- 异步：异步是指进程不需要一直等待下去，而是继续执行下面的操作，不管其他进程的状态，当有信息返回的时候会通知进程进行处理，这样可以提高执行效率。

**Future 模式的核心思想是能够让主线程将原来需要同步等待的这段时间用来做其他的事情，因为可以异步获得执行结果，所以不用一直同步等待去获得执行结果。**

<!--more-->

# 1、Future 接口

Future 接口中的方法如下：

```java
public interface Future<V> {
    // 取消
    boolean cancel(boolean mayInterruptIfRunning);
    // 是否取消
    boolean isCancelled();
    // 是否执行完毕
    boolean isDone();
    // 获取返回结果
    V get() throws InterruptedException, ExecutionException;
    // 超时获取
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

**① cancel：mayInterruptRunning 表示是否中断执行中的线程。**

- 如果任务还没开始，执行 `cancel(...)` 方法将返回 false；
- 如果任务已经启动：
  - 执行 `cancel(true)` 方法将以中断执行此任务线程的方式来试图停止任务，如果停止成功，返回 true；
  - 执行 `cancel(false)` 方法将不会对正在执行的任务线程产生影响（让线程正常执行到完成），此时返回 false；
- 当任务已经完成，执行 `cancel(...)` 方法将返回 false。

**② isCancelled：如果任务完成前被取消，则返回 true；**

**③ isDone：如果任务执行结束，无论是正常结束或是中途取消还是发生异常，都返回 true；**

**④ get：有两种类型的重载**

- 没有时间参数：获取异步执行的结果，如果没有结果可用，此方法会阻塞直到异步任务完成，一般不建议使用；

- 有时间参数：获取异步执行结果，如果没有结果可用，此方法会阻塞，但是会有时间限制，如果阻塞时间超过设定的时间将抛出 TimeoutException 异常。

# 2、FutureTask

常见的直接继承 Thread 和实现 Runnable 接口实现多线程这两种方式在执行之后都无法获取返回结果，Callable 和 Future 在 JDK1.5 之后开始提供，通过他们在任务执行完毕之后也可以获得结果。

- Callable 接口有返回值，但是 Thread 只能接受没有返回值的 Runnable 接口；
- RunnableFuture 接口继承了 Runnable 接口和 Future 接口，所以 RunnableFuture 接口既可以作为 Runnable 被线程执行，又可以作为 Future 得到 Callable 的返回值；
- FutureTask 实现了 RunnableFuture 接口，使用 FutureTask 获取结果：把 Callable 实例当作参数，生成 FutureTask 的对象，然后把这个对象当作一个 Runnable 对象，用线程池或另起线程去执行这个 Runnable 对象，最后通过 FutureTask 获取刚才执行的结果。

<img src="https://haining820-bucket.oss-cn-beijing.aliyuncs.com/typora_img/image-20220912142952407.png" alt="image-20220912142952407" style="zoom: 50%;" />

<font size=4 style="font-weight:bold;background:yellow;">Future 与 Callable 之间的关系</font>

- `Future.get()` 获取 Callable 接口返回的执行结果，还可以通过 `Future.isDone()` 来判断任务是否已经执行完了，以及取消这个任务，限时获取任务的结果等；
- 在 `call()` 未执行完毕之前，调用 `get()` 的线程（假定是主线程）直到 `call()` 方法返回了结果后，此时 `Future.get()` 才会得到该结果，然后主线程才会切换到 runnable 状态；
- 所以 Future 是一个存储器，它存储了 `call()` 这个任务的结果，然而这个任务的执行时间是无法提前确定的，因为这完全取决于 `call()` 方法执行的情况。

## 2.1、FutureTask 源码

Future 只是一个接口，无法直接创建对象，FutureTask 实现 RunnableFuture 接口，是 Future 的一个实现类。

<font size=4 style="font-weight:bold;background:yellow;">state 参数</font>

FutureTask 源码中定义了以下几个变量：

```java
private volatile int state;
private static final int NEW          = 0;	// FutureTask新建出来的初始状态
private static final int COMPLETING   = 1;	// 正在执行
private static final int NORMAL       = 2;	// 正常结束
private static final int EXCEPTIONAL  = 3;	// 执行过程出现异常
// 任务还未执行之前就调用了cancel(true)方法，任务处于CANCELLED
private static final int CANCELLED    = 4;
// 当任务调用cancel(true)中断程序时，任务处于INTERRUPTING状态，这是一个中间状态
private static final int INTERRUPTING = 5;
// 任务调用cancel(true)中断程序时会调用interrupt()方法中断线程运行，任务状态由INTERRUPTING转变为INTERRUPTED
private static final int INTERRUPTED  = 6;
```

对应的，在一个 FutureTask 对象新建出来后可能会有以下几种情况：

```java
NEW -> COMPLETING -> NORMAL				// 执行过程顺利完成
NEW -> COMPLETING -> EXCEPTIONAL		// 执行过程出现异常
NEW -> CANCELLED						// 执行过程被取消
NEW -> INTERRUPTING -> INTERRUPTED		// 执行过程中，线程中断
```

<font size=4 style="font-weight:bold;background:yellow;">初始化</font>

```java
private Callable<V> callable;
// 在使用Callable创建FutureTask时会将其传递进来并设置state为NEW
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

<font size=4 style="font-weight:bold;background:yellow;">run() 方法</font>

```java
public void run() {
    // 判断是否是新线程，不是新线程自动return
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
                // FutureTask的run()实际上调用的还是Callable接口
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            // 运行成功，设置返回结果
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
        // 如果线程被打断，会yield
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

<font size=4 style="font-weight:bold;background:yellow;">set 方法</font>

FutureTask 执行成功后调用，修改线程状态，同时设置返回值 outcome。

```java
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

<font size=4 style="font-weight:bold;background:yellow;">get 方法</font>

可以看到当状态小于等于 `COMPLETING` 时（`NEW`、`COMPLETING`）会阻塞，调用 `awaitDone()` 一直等到状态改变。

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

最后返回 `report()`的调用结果，如果线程状态不是正常结束的话就会抛出异常。

```java
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```
