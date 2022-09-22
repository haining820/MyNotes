---
title: qfc课堂笔记
date: 2022-08-14
tags: []
---

# 1、命名

parse，tryParse：`try-` -> 通过返回值表示成功或失败

XXXs、XXXXUtils：工具类

XXXs2：如 `com.google.commom.collect.Collections2`，已经有其他的权威实现了，所以加一个 ”2“。

**读文档/注释**

用：看入参出参，抛出的异常，和已有的工具作比较

开发内部工具：对比已有功能

<!--more-->

# 2、String

- 使用 final 关键字修饰，不可变，`private final char values[]`

- 线程安全

- interning：字符串常量池

  IntegerCache：

- hashcode 在创建时缓存，适合做 map 的 key，速度快

  适合做 key的条件：

  - hashcode  不变
  - 速度，如 String 不仅不变，还会缓存 hashcode

# 3、splitter/joiner

编解码，序列化角度看 splitter/joiner

- 容量
- 可读性，日志等人看的东西，要用明文
- 操作难度
- 性能

# 4、读源码

- 精读/泛读

- 先自己思考尝试写出来

- 进行对比，验证想法是否正确

# 5、开源的正确打开方式

**如何学习 guava/其他技术知识？**

- wiki：”生肉“ -> https://github.com/google/guava/wiki

- release：https://github.com/google/guava/releases

- issues：https://github.com/google/guava/issues

升级系统组件版本，要看官方文档，查看现有版本与目标版本的区别

# 6、线程/异步/并发

**==池化思想==**

**池化的问题**

- 污染问题；
- ThreadLocal；
- 适应问题：线程池调优，调整线程池的参数，根据任务的请求量以及处理任务的时间等设置合适的参数，不断尝试；
- 对应的数据库连接池配置，比如连接数是 5，但是并发量很高，还要再调。

**==Future 与异步==**

Future 的问题？

- 阻塞：线程阻塞，浪费资源
- 轮询：有间隔时间

**==ListenableFuture==**

出得早，先入为主，占据了大量第三方接口库的设计，如 ningHttpClient 的返回值、dubbo 的异步调用模式返回值

Future 的增强，给 Future 添加监听器，通知，事件完成后立即执行。

多监听器

```
listenableFutre
listen1,listen2,listen3...
```

- 实现：SettableFuture，ListenableFutureTask

Fiteures：Future 工具类

successfulAsList/allAsList：true/false

**==CompletableFuture==**

JDK，链式调用方便

# 7、并发编程常见问题

多线程，线程池中可能会出现异常，异常可能会被吞掉

异常的表示

- Ints，转化为结果
- 状态码

# 8、资源隔离

线程池隔离

根据返回结果需要的时间对连接池进行分类

内存级别

- http 连接池

分布式存储

- 数据库分库分表：资源崩的时候不会互相影响

- redis 分片

- ES 分片

分布式部署

- 应用分组，集群

  搜索/交易：搜索多，但是不买；交易少但是重要，放在一起搜索会占用交易的资源，需要分配到不同的集群中进行隔离

# 9、lazily vs eagerly

懒加载/预加载：线程池的预初始化机制，对于请求量大的系统可以预先加载资源，请求量非常大，资源不会浪费。

对于双11、618等，会将数据全部预加载到内存中，防止崩溃





10、map key 的更改

将类中的成员变量声明为 final

TreeMap 排序问题，读多写少 写多读少 什么时候排序

guava suppliers实现懒加载

countDownLatch

缓存： 几个要素

# 10、

```bash
awk sort -c unique sort
```

sdf 线程不安全，明确线程安全





输入异常处理

在完成功能后考虑可能出问题的点

预留拓展空间



# MySQL

写 sql update、delete、时一定要带 where 条件

引擎统一 InnoDB、字符集统一使用 utf8mb4

数据库架构

- 3M

- QMHA
  - namespace 切换方式

- PXC
  - 三个节点都可以读，不用建立主从关系
  - namespace 切换方式

# Redis

Bigkey：大小或元素数量超过一定数量的 key。

- DB故障，设置超时时间，先随机少量重试

- 缓存故障

Qunar Redis



tcDEV





AQS

JUC 中核心的就是 AQS

获取同步状态，线程睡眠

- 属性 state、CHL 同步队列
- 两种模式独占模式、共享模式

## 同步状态state







RetrannenetLock













