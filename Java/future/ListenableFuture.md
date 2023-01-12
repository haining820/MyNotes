---
title: ListenableFuture学习
date: 2022-08-14
tags: [异步,future]
toc: true
---

ListenableFuture 是 Guava 工具包提供的 Future 扩展类，是可以监听的 Future，是对 Java 原生Future 的扩展增强。

老版 Future 模式一个最大的问题是需要获取结果做后续处理操作的时候，还是需要阻塞等待。这样的话和同步调用方式就没有多大区别了。而 ListenableFuture 和 CompletableFuture 对于这种情况提供了很多易用的 API。

**如果使用 ListenableFuture，Guava 会帮助检测 Future 是否完成了，如果完成就自动调用回调函数，这样可以减少并发程序的复杂度。**

ListenableFuture 接口扩展了 Future 接口，增加了 addListener 方法，该方法在给定的 excutor 上注册一个监听器，当计算完成时会马上调用该监听器。虽然不能够确保监听器执行的顺序，但可以确保在计算完成时马上被调用。

<img src="https://haining820-bucket.oss-cn-beijing.aliyuncs.com/typora_img/image-20230109195031470.png" alt="image-20230109195031470" style="zoom:67%;" />

# 1、ListenableFuture 的初始化

`addListener()`：注册回调函数，在实际使用时可能不会直接使用这个接口，因为这个接口只能传 Runnable 实例，Futures 类提供了另一个 addCallback 方法。

```java
public interface ListenableFuture<V> extends Future<V> {
    void addListener(Runnable listener, Executor executor);
}
```

```java
class Task implements Callable<String> {
    @Override
    public String call() throws Exception {
        TimeUnit.SECONDS.sleep(1);
//        int a = 1 / 0;	// 异常
        return "task done";
    }
}
```

```java
 public void lisFuture() throws ExecutionException, InterruptedException {
     ListeningExecutorService pool = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(5));
     ListenableFuture<String> lf = pool.submit(new Task());
     System.out.println(lf.get());
     lf.addListener(() -> {
         System.out.println("finish lf");
     }, Executors.newFixedThreadPool(5));
 }
```

<font size=4 style="font-weight:bold;background:yellow;">Demo</font>

```java
public static void testListenFuture() {
    System.out.println("主任务执行完,开始异步执行副任务1.....");
    ListeningExecutorService pool = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(5));
    ListenableFuture<String> future = pool.submit(new Task());
    // 为任务绑定回调接口
    Futures.addCallback(future, new FutureCallback<String>() {
        @Override
        public void onSuccess(String result) {
            System.out.println("success,result->:" + result);
        }

        @Override
        public void onFailure(Throwable t) {
            System.out.println("error->" + t);
        }
    });
    System.out.println("副本任务启动,回归主任务线,主业务正常返回.....");
}
```

**MoreExecutors：**该类是 final 类型的工具类，提供了很多静态方法。

- 使用 `MoreExecutors.listeningDecorator(ExecutorService delegate)` 方法初始化 `ListeningExecutorService`；

**ListeningExecutorService：**该类是对 ExecutorService 的扩展，重写 ExecutorService 类中的 submit 方法，返回 ListenableFuture 对象。

- 使用 `ListeningExecutorService` 的 `submit(Callable<T> var1)` 方法即可初始化 `ListenableFuture` 对象。

**Futures：**该类提供和很多实用的静态方法以供使用。(todo: successfulAsList,AllAsList)

`addCallback()`：在线程结束时，立即调用 `FutureCallBack()`。

{% spoiler 展开查看折叠代码 %}

```java
// addCallback源码
public static <V> void addCallback(ListenableFuture<V> future, 
                                   FutureCallback<? super V> callback) {
    addCallback(future, callback, MoreExecutors.directExecutor());
}

public static <V> void addCallback(final ListenableFuture<V> future, 
                                   final FutureCallback<? super V> callback, Executor executor) {
    Preconditions.checkNotNull(callback);
    Runnable callbackListener = new Runnable() {
        public void run() {
            Object value;
            try {
                value = Futures.getDone(future);
            } catch (ExecutionException var3) {
                callback.onFailure(var3.getCause());
                return;
            } catch (RuntimeException var4) {
                callback.onFailure(var4);
                return;
            } catch (Error var5) {
                callback.onFailure(var5);
                return;
            }

            callback.onSuccess(value);
        }
    };
    future.addListener(callbackListener, executor);
}
```

{% endspoiler %}

**FutureCallback：**该接口提供了 `onSuccess()` 和 `onFailure()`，获取异步计算的结果并回调，ListenableFuture  可以使用以下方式通过重写 `onSuccess()` 和 `onFailure()` 增加回调函数：

```java
@GwtCompatible
public interface FutureCallback<V> {
    void onSuccess(@Nullable V var1);	// 当future执行成功，执行此方法
    void onFailure(Throwable var1);		// 当 future 执行失败，执行此方法
}
```





