---
title: guava中的集合类
date: 2022-09-29
tags: [guava,集合]
---

# 1、ImmutableMap

## 1.1、Immutable 的概念

> http://www.blogjava.net/stevenjohn/archive/2015/01/08/422140.html

如果一个对象实例不能被更改，该对象就是一个 Immutable 的对象，比如 String 等都是 Immutable 的对象

**使对象变得 Immutable 的条件**

- 保证类不能被继承，为了避免其继承的类进行 mutable 的操作；
- 去掉所有 setter/update 等修改对象实例的操作；
- 保证所有的 field 是 private 和 final 的。

<!--more-->

**不可变对象与可变对象**

- 不可变对象（immutable Objects）就是那些一旦被创建，它们的状态就不能被改变的对象，每次对他们的改变都是产生了新的不可变对象，如 String（但是也会有安全问题：[密码不能放在 String 中](https://my.oschina.net/jasonultimate/blog/166968)）

- mutable Objects 就是那些创建后，状态可以被改变的对象，如 StringBuilder

**Immutable 优点**

immutable 也有一个缺点就是会制造大量垃圾，由于他们不能被重用而且对于它们的使用就是”用“然后”扔“，字符串就是一个典型的例子，它会创造很多的垃圾，给垃圾收集带来很大的麻烦。当然这只是个极端的例子，合理的使用 immutable 对象会创造很大的价值。

- Immutable 对象是线程安全的，可以不用被 synchronize 就在并发环境中共享；
- Immutable 对象简化了程序开发，因为它无需使用额外的锁机制就可以在线程间共享；
- Immutable 对象提高了程序的性能，因为它减少了 synchroinzed 的使用

- Immutable 对象是可以被重复使用的，可以将它们缓存起来重复使用，就像字符串字面量和整型数字一样。可以使用静态工厂方法来提供类似于 valueOf() 这样的方法，它可以从缓存中返回一个已经存在的 Immutable 对象，而不是重新创建一个。

## 1.2、ImmutableMap

创建的两种方法，demo 如下：

{% spoiler 展开查看折叠代码 %}

```java
public class ImmutableMapDemo {

    public void newMapTest() {
        /**
         * 使用Builder创建ImmutableMap
         * Map<String, String> immutableMap = new ImmutableMap.Builder<String, String>().build();
         */
        Map<String, String> immutableMap = new ImmutableMap.Builder<String, String>()
                .put("1", "aa")
                .put("2", "bb")
                .build();
        System.out.println(immutableMap.toString());
    }

    public void ofTest() {
        /**
         * 使用of创建，最多5对kv
         */
        Map<String, String> immutableMap = ImmutableMap.of(
                "3", "cc",
                "4", "dd"
        );
        System.out.println(immutableMap.toString());
    }

    public static void main(String[] args) {
        ImmutableMapDemo demo = new ImmutableMapDemo();
        demo.ofTest();
    }

}
```

{% endspoiler %}

# 2、Maps

不用 new 创建 HashMap

```java
Map<String, String> aNewMap = Maps.newHashMap();
```

