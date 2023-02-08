---
title: CompletableFuture学习
date: 2022-08-14
tags: [异步,future]
toc: true
---

```java
感谢
https://www.zhihu.com/question/318472639/answer/652254785
https://zhuanlan.zhihu.com/p/344431341
https://www.bilibili.com/video/BV1ui4y1T7xf
```

# 0、准备

ListenableFuture 是由 Google Guava工具包提供的 Future 扩展类，随后，JDK 在 1.8 版本中马上也提供了类似这样的类，在 9 中进一步增强，这就是 CompletableFuture。CompletableFuture 同样实现了 Future 接口，并在此基础上进行了丰富的扩展，弥补了 Future 的局限性，是对 Future 的扩展和增强，同时 CompletableFuture 实现了对任务编排的能力。

<!--more-->

# 1、get/join

## 1.1、get

get 有两种类型的重载，继承自父接口 Future 中两种方法，详情可见 Future 接口相关内容。

- get 无参数：获得异步执行结果，如果任务一直不返回会一直阻塞，一般不建议使用

```java
public T get()
```

```java
// 这种写法主线程会一直阻塞，因为这种方式创建的 future 从未完成，
// 可以打个断点看看，状态一直是 not completed。
CompletableFuture<Integer> future = new CompletableFuture<>();
Integer integer = future.get();
```

- get 有参数：获得异步执行结果，指定超时时间，超时抛出异常

```java
public T get(long timeout, TimeUnit unit)
```

<font size=4 style="font-weight:bold;background:yellow;">demo</font>（[github](https://github.com/haining820/study-projects/blob/main/future-study/src/main/java/com/haining820/futureapi/_00_getjoin.java)）

{% spoiler 展开查看折叠代码 %}

```java
    @Test
    public void getTest() throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = new CompletableFuture<>();
        Integer integer = future.get();
        // future中没有内容，会一直阻塞，不会输出结果
        log.info("1");
    }

    @Test
    public void timeoutGetTest() throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = new CompletableFuture<>();
        try {
            // 超过规定的5s就会抛出超时异常
            Integer integer = future.get(5, TimeUnit.SECONDS);
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        log.info("3");
    }
```

{% endspoiler %}

## 1.2、getNow

获取任务的执行结果，但不会产生阻塞。如果任务还没执行完成，那么就会返回传入的 valueIfAbsent 参数值，如果执行完成了，就会返回任务执行的结果。

```java
public T getNow(T valueIfAbsent)
```

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

{% spoiler 展开查看折叠代码 %}	

```java
    @Test
    public void getNowTest() {
        CompletableFuture<Integer> cf1 = new CompletableFuture<>();
        // cf没有结果，打印 666
        log.info("cf getNow res#={}", cf1.getNow(666));
        CompletableFuture<Integer> cf2 = new CompletableFuture<>();
        cf2.complete(777);
        // cf有结果，直接输出结果，打印 777
        log.info("cf get res#={}", cf2.getNow(666));
    }

    @Test
    public void getNowTest2() {
        CompletableFuture<Integer> getNowCf = CompletableFuture.supplyAsync(() -> sleepMillis(1000));
        // sleepMillis操作需要等待，没有结果立即返回，打印 666
        log.info("cf getNow res#={}", getNowCf.getNow(666));
        CompletableFuture<Integer> getCf = CompletableFuture.supplyAsync(() -> sleepMillis(1000));
        try {
            // 等待sleepMillis结果完成，打印 1
            log.info("cf get res#={}", getCf.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public Integer sleepMillis(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return 1;
    }
```

{% endspoiler %}

## 1.3、join

- 返回的类型就是 CompletableFuture 的泛型，即 supplier 的返回值；
- 实际上就是 Future 中 `get()` 的升级版，等待任务结束，返回任务的结果；
- **区别：**get() 会抛出检查时的异常，可被捕获进行处理或直接抛出，join() 会抛出未经检查的异常。

```java
public T join()
```

# 2、supplyAsync/runAsync

> 在学习异步任务的编排前，首先准备打印当前时间和线程名的工具类：
>
> ```java
> public class SmallTool {
>     public static void sleepMillis(long millis) {
>         try {
>             Thread.sleep(millis);
>         } catch (InterruptedException e) {
>             e.printStackTrace();
>         }
>     }
>     public static void printTimeAndThread(String tag) {
>         SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
>         String result = new StringJoiner("\t|\t")
>                 .add(sdf.format(new Date()))
>                 .add(String.valueOf(Thread.currentThread().getId()))
>                 .add(Thread.currentThread().getName())
>                 .add(tag).toString();
>         System.out.println(result);
>     }
> }
> ```

**功能：**<font color='#0000ff' style="font-weight:bold;">执行异步任务，可以初始化 cf。</font>

**区别：**supplyAsync 用于有返回值的任务，runAsync 用于没有返回值的任务，不需要返回值时使用 runAsync 即可。

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
```

**参数：**

- 供应者 supplier，是一个函数式接口；（详见[函数式接口](https://haining820.github.io/2022/10/04/mynotes/Java/functional-interface/)）

- supplyAsync 后括号中的 supplier 会去另一个线程中执行。

- Executor 参数可以手动指定线程池，否则默认使用 `ForkJoinPool.commonPool()` 系统级公共线程池

<font color='red' style="font-weight:bold;">注意：这些线程都是 Daemon 线程，主线程结束 Daemon 线程不结束，只有 JVM 关闭时，生命周期才终止。</font>

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

可以看到，在小白点餐后，厨师的操作和小白后续的操作几乎是同时进行的。

{% spoiler 展开查看折叠代码 %}

```java
    /**
     * supplyAsync 用于有返回值的任务
     */
    @Test
    public void supplyAsyncTest(){
        SmallTool.printTimeAndThread("小白进入餐厅");
        SmallTool.printTimeAndThread("小白点了 番茄炒蛋 + 一碗米饭");

        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("厨师炒菜");
            SmallTool.sleepMillis(200);
            SmallTool.printTimeAndThread("厨师打饭");
            SmallTool.sleepMillis(100);
            return "番茄炒蛋 + 米饭 做好了";
        });
        SmallTool.printTimeAndThread("小白在打王者");
        SmallTool.printTimeAndThread(String.format("%s ,小白开吃", cf.join()));
    }

    @Test
    public void runAsyncTest(){
        SmallTool.printTimeAndThread("小白进入餐厅");
        SmallTool.printTimeAndThread("小白点了 番茄炒蛋 + 一碗米饭");

        // runAsync无返回值
        CompletableFuture cf = CompletableFuture.runAsync(() -> {
            SmallTool.printTimeAndThread("厨师炒菜");
            SmallTool.sleepMillis(200);
            SmallTool.printTimeAndThread("厨师打饭");
            SmallTool.sleepMillis(100);
            SmallTool.printTimeAndThread("番茄炒蛋 + 米饭 做好了");
        });
        SmallTool.printTimeAndThread("小白在打王者");
        // cf.join()获取不到值，因为runAsync没有返回结果
        SmallTool.printTimeAndThread(String.format("%s ,小白开吃", cf.join()));
    }
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">res</font>

{% spoiler 展开查看折叠代码 %}

```java
// supplyAsyncTest
2023-01-15 21:31:22.878	|	1	|	main	|	小白进入餐厅
2023-01-15 21:31:22.879	|	1	|	main	|	小白点了 番茄炒蛋 + 一碗米饭
2023-01-15 21:31:22.916	|	1	|	main	|	小白在打王者
2023-01-15 21:31:22.917	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师炒菜
2023-01-15 21:31:23.122	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师打饭
2023-01-15 21:31:23.232	|	1	|	main	|	番茄炒蛋 + 米饭 做好了 ,小白开吃

// runAsyncTest
2023-01-15 21:31:40.807	|	1	|	main	|	小白进入餐厅
2023-01-15 21:31:40.808	|	1	|	main	|	小白点了 番茄炒蛋 + 一碗米饭
2023-01-15 21:31:40.842	|	1	|	main	|	小白在打王者
2023-01-15 21:31:40.842	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师炒菜
2023-01-15 21:31:41.042	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师打饭
2023-01-15 21:31:41.153	|	20	|	ForkJoinPool.commonPool-worker-9	|	番茄炒蛋 + 米饭 做好了
2023-01-15 21:31:41.153	|	1	|	main	|	null ,小白开吃
```

{% endspoiler %}

## 4.2、thenCompose

**thenCompose：**<font color='#0000ff' style="font-weight:bold;">连接，在异步执行线程1的基础上再开一个新的线程，使用 thenCompose 会将之前线程的结果交给下一个的线程，前一个线程运行完了有结果后下一个线程才能触发。</font>

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn) {
    return uniComposeStage(null, fn);
}
```

**参数：**

- Function：函数式接口，传入T，返回 R；
- CompletionStage：CompletableFuture 的父接口，使用 CompletableFuture 替代即可。

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

在下面例子中入参 dish 是厨师任务的返回值，只有厨师炒菜完成后服务员才会开始打饭，然后进入第二个 CompletableFuture，再开启一个线程运行后返回，观察运行结果也可知，炒菜和打饭不在同一个线程上运行。

```java
public class ThenComposeTest {
    public static void main(String[] args) {
        SmallTool.printTimeAndThread("小白进入餐厅");
        SmallTool.printTimeAndThread("小白点了 番茄炒蛋 + 一碗米饭");
        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("厨师炒菜");
            SmallTool.sleepMillis(200);
            return "番茄炒蛋";
        }).thenCompose(dish -> CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("服务员打饭");
            SmallTool.sleepMillis(100);
            return dish + " + 米饭";
        }));
        SmallTool.printTimeAndThread("小白在打王者");
        SmallTool.printTimeAndThread(String.format("%s 好了,小白开吃", cf1.join()));
    }
}
```

```
1658369953335	|	1	|	main	|	小白进入餐厅
1658369953336	|	1	|	main	|	小白点了 番茄炒蛋 + 一碗米饭
1658369953390	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师炒菜
1658369953390	|	1	|	main	|	小白在打王者
1658369953593	|	21	|	ForkJoinPool.commonPool-worker-2	|	服务员打饭
1658369953722	|	1	|	main	|	番茄炒蛋 + 米饭 好了,小白开吃
```

## 4.3、thenCombine

**thenCombine：**<font color='#0000ff' style="font-weight:bold;">合并，将任务1和任务2一起执行，然后将任务1和任务2的返回结果用作任务3的输入。</font>

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

demo 中 dish 和 rice 作为线程 3 的输入，只有等线程 1 和线程 2 运行完毕后线程 3 才能开始运行，观察运行结果可知，线程 2 和线程 3 使用的使用同一个线程。

{% spoiler 展开查看折叠代码 %}

```java
    @Test
    public void thenCombineTest() {
        SmallTool.printTimeAndThread("小白进入餐厅");
        SmallTool.printTimeAndThread("小白点了 番茄炒蛋 + 一碗米饭");

        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("厨师炒菜");
            SmallTool.sleepMillis(200);
            return "番茄炒蛋";
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("服务员蒸饭");
            SmallTool.sleepMillis(300);
            return "米饭";
        }), (dish, rice) -> {
            SmallTool.printTimeAndThread("服务员打饭");
            SmallTool.sleepMillis(100);
            return String.format("%s + %s 好了", dish, rice);
        });
        SmallTool.printTimeAndThread("小白在打王者");
        SmallTool.printTimeAndThread(String.format("%s ,小白开吃", cf.join()));
    }
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">res</font>

{% spoiler 展开查看折叠代码 %}

```
2023-01-15 21:40:46.514	|	1	|	main	|	小白进入餐厅
2023-01-15 21:40:46.515	|	1	|	main	|	小白点了 番茄炒蛋 + 一碗米饭
2023-01-15 21:40:46.558	|	20	|	ForkJoinPool.commonPool-worker-9	|	厨师炒菜
2023-01-15 21:40:46.558	|	21	|	ForkJoinPool.commonPool-worker-2	|	服务员蒸饭
2023-01-15 21:40:46.558	|	1	|	main	|	小白在打王者
2023-01-15 21:40:46.869	|	21	|	ForkJoinPool.commonPool-worker-2	|	服务员打饭
2023-01-15 21:40:46.979	|	1	|	main	|	番茄炒蛋 + 米饭 好了 ,小白开吃
```

{% endspoiler %}

### thenAcceptBoth/runAfterBoth

**thenAcceptBoth/runAfterBoth 与 thenCombine相比的区别？**

- thenCombine：需要前边任务的结果，有返回值；

- thenAcceptBoth：得到前边两个任务的参数之后直接内部消化，没有返回值；

- runAfterBoth：不需要前边任务的结果也不需要返回值。

## 4.4、thenApply/thenApplyAsync

**thenApply：**<font color='#0000ff' style="font-weight:bold;">将前面异步任务的结果交给后面的 Function，不会处理异常，异常会被直接抛出，交给上一层处理。 </font>

- thenApply 使用同一线程运行前后两个任务；
- thenApplyAsync 使用不同的线程运行前后两个任务，类似之前的 thenCompose。

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

{% spoiler 展开查看折叠代码 %}

```java
public class _04_thenApply {
    /**
     * 替换cf默认线程池
     * 创建一个keepAliveTime为0的线程池，线程处理完任务处于空闲状态时，存活时间为0，立即消失
     */
    public ExecutorService verifyPool = new ThreadPoolExecutor(1, Integer.MAX_VALUE,
            0, TimeUnit.MILLISECONDS,
            new SynchronousQueue<>());

    public void thenApplyTest() {
        SmallTool.printTimeAndThread("小白吃好了");
        SmallTool.printTimeAndThread("小白 结账、要求开发票");

        CompletableFuture<String> invoice = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("服务员收款 500元");
            SmallTool.sleepMillis(100);
            return "500";
            // 使用thenApply任务会在同一线程中执行
        }, verifyPool).thenApply(money -> {
            SmallTool.printTimeAndThread(String.format("服务员开发票 面额 %s元", money));
            SmallTool.sleepMillis(200);
            return String.format("%s元发票", money);
        });

        SmallTool.printTimeAndThread("小白 接到朋友的电话，想一起打游戏");
        SmallTool.printTimeAndThread(String.format("小白拿到%s，准备回家", invoice.join()));
    }

    public void thenApplyAsyncTest() {
        SmallTool.printTimeAndThread("小白吃好了");
        SmallTool.printTimeAndThread("小白 结账、要求开发票");

        CompletableFuture<String> invoice = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("服务员收款 500元");
            SmallTool.sleepMillis(100);
            return "500";
            // 使用thenApplyAsync任务会在不同线程中执行
        }, verifyPool).thenApplyAsync(money -> {
            SmallTool.printTimeAndThread(String.format("服务员开发票 面额 %s元", money));
            SmallTool.sleepMillis(200);
            return String.format("%s元发票", money);
        }, verifyPool);

        SmallTool.printTimeAndThread("小白 接到朋友的电话，想一起打游戏");
        SmallTool.printTimeAndThread(String.format("小白拿到%s，准备回家", invoice.join()));
    }

}
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">res</font>

**thenApply：**会在相同的线程中执行

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:46:42.436	|	1	|	main	|	小白吃好了
2023-01-15 21:46:42.437	|	1	|	main	|	小白 结账、要求开发票
2023-01-15 21:46:42.476	|	20	|	pool-1-thread-1	|	服务员收款 500元
2023-01-15 21:46:42.476	|	1	|	main	|	小白 接到朋友的电话，想一起打游戏
2023-01-15 21:46:42.589	|	20	|	pool-1-thread-1	|	服务员开发票 面额 500元
2023-01-15 21:46:42.796	|	1	|	main	|	小白拿到500元发票，准备回家
```

{% endspoiler %}

**thenApplyAsync：**会在不同的线程中执行

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:46:25.975	|	1	|	main	|	小白吃好了
2023-01-15 21:46:25.976	|	1	|	main	|	小白 结账、要求开发票
2023-01-15 21:46:26.014	|	20	|	pool-1-thread-1	|	服务员收款 500元
2023-01-15 21:46:26.014	|	1	|	main	|	小白 接到朋友的电话，想一起打游戏
2023-01-15 21:46:26.118	|	21	|	pool-1-thread-2	|	服务员开发票 面额 500元
2023-01-15 21:46:26.322	|	1	|	main	|	小白拿到500元发票，准备回家
```

{% endspoiler %}

### thenAccept/thenRun

**thenAccept/thenRun 与 thenApply 相比的区别？**

- thenApply 接收前边任务的参数，有返回值；

- thenAccept 会接收前边任务的参数，但是没有返回值；
- thenRun 不接收前边任务的参数，也没有返回值；

## 4.5、applyToEither

**applyToEither：**<font color='#0000ff' style="font-weight:bold;">两个任务一起运行，谁先运行完返回谁。</font>

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

demo 中 800 路的线程运行时间更快一些，会提前完成。

{% spoiler 展开查看折叠代码 %}

```java
    @Test
    public void applyToEitherTest() {
        SmallTool.printTimeAndThread("张三走出餐厅，来到公交站");
        SmallTool.printTimeAndThread("等待 700路 或者 800路 公交到来");
        CompletableFuture<String> bus = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("700路公交正在赶来");
            SmallTool.sleepMillis(900);
            return "700路到了";
        }).applyToEither(CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("800路公交正在赶来");
            SmallTool.sleepMillis(200);
            return "800路到了";
        }), firstComeBus -> firstComeBus);
        SmallTool.printTimeAndThread(String.format("%s,小白坐车回家", bus.join()));
    }
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">res</font>

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:51:11.519	|	1	|	main	|	张三走出餐厅，来到公交站
2023-01-15 21:51:11.519	|	1	|	main	|	等待 700路 或者 800路 公交到来
2023-01-15 21:51:11.555	|	21	|	ForkJoinPool.commonPool-worker-2	|	800路公交正在赶来
2023-01-15 21:51:11.555	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路公交正在赶来
2023-01-15 21:51:11.756	|	1	|	main	|	800路到了,小白坐车回家
```

{% endspoiler %}

### acceptEither/runAfterEither

acceptEither/runAfterEither 和 applyToEither 的区别？

- applyToEither：获取前两个任务最先完成的任务的结果，处理后得到返回值；
- acceptEither：得到结果，没有返回值；
- runAfterEither：不关心结果，也不关心返回值。

## 4.6、exceptionally

**exceptionally：**<font color='#0000ff' style="font-weight:bold;">在链式调用中如果出现异常，就会调用 exceptionally 块中的内容。</font>

exceptionally 不一定放在最后，放在前面也可以。

<font size=4 style="font-weight:bold;background:yellow;">demo</font>

{% spoiler 展开查看折叠代码 %}

```java
    @Test
    public void testEnd() {
        SmallTool.printTimeAndThread("张三走出餐厅，来到公交站");
        SmallTool.printTimeAndThread("等待 700路 或者 800路 公交到来");
        CompletableFuture<String> bus = CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("700路公交正在赶来");
            // 调整时间，700路一定先到
            SmallTool.sleepMillis(400);
            return "700路到了";
        }).applyToEither(CompletableFuture.supplyAsync(() -> {
            SmallTool.printTimeAndThread("800路公交正在赶来");
            SmallTool.sleepMillis(2000);
            return "800路到了";
        }), firstComeBus -> {
            SmallTool.printTimeAndThread(firstComeBus);
            // 如果先到的是700路，会出现异常
            if (firstComeBus.startsWith("700")) {
                throw new RuntimeException("撞树了...");
            }
            return firstComeBus;
        }).exceptionally(e -> {
            SmallTool.printTimeAndThread(e.getMessage());
            SmallTool.printTimeAndThread("小白叫出租车");
            return "出租车 叫到了";
        });
        SmallTool.printTimeAndThread(String.format("%s,小白坐车回家", bus.join()));
    }

    /**
     * exceptionally也可以放在链式调用的中间进行判断
     */
    @Test
    public void testMiddle() {
        SmallTool.printTimeAndThread("小白走出餐厅，来到公交站");
        SmallTool.printTimeAndThread("等待 700路 或者 800路 公交到来");
        CompletableFuture<String> bus = CompletableFuture.supplyAsync(() -> {
            // 并行执行
            SmallTool.printTimeAndThread("700路公交正在赶来");
            SmallTool.sleepMillis(100);
            if ("700路公交".startsWith("700")) {
                throw new RuntimeException("700路撞树了...");
            }
            return "700路到了";
        }).exceptionally(e -> {
            // 该exceptionally只对上边的第一个supplyAsync进行处理
            SmallTool.printTimeAndThread(e.getMessage());
            SmallTool.printTimeAndThread("700路撞树了，小白骑共享单车回家");
            return "小白骑共享单车";
        }).applyToEither(CompletableFuture.supplyAsync(() -> {
            // 并行执行
            SmallTool.printTimeAndThread("800路公交正在赶来");
            SmallTool.sleepMillis(1000);
            return "800路到了";
        }), firstComeBus -> {
            SmallTool.printTimeAndThread(firstComeBus);
            return firstComeBus;
        }).exceptionally(e -> {
            /**
             * 该exceptionally对上边两个supplyAsync进行处理
             * 由于第一个supplyAsync出现异常已经被处理，future任务bus已经获得结果返回，程序并不会运行到这里
             */
            SmallTool.printTimeAndThread(e.getMessage());
            SmallTool.printTimeAndThread("小白叫出租车");
            return "出租车 叫到了";
        });
        SmallTool.printTimeAndThread(String.format("%s,小白回家", bus.join()));
    }
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">运行结果</font>

**testEnd：**exceptionally 放在程序最后

在程序中，700 路会撞车，并且提前到，小白一定会上 700；

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:54:16.872	|	1	|	main	|	张三走出餐厅，来到公交站
2023-01-15 21:54:16.873	|	1	|	main	|	等待 700路 或者 800路 公交到来
2023-01-15 21:54:16.919	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路公交正在赶来
2023-01-15 21:54:16.919	|	21	|	ForkJoinPool.commonPool-worker-2	|	800路公交正在赶来
2023-01-15 21:54:17.324	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路到了
2023-01-15 21:54:17.324	|	20	|	ForkJoinPool.commonPool-worker-9	|	java.lang.RuntimeException: 撞树了...
2023-01-15 21:54:17.325	|	20	|	ForkJoinPool.commonPool-worker-9	|	小白叫出租车
2023-01-15 21:54:17.325	|	1	|	main	|	出租车 叫到了,小白坐车回家
```

{% endspoiler %}

如果调整时间，让 800 提前到，小白就会上 800，就不会出现异常。

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:55:09.471	|	1	|	main	|	张三走出餐厅，来到公交站
2023-01-15 21:55:09.471	|	1	|	main	|	等待 700路 或者 800路 公交到来
2023-01-15 21:55:09.525	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路公交正在赶来
2023-01-15 21:55:09.525	|	21	|	ForkJoinPool.commonPool-worker-2	|	800路公交正在赶来
2023-01-15 21:55:09.732	|	21	|	ForkJoinPool.commonPool-worker-2	|	800路到了
2023-01-15 21:55:09.733	|	1	|	main	|	800路到了,小白坐车回家
```

{% endspoiler %}

**testMiddle：**exceptionally 放在程序中间

{% spoiler 展开查看折叠代码 %}

```java
2023-01-15 21:54:28.273	|	1	|	main	|	小白走出餐厅，来到公交站
2023-01-15 21:54:28.274	|	1	|	main	|	等待 700路 或者 800路 公交到来
2023-01-15 21:54:28.316	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路公交正在赶来
2023-01-15 21:54:28.317	|	21	|	ForkJoinPool.commonPool-worker-2	|	800路公交正在赶来
2023-01-15 21:54:28.418	|	20	|	ForkJoinPool.commonPool-worker-9	|	java.lang.RuntimeException: 700路撞树了...
2023-01-15 21:54:28.419	|	20	|	ForkJoinPool.commonPool-worker-9	|	700路撞树了，小白骑共享单车回家
2023-01-15 21:54:28.419	|	20	|	ForkJoinPool.commonPool-worker-9	|	小白骑共享单车
2023-01-15 21:54:28.419	|	1	|	main	|	小白骑共享单车,小白回家
```

{% endspoiler %}

### handle/whenComplete

**handle/whenComplete 与 exceptionally 的区别？**

- handle：参数是 BiFunction
  - 如果前面的程序正常执行，那么会接收到正常执行的结果；
  - 如果出现异常，就会接收到异常；
  - 无论正常或异常，都会返回一个结果，让后面的程序继续执行。
- whenComplete：与 handle 类似，没有返回值。
  - whenCompleteAsync：使用异步处理

```java
// 开启一个异步方法
CompletableFuture<List> future = CompletableFuture.supplyAsync(() -> {
    List<String> list = new ArrayList<>();
    list.add("语文");
    list.add("数学");
    // 获取得到今天的所有课程
    return list;
});
// 使用handle()方法接收list数据和error异常
CompletableFuture<Integer> future2 = future.handle((list,error)-> {
    // 如果报错，就打印出异常
    error.printStackTrace();
    // 如果不报错，返回一个包含Integer的全新的CompletableFuture
    return list.size();
    // 注意这里的两个CompletableFuture包含的返回类型不同
});
```

## 4.7、complete/completeExceptionally

complete：主动触发当前异步任务的完成。

- 调用此方法时如果你的任务已经完成，那么方法就会返回 false；
- 如果任务没完成，就会返回 true，**并且其它线程获取到的任务的结果就是 complete 的参数值。**

completeExceptionally：跟 complete 的作用差不多，complete 是正常结束任务，返回结果，而completeExceptionally 就是触发任务执行的异常。

```java
public class _07_complete {

    public void completeTest() throws ExecutionException, InterruptedException {
        CompletableFuture<String> future = new CompletableFuture<>();
        try {
            future.complete("test success");
        } catch (Exception e) {
            future.completeExceptionally(e);
        }         
        System.out.println(future.get());
    }
}
```

## 4.8、allOf/anyOf

allOf：多个任务都执行完成后才会执行。

- 只有一个任务执行异常，则返回的 CompletableFuture 执行 get 方法时会抛出异常；
- 如果都是正常执行，则 get 返回 null。

anyOf：多个任务中只要有一个任务执行完成，get 就会返回已经执行完成任务的结果。

- 成功执行就返回成功执行的结果；
- 有异常就返回异常。

**举例如下：**

- allOf 等待所有任务执行完成才执行 cfAll，如果有一个任务异常终止，则 `cfAll.get()` 时会抛出异常，都是正常执行，`cfAll.get()` 返回 null；
- anyOf 是只有一个任务执行完成，无论是正常执行或者执行异常，都会执行 cfAny，`cfAny.get()`的结果就是已执行完成的任务的执行结果。

```java
public class _08_allOfAnyOf {

    public void testAllOfAnyOf() throws ExecutionException, InterruptedException {

        // cf1会提前完成
        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            SmallTool.sleepMillis(10);
            SmallTool.printTimeAndThread("cf1 working...");
            return "cf1 任务完成";
        });

        CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> {
            SmallTool.sleepMillis(500);
            SmallTool.printTimeAndThread("cf2 working...");
            return "cf2 任务完成";
        });

        CompletableFuture<String> cf3 = CompletableFuture.supplyAsync(() -> {
            SmallTool.sleepMillis(500);
            SmallTool.printTimeAndThread("cf3 working...");
//        int i = 1 / 0;
            return "cf3 任务完成";
        });

        System.out.println("===========cfAll===========");
        /**
         * 有异常就抛异常
         * 没异常get()会返回null
         */
        CompletableFuture<Void> cfAll = CompletableFuture.allOf(cf1, cf2, cf3);
        System.out.println("cfAll->" + cfAll.get());
        System.out.println("===========cfAny===========");
        /**
         * cf3会出现异常，但是cf1提前完成，anyOf会提前返回结果
         * 此时程序直接结束，不会抛出异常
         */
        CompletableFuture<Object> cfAny = CompletableFuture.anyOf(cf1, cf2, cf3);
        System.out.println("cfAny->" + cfAny.get());

    }
}
```

# 5、规律

<font size=4 style="font-weight:bold;background:yellow;">规律</font>

```
xxx(arg)
xxxAsync(arg)
xxxAsync(arg,Executor)
```

- 一般情况下，大多数方法的构造方法都会有一个带有两个参数的重载构造方法，会增加一个线程池 Executor 参数，如 supplyAsync：

  ```java
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
  ```

- 有方法 xxx，就会有 xxxAsync 方法，例如 `thenCompose/thenComposeAsync`、`thenApply/thenApplyAsync`、`thenCombine/thenCombineAsync`。

<font size=4 style="font-weight:bold;background:yellow;">Async</font>

- 不加的话，在 CompletableFuture 看来，前后两段代码就是同一个任务，运行时会把后面的代码放到前一段代码的末尾封装成一个任务交给一个线程运行；

- async：异步的，加上这个单词的话，CompletableFuture 会把前后两段代码当成是独立的两个任务，分别放到两个线程中执行。

- 以下面的代码为例，在 thenComposeAsync 不再调用 CompletableFuture，而是直接返回 CompletableFuture，这样在 return 之前还可以进行其他的任务；

  - 使用 thenCompose，因为前后两段代码本身就不在一个线程中执行，使用 return 之后可以看得更加明显。线程 2 会被添加到线程 1 的末尾，划分到同一个任务中提交到线程池，而线程 3 是另一个任务，在整个程序中实际上有 2 个异步任务。
  - 使用 thenComposeAsync，线程 2 就是一个独立的任务，这样一来三个线程都是各自独立，每次任务结束后都会重新寻找空闲线程运行，在整个程序中实际上有 3 个异步任务。（由于线程复用，前后任务选择的可能是同一个线程）

  ```java
  public class ThenComposeAsyncTest {
      public static void main(String[] args) {
          CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
              SmallTool.printTimeAndThread("厨师炒菜");
              SmallTool.sleepMillis(200);
              return "番茄炒蛋";		// 线程1
          }).thenComposeAsync(dish -> {
              SmallTool.printTimeAndThread("服务员A 准备打饭，但是被领导叫走，打饭交接给服务员B");	// 线程2
              return CompletableFuture.supplyAsync(() -> {
                  SmallTool.printTimeAndThread("服务员B 打饭");
                  SmallTool.sleepMillis(100);
                  return dish + " + 米饭";		// 线程3
              });
          });
          SmallTool.printTimeAndThread(cf1.join()+"好了，开饭");
      }
  }
  ```

<font size=4 style="font-weight:bold;background:yellow;">相同类型的方法</font>

| 无入参，有出参 |    有入参，有出参    | 有入参，无出参 | 无入参，无出参 |
| :------------: | :------------------: | :------------: | :------------: |
|  supplyAsync   |                      |                |    runAsync    |
|                |     thenCompose      |                |                |
|                |     thenCombine      | thenAcceptBoth |  runAfterBoth  |
|                |      thenApply       |   thenAccept   |    thenRun     |
|                |    applyToEither     |  acceptEither  | runAfterEither |
|                | exceptionally/handle |  whenComplete  |                |



# 6、CompletableFuture 与 ListenableFuture 之间的转换

> 更新时间：2022-09-11 18:20

FutureUtil 工具类，实现 ListenableFuture 和 CompletableFuture 的相互转换：

{% spoiler 展开查看折叠代码 %}

```java
public class FutureUtil {
    public static <T> CompletableFuture<T> convert(ListenableFuture<T> lf) {
        // CompletableFuture中的线程默认是守护线程，这里也可以选择设为守护线程
/*        return convert(lf, Executors.newSingleThreadExecutor(new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setDaemon(true);
                t.setName("CompletableFuture");
                return t;
            }
        }));*/
        return convert(lf, Executors.newSingleThreadExecutor());
    }

    // 重载
    public static <T> CompletableFuture<T> convert(ListenableFuture<T> lf, Executor executor) {
        if (lf == null) {
            return null;
        }
        CompletableFuture<T> cf = new CompletableFuture<>();
        lf.addListener(() -> {
            T value = null;
            try {
                value = lf.get();
                cf.complete(value);
            } catch (InterruptedException | ExecutionException e) {
                cf.completeExceptionally(e);
            }
        }, executor);
        return cf;
    }

    public static <T> ListenableFuture<T> convert(CompletableFuture<T> cf) {
        if (cf == null) {
            return null;
        }
        SettableFuture<T> lf = SettableFuture.create();
        cf.whenComplete((r, t) -> {
            if (t != null) {
                lf.setException(t);
            } else {
                lf.set(r);
            }
        });
        return lf;
    }
}
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">测试</font>

{% spoiler 展开查看折叠代码 %}

```java
public class TestConvert {

    private final Logger LOGGER = Logger.getLogger(TestConvert.class);

    class Task implements Callable<String> {
        @Override
        public String call() throws Exception {
            TimeUnit.SECONDS.sleep(1);
//            int a = 1 / 0;    // 异常
            return "task done";
        }
    }

    @Test
    public void testLfToCf() {
        ListeningExecutorService pool = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(5));
        ListenableFuture<String> lf = pool.submit(new Task());
        CompletableFuture<String> cf = FutureUtil.convert(lf);
        LOGGER.info(cf.join());
    }

    @Test
    public void testCfToLf() throws ExecutionException, InterruptedException {
        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> "cf supplyAsync");
        ListenableFuture<String> lf = FutureUtil.convert(cf);
        LOGGER.info(lf.get());
    }

    @Test
    public void testCfToLf2() {
        CompletableFuture<String> cf = new CompletableFuture();
//        cf.complete("cf complete");
        cf.completeExceptionally(new Exception("cf exception"));
        ListenableFuture<String> lf = FutureUtil.convert(cf);
        Futures.addCallback(lf, new FutureCallback<String>() {
            @Override
            public void onSuccess(String result) {
                LOGGER.info("success->" + result);
            }
            @Override
            public void onFailure(Throwable t) {
                LOGGER.info("failure->" + t);
            }
        });
    }
}
```

{% endspoiler %}



{% spoiler 展开查看折叠代码 %}

```java
    /**
     * 将ListenableFuture转换为CompletableFuture
     *
     * @param lf
     * @param <T>
     * @return
     */
    public static <T> CompletableFuture<T> convert(ListenableFuture<T> lf) {
        if (lf == null) {
            return null;
        }
        CompletableFuture<T> cf = new CompletableFuture<>();
        Futures.addCallback(lf, new FutureCallback<T>() {
            @Override
            public void onSuccess(T result) {
                cf.complete(result);
            }

            @Override
            public void onFailure(Throwable t) {
                cf.completeExceptionally(t);
            }
        });
        return cf;
    }

    /**
     * 将CompletableFuture转换为ListenableFuture
     *
     * @param cf
     * @param <T>
     * @return
     */
    public static <T> ListenableFuture<T> convert(CompletableFuture<T> cf) {
        if (cf == null) {
            return null;
        }
        SettableFuture<T> lf = SettableFuture.create();
        cf.whenComplete((r, t) -> {
            if (t != null) {
                lf.setException(t);
            } else {
                lf.set(r);
            }
        });
        return lf;
    }
```

{% endspoiler %}

