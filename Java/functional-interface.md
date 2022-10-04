---
title: 函数式接口
date: 2022-10-04
tags: [函数式接口]
toc: true
---

# 1、函数式接口

**只有一个抽象方法的接口就叫函数式接口，**Runnable 接口中只有一个 `run()` 方法，所以 Runnable 接口就是一个函数式接口。

只要是函数式接口，就可以用 lambda 表达式简化。

```java
public interface Runnable {
	public abstract void run();
}
```

对于函数式接口，可以通过 lambda 表达式来创建该接口的对象，避免匿名内部类定义过多，可以让代码更简洁的一种表示方法。

<!--more-->

# 2、lambda 表达式（JDK 8）

**使用 lambda 表达式简化一个函数式接口对象的过程：**<font color='red' style="font-weight:bold;">普通实现类 -> 静态内部类 -> 局部内部类 -> 匿名内部类 -> lambda表达式</font>

{% spoiler 展开查看折叠代码 %}

```java
// 定义一个函数式接口
interface ILikeLambda {
    void lambda();
}

class Like implements ILikeLambda {	// 普通实现类
    @Override
    public void lambda() {
        System.out.println("实现类：Lambda666!");
    }
}

public class LambdaTest01 {
    static class Like2 implements ILikeLambda {	// 静态内部类
        @Override
        public void lambda() {
            System.out.println("静态内部类：Lambda666!");
        }
    }

    public static void main(String[] args) {
        ILikeLambda like = null;

        // 1、普通实现类
        like = new Like();
        like.lambda();
        
        // 2、静态内部类
        like = new Like2();
        like.lambda();
        
        // 3、局部内部类
        class Like3 implements ILikeLambda {
            @Override
            public void lambda() {
                System.out.println("局部内部类：Lambda666!");
            }
        }
        like = new Like3();
        like.lambda();
        
        // 4、匿名内部类，没有类的名称，必须借助父类或者接口
        like = new ILikeLambda() {
            @Override
            public void lambda() {
                System.out.println("匿名内部类：Lambda666!");
            }
        };
        like.lambda();
        
        // 5、用lambda表达式简化
        like = () -> {
            System.out.println("Lambda:Lambda666!");
        };
        like.lambda();
    }
}
```

{% endspoiler %}

**Lambda 表达式本身还可以进行简化**

比如函数式接口 Runnable 使用 Lambda 表达式不断简化：<font color='red' style="font-weight:bold;">去掉参数类型 -> 去括号 -> 去花括号</font>

{% spoiler 展开查看折叠代码 %}

```java
interface ILoveLambda {
    void love(int a);
}

public class LambdaTest02 {

    public static void main(String[] args) {
        ILoveLambda i = null;

        // 普通的lambda表达式:()->{}
        i = (int a) -> {
            System.out.println("普通的lambda表达式-->" + a);
        };
        i.love(666);

        // 简化1：去掉参数类型
        i = (a) -> {
            System.out.println("lambda去掉参数类型-->" + a);
        };
        i.love(777);

        // 简化2：去掉括号
        i = a -> {
            System.out.println("lambda去掉括号-->" + a);
        };
        i.love(888);

        // 简化3：去掉花括号
        i = a -> System.out.println("lambda去掉花括号-->" + a);
        i.love(999);
    }
}
```

{% endspoiler %}

**简化的注意事项**

- 接口必须是函数式接口；

- 方法如果有多个参数也可以去掉参数类型，但是参数类型要去掉就都去掉，去掉之后必须加上括号；
- Lambda 表达式只能在有一行代码的情况下才能简化成一行，如果有多行就用代码块包裹。

# 3、四大函数式接口

## 3.1、Supplier

`java.util.function.Supplier<T>` 接口，仅包含一个无参的方法：`T get()`，用来获取一个泛型参数指定类型的对象数据

Supplier 接口被称之为生产型接口，传入一个泛型 T，将返回一个泛型 T。泛型如果不进行特别指定会默认为 Object，注意要与函数式接口的传入返回类型相匹配。

**测试**

{% spoiler 展开查看折叠代码 %}

```java
public class _01_SupplierDemo_get {
    // 定义一个方法，方法的参数传递Supplier<T>接口，泛型设置为String，get方法就会返回一个String
    public static String getString(Supplier<String> sup){
        return sup.get();
    }

    public static void helloSupplier(){
        // 调用getString方法，方法参数Supplier是一个函数式接口，所以可以传递Lambda表达式
        String s = getString(() -> {
            // 生产一个字符串，并返回
            return "hello,supplier!";
        });
        System.out.println(s);
    }

    public static void byeSupplier(){
        // 简化
        String s1 = getString(() -> "goodbye,supplier!");
        System.out.println(s1);
    }

    public void supplierTest(){
        // 传一个字符串：hello world，使用get返回这个字符串
        Supplier<String> supplier = () -> "hello world";
        System.out.println(supplier.get());

        // 传一个对象，使用get返回这个对象
        Supplier<Student> studentSupplier = () -> new Student("小明",12);
        System.out.println(studentSupplier.get());
    }


    public static void main(String[] args) {
        helloSupplier();
        byeSupplier();
        System.out.println("================");
        _01_SupplierDemo_get a01SupplierDemoAccept = new _01_SupplierDemo_get();
        a01SupplierDemoAccept.supplierTest();
    }


    class Student{
        private String name;
        private int age;

        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}

```

{% endspoiler %}

## 3.2、Consumer

> https://blog.csdn.net/weixin_44230693/article/details/113847162

`java.util.function.Consumer` 接口则正好与 Supplier 接口相反，它不是生产一个数据，而是消费一个数据， 其数据类型由泛型决定。

<font size=4 style="font-weight:bold;background:yellow;">accept 方法</font>

消费一个指定泛型的数据

**测试**

{% spoiler 展开查看折叠代码 %}

```java
public class _02_ConsumerDemo_accept {

    /**
     * 定义一个方法，方法的参数传递一个字符串的姓名
     * 方法的参数传递Consumer接口，泛型使用String，可以使用Consumer接口消费字符串的姓名
     */
    public static void method(String name, Consumer<String> con) {
        con.accept(name);
    }

    public static void consumerTest() {
        // 调用method方法，传递字符串姓名，方法的另一个参数是Consumer接口，是一个函数式接口，所以可以传递Lambda表达式
        method("小明", name -> {
            // 对传递的字符串进行消费，消费方式：直接输出字符串
            System.out.println(name);	// 小明
            // 消费方式:把字符串进行反转输出
            String str = new StringBuilder(name).reverse().toString();
            System.out.println(str);	// 明小
        });
    }

    public static void main(String[] args) {
        consumerTest();
    }
}
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">andThen 方法</font>

如果一个方法的参数和返回值全都是 Consumer 类型，在消费数据的时候，可以首先做一个操作，然后再做一个操作，实现组合。由 Consumer 接口中的 default 方法 `andThen()` 可以实现。

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    // 在参数为null时主动抛出NullPointerException异常，避免对null的判断和抛出异常
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```

Consumer 接口的默认方法 andThen：需要两个 Consumer 接口，可以把两个 Consumer 接口组合到一起，再对数据进行消费。

```java
Consumer<String> con1
Consumer<String> con2
String s = "hello";
con1.accept(s);
con2.accept(s);
// 连接两个Consumer接口进行消费，谁在前边谁先消费
con1.andThen(con2).accept(s);
```

**测试**

{% spoiler 展开查看折叠代码 %}

```java
public class _03_ConsumerDemo_andThen {
    // 定义一个方法，方法的参数传递一个字符串和两个Consumer接口，Consumer接口的泛型使用字符串
    public static void method(String s, Consumer<String> con1, Consumer<String> con2) {
        // con1.accept(s);
        // con2.accept(s);
        // 使用andThen方法，把两个Consumer接口连接到一起，再消费数据
        con1.andThen(con2).accept(s);
    }

    public static void main(String[] args) {
        // 调用method方法，传递一个字符串，两个lambda表达式
        method("hello",
                (t) -> {
                    // 消费方式：把字符串转换为大写输出
                    System.out.println(t.toUpperCase());
                },
                (t) -> {
                    // 消费方式：把字符串转换为小写输出
                    System.out.println(t.toLowerCase());
                });
    }
}
```

{% endspoiler %}

## 3.3、Function

Function：函数型接口，有一个输入参数，有一个输出参数。

**源码**

{% spoiler 展开查看折叠代码 %}

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * 传入类型T，返回类型R
     */
    R apply(T t);

    /**
     * 
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * 
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

{% endspoiler %}

**测试：**将输入的小写字母转换为大写

```java
public class _04_FunctionDemo {
    public static void functionTest(String str, Function<String,String> function) {
        System.out.println(function.apply(str));
    }
    public static void main(String[] args) {
        functionTest("hhh",str->{
            // 将输入的小写字母转换为大写
            return str.toUpperCase();
        });
    }
}
```

## 3.4、Predicate

Predicate：断定型接口，只有一个输入参数，返回值只能是 boolean 值。

**源码**

{% spoiler 展开查看折叠代码 %}

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}

```

{% endspoiler %}

**测试：**判断字符串是否为空

```java
public class _05_PredicateDemo {
    public static void main(String[] args) {
        Predicate<String> p = str -> str.isEmpty();
        System.out.println(p.test(""));     // true
        System.out.println(p.test("a"));    // false
    }
}
```