---
title: guava
date: 2022-08-14
tags: [guava]
toc: true
---

# 1、Java

<!--more-->

## 基础数据类型

溢出问题，改成5000L，不能用1000L或是3600L，因为之前已经溢出，乘完之后还是溢出

![image-20220715102314037](guava/image-20220715102314037.png)

![image-20220715102440126](guava/image-20220715102440126.png)

空指针问题

![image-20220715102558105](guava/image-20220715102558105.png)

精度丢失问题

![image-20220715102622108](guava/image-20220715102622108.png)

![image-20220715102646651](guava/image-20220715102646651.png)

BigDecimal

![image-20220715102808989](guava/image-20220715102808989.png)

![image-20220715102831841](guava/image-20220715102831841.png)

equals：既比较大小又比较精度

compareTo：只比较大小

![image-20220715102938992](guava/image-20220715102938992.png)

不指定精度会无限循环

![image-20220715103048199](guava/image-20220715103048199.png)

指定精度，保留16位

![image-20220715103107822](guava/image-20220715103107822.png)

公司封装的Money类，防止忘记设置精度问题

![image-20220715103249827](guava/image-20220715103249827.png)

## StringBuilder

**StringBuilder 与 StringBuffer 的区别**

- 2 线程安全，1 线程不安全

- 大多数单线程，1 用的多，效率也高，不需要锁获取和释放

**StringBuilder**

- 拼接null会自动加上`"null"`
- 扩容：value*2+2	6->14，第一句不扩容，第二句扩容

![image-20220715104140384](guava/image-20220715104140384.png)

# 2、Guava

## Strings

判断 `null / ""` 及转换

![image-20220715104344329](guava/image-20220715104344329.png)

公共前缀/后缀

![image-20220715104516419](guava/image-20220715104516419.png)

padding，追加（前/后）字符串，原理 StringBuilder

![image-20220715104605174](guava/image-20220715104605174.png)

## Ints

补充 Integer 、Arrays 中对 int 的操作

**比较**

溢出，负数

![image-20220715105014353](guava/image-20220715105014353.png)

**判断是否包含**

![image-20220715105138967](guava/image-20220715105138967.png)

**max/min**

![image-20220715105156872](guava/image-20220715105156872.png)

**asList**

JDK List 中是数组

![image-20220715105541936](guava/image-20220715105541936.png)

Guava 返回的 list 不可变，不能使用 add 方法

![image-20220715105515759](guava/image-20220715105515759.png)

## Splitter

Java 使用正则表达式 split 会分不彻底

![image-20220715111213853](guava/image-20220715111213853.png)

trimResults：去掉空格，后边还可以接 CharMatcher

![image-20220715111341942](guava/image-20220715111341942.png)

omitEmptyString：去掉空的元素

![image-20220715111642842](guava/image-20220715111642842.png)

将 String 分割成 Map

![image-20220715111734321](guava/image-20220715111734321.png)

## Joiner

joiner，线程安全，不可变

**join()：将字符串插入**

![image-20220715105713112](guava/image-20220715105713112.png)

数组 a 中不能有 null 元素

![image-20220715105828444](guava/image-20220715105828444.png)

跳过 null：`skipNulls()`

![image-20220715105912356](guava/image-20220715105912356.png)

替换 null：`useForNull()`

![image-20220715110008521](guava/image-20220715110008521.png)

- `useForNull()`，`skipNulls()`，不能同时使用！会抛异常

- 也不能连续两次使用两次`useForNull()`

joiner 链接 map，给 url 拼参数

![image-20220715110817364](guava/image-20220715110817364.png)





## Objects

Guava 操作 Object 的辅助类

![image-20220718105426361](guava/image-20220718105426361.png)

**equal**

JDK 中的 equals 不能为 null，使用 Objects 中的 equal 就随意了

![image-20220715113617159](guava/image-20220715113617159.png)

**hashCode **

![image-20220715113738643](guava/image-20220715113738643.png)

**firstNonNull**

谁空返回谁

![image-20220715113837646](guava/image-20220715113837646.png)

**toSringHelper**

![image-20220715114056143](guava/image-20220715114056143.png)



![image-20220715114158755](guava/image-20220715114158755.png)

## CharMatcher

![image-20220715114511416](guava/image-20220715114511416.png)

**removeFrom/retainFrom**，**anyOf**

![image-20220715114322483](guava/image-20220715114322483.png)

![image-20220715114347205](guava/image-20220715114347205.png)

**or，is**

![image-20220715114433600](guava/image-20220715114433600.png)

配合 Splitter

![image-20220715114913016](guava/image-20220715114913016.png)

![image-20220715114956751](guava/image-20220715114956751.png)

## Optional

告知调用方返回值可能为 null

![image-20220718105527443](guava/image-20220718105527443.png)

用法1

![image-20220718105932125](guava/image-20220718105932125.png)

用法2

![image-20220718110112795](guava/image-20220718110112795.png)



 







## Function

![image-20220715115201151](guava/image-20220715115201151.png)

function：接受输入参数，调用 `apply()` 之后得到输出结果

![image-20220715115251679](guava/image-20220715115251679.png)



predicate

![image-20220715165612507](guava/image-20220715165612507.png)



List 之间的转换

对 Lists.transform 返回的结果使用 add 方法会抛异常

以下图中为例，想实现添加元素可以对 intList 使用 add 方法 

![image-20220715165813093](guava/image-20220715165813093.png)

predicate

![image-20220718105107130](guava/image-20220718105107130.png)

当有更多比5大的元素时，返回一个列表

![image-20220718105208855](guava/image-20220718105208855.png)



![image-20220718105337164](guava/image-20220718105337164.png)



举例

![image-20220718105358066](guava/image-20220718105358066.png)



# 3、Joda-Time

官网：https://www.joda.org/joda-time/

Java 内部的日期类十分不方便，这些方法已经被废弃

![image-20220718110455137](guava/image-20220718110455137.png)

推荐使用这种方法，从 0 开始计数不方便

![image-20220718110846614](guava/image-20220718110846614.png)

Joda-Time

![image-20220718111107925](guava/image-20220718111107925.png)

还可以转换日期格式，计算几个月几天后的时间，计算时间之间间隔的天数

![image-20220718111603269](guava/image-20220718111603269.png)

simpleDateFormat 消耗很多资源，时间慢效率低，而且线程不安全

![image-20220718111801477](guava/image-20220718111801477.png)

Joda-Time 线程安全，可以使用 static 关键字避免多次初始化消耗资源

![image-20220718111943919](guava/image-20220718111943919.png)

在进行日期转换时，sdf 还会有隐藏的 bug，比如 6 月没有 31 日，要进行额外的设置才会报异常：

![image-20220718112439863](guava/image-20220718112439863.png)

# 4、Java 异常处理

![image-20220718113831873](guava/image-20220718113831873.png)

## 4.1、Java

**==异常继承关系与介绍==**

<img src="guava/image-20220718114948411.png" alt="image-20220718114948411" style="zoom: 67%;" />

**Error**

![image-20220718114426304](guava/image-20220718114426304.png)

**Exception**

- 受检异常和非受检异常

  ![image-20220718114513830](guava/image-20220718114513830.png)

- 常用方法

  ![image-20220718114651804](guava/image-20220718114651804.png)

**==处理异常==**

- 使用错误码

  ![image-20220718114019745](guava/image-20220718114019745.png)

- 使用异常

  ![image-20220718114124573](guava/image-20220718114124573.png)

## 4.2、如何处理异常

**==什么时候抛出异常？==**

![image-20220718115223015](guava/image-20220718115223015.png)

**==社么时候捕获异常？==**

- 当能够对异常进行妥善处理时，捕获异常，否则不捕获；

  ![image-20220718115609446](guava/image-20220718115609446.png)

- 需要进行事务回滚等操作时；

  如下图，在事务提交之前出现了异常，则需要进行事务的回滚并捕获异常进行日志监控等工作，同时在 final 块中进行资源的释放。

  ![image-20220718115702885](guava/image-20220718115702885.png)

- 关注某些特定异常类型，需要进行业务、日志、监控处理时，捕获异常，向上抛出；

  ![image-20220718120300552](guava/image-20220718120300552.png)

**==捕获异常后==**

![image-20220718120441384](guava/image-20220718120441384.png)

**==处理异常==**

- **catch / finally**

  ![image-20220718120600619](guava/image-20220718120600619.png)

## 4.3、Guava 中的异常处理

**==Throwables==**

- **getCasualChain**

  得到异常链，一般用于跟踪某种特定的异常；

- **getRootCause**

  得到最原始的异常，即最早的 throw 点；

- **propagate**

  将 Exception 包装成 RuntimeException 或 Error，简化异常处理；

  在如下代码中，willThrowExceptions 会抛出多种异常，但是 catch 只关心 ArithmeticException，在遇到 IllegalArgumentException 时会原样抛出，其余的 exception 会通过 propagate 包装成 RuntimeException  后抛出。

  ![image-20220718162714776](guava/image-20220718162714776.png)

- **getStackTraceAsString**

  获取异常调用栈信息，通过 String 返回，当需要使用邮件的形式发送异常的详细信息时可以使用这个方法



# 5、日志与规范

## 5.1、日志与标准输出/错误流

**==为什么要记录日志？==**

<img src="guava/image-20220718163756484.png" alt="image-20220718163756484" style="zoom: 67%;" />

**==使用标准输出/错误流的问题==**

![image-20220718164013714](guava/image-20220718164013714.png)

**==使用日志系统简化日志工作==**

![image-20220718164039140](guava/image-20220718164039140.png)

## 5.2、slf4j 与 logback

![image-20220718164151150](guava/image-20220718164151150.png)

![image-20220718164205750](guava/image-20220718164205750.png)

**==logback：Logger/Appender/Layout==**

- Logger 面向应用程序，用于产生日志事件。

  - 在调用 Appender 之前，需要根据有效日志级别来决策日志事件是否有效；

  - 有效日志级别可配置。

- Appender 用于将日志事件代理到具体的日志记录组件。

  - 例如 DBAppender 用于将日志事件记录到数据库中；
  - FileAppender 用于将日志事件记录到文件；

  - RollingFileAppender 可以将日志按一 定策略进行分割存储。

- Layout 用于将日志事件格式化成具体的日志记录；

  - HTMLLayout 可以将日志事件格式化成 HTML，PatternLayout则可以根据配置的 Pattern 格式化
  日志事件

**==logback 配置==**

![image-20220718165925985](guava/image-20220718165925985.png)

**==logback 日志级别==**

![image-20220718165949152](guava/image-20220718165949152.png)

## 5.3、如何查看日志

![image-20220718170121062](guava/image-20220718170121062.png)

## 5.4、Qunar 日志规范

![image-20220718170514588](guava/image-20220718170514588.png)

![image-20220718170610395](guava/image-20220718170610395.png)

![image-20220718170856089](guava/image-20220718170856089.png)