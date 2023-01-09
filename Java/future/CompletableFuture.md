---
title: CompletableFuture学习
date: 2022-08-14
tags: [异步,future]
toc: true
---


# 0、准备

ListenableFuture 是由 Google Guava工具包提供的 Future 扩展类，随后，JDK 在 1.8 版本中马上也提供了类似这样的类，在 9 中进一步增强，这就是 CompletableFuture。

<!--more-->

CompletableFuture 同样实现了 Future 接口，并在此基础上进行了丰富的扩展，弥补了 Future 的局限性，是对 Future 的扩展和增强，同时 CompletableFuture 实现了对任务编排的能力。在进行方法的测试前，首先准备打印当前时间和线程名的工具类：

```java
public class SmallTool {
    public static void sleepMillis(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static void printTimeAndThread(String tag) {
        String result = new StringJoiner("\t|\t")
                .add(String.valueOf(System.currentTimeMillis()))
                .add(String.valueOf(Thread.currentThread().getId()))
                .add(Thread.currentThread().getName())
                .add(tag)
                .toString();
        System.out.println(result);
    }
}
```

# 1、get/join

<font size=4 style="font-weight:bold;background:yellow;">get</font>

get 方法有两种类型的重载，继承自父接口 Future 中两种方法，详情可见 Future。

```java
// 获得异步执行结果，如果任务一直不返回会一直阻塞，一般不建议使用
public T get()
// 获得异步执行结果，指定超时时间，超时抛出异常
public T get(long timeout, TimeUnit unit)
```

```java
// 这种写法主线程会一直阻塞，因为这种方式创建的 future 从未完成，
// 可以打个断点看看，状态一直是 not completed。
CompletableFuture<Integer> future = new CompletableFuture<>();
Integer integer = future.get();
```

<font size=4 style="font-weight:bold;background:yellow;">getNow</font>

获取任务的执行结果，但不会产生阻塞。如果任务还没执行完成，那么就会返回传入的 valueIfAbsent 参数值，如果执行完成了，就会返回任务执行的结果。

```java
public T getNow(T valueIfAbsent)
```

<font size=4 style="font-weight:bold;background:yellow;">join</font>

- 返回的类型就是 CompletableFuture 的泛型，即 supplier 的返回值；
- 实际上就是 Future 中 `get()` 的升级版，等待任务结束，返回任务的结果；
- **区别：**get() 会抛出检查时的异常，可被捕获进行处理或直接抛出，join() 会抛出未经检查的异常。

```java
public T join()
```

<font size=4 style="font-weight:bold;background:yellow;">demo</font> ：[github](https://github.com/haining820/study-projects/blob/main/future-study/src/main/java/com/haining820/futureapi/_00_getjoin.java)

{% spoiler 展开查看折叠代码 %}

```java
public class _00_getjoin {

    public void getTest() throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = new CompletableFuture<>();
        Integer integer = future.get();
        int i = 3;
        // future中没有内容，会一直阻塞，不会输出结果
        System.out.println(i);
    }

    public void timeoutGetTest() throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = new CompletableFuture<>();
        try {
            // 超过规定的5s就会抛出超时异常
            Integer integer = future.get(5, TimeUnit.SECONDS);
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        int i = 3;
        System.out.println(i);
    }

    public void getNowTest() {
        CompletableFuture<Integer> cf = new CompletableFuture<>();
        Integer i = cf.getNow(666);
        // cf没有结果，输出默认结果666
        System.out.println("cf->" + i);

        CompletableFuture<Integer> cf2 = new CompletableFuture<>();
        cf2.complete(777);
        Integer i2 = cf2.getNow(666);
        // cf有结果，直接输出结果，输出777
        System.out.println("cf2->" + i2);
    }
}
```

{% endspoiler %}



