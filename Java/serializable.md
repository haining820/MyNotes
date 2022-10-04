---
title: 序列化
date: 2022-10-04
tags: [Java,序列化]
toc: true
---

# 1、序列化

> https://mp.weixin.qq.com/s/0EfIUB9E-0Oh_Clwuxswuw

序列化（Serialization）是将对象的状态信息转换为可以存储或传输的形式的过程。

在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。除了**字符串、数组、枚举**之外，一个对象想要进行序列化，就必须要有实现 Serializable 接口，否则会抛出 NotSerializableException 异常。

- 序列化：将 Java 对象转换为字节序列；
- 反序列化：把字节序列回复为原先的 Java 对象。

<!--more-->

序列化实际上就是将数据持久化，防止一直存储在内存当中，消耗内存资源，而且序列化后更方便传输，用途：

- 把对象的字节序列永久保存到硬盘上，通常存放在一个文件中（序列化对象）
- 在网络上传输对象的字节序列（网络传输对象）

## 1.1、序列化的实现

<font size=4 style="font-weight:bold;background:yellow;">序列化/反序列化 Demo</font>

**实现序列化接口的实体类**

{% spoiler 展开查看折叠代码 %}

```java
// get/set/toString/构造方法 省略
public class Student implements Serializable {
    private static final long serialVersionUID = -4844659850613636533L;
    private String name;
    private Integer age;
    private String description;
}
```

{% endspoiler %}

**SerializableDemo**

{% spoiler 展开查看折叠代码 %}

```java
public class SerializableDemo {

    public static void serialize() throws IOException {
        Student student = new Student("Tom", 18, "I am Tom");
        ObjectOutputStream objectOutputStream =
                new ObjectOutputStream(new FileOutputStream(new File("student.txt")));
        objectOutputStream.writeObject(student);
        objectOutputStream.close();
        System.out.println("=========序列化成功！已经生成student.txt文件=========");
    }

    public static void deserialize() throws IOException, ClassNotFoundException {
        ObjectInputStream objectInputStream =
                new ObjectInputStream(new FileInputStream(new File("student.txt")));
        Student student = (Student) objectInputStream.readObject();
        objectInputStream.close();
        System.out.println("反序列化结果为：");
        System.out.println(student);
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        serialize();
        deserialize();
    }
}
```

{% endspoiler %}

<font size=4 style="font-weight:bold;background:yellow;">特殊情况</font>

- 被 static 修饰的字段是不会被序列化的：

  序列化保存的是对象的状态而非类的状态，所以会忽略 static 静态域；

- 实现 Serializable 接口的对象中被 transient 修饰符修饰的字段也是不会被序列化的：

  如果在序列化某个类的对象时，就是不希望某个字段被序列化（比如这个字段存放的是隐私值，如密码等），那这时就可以用 transient 修饰符来修饰该字段，在反序列化的时候该字段会是 null。

```java
private transient String password;	// 添加transient修饰的字段
```

```
=========序列化成功！已经生成student.txt文件=========
反序列化结果为：
Student(name=Tom, age=18, description=I am Tom, password=null)
```

## 1.1、serialVersionUID

> 在 idea 设置中搜索 serializable class without 'serialVersionUID' 并勾选，在创建类实现序列化接口时按 alt+enter可以自动生成 serialVersionUID。

**serialVersionUID 是序列化前后的唯一标识符，**serialVersionUID 在序列化时会随着对象一起被序列化，在反序列化时会被拿出来比较，如果反序列化的时候 id 不一致，会序列化失败。

比如是用上面的例子序列化，然后在反序列化的时候对 id 进行修改，会报错：

```java
Exception in thread "main" java.io.InvalidClassException: com.haining820.serializable.Student; local class incompatible: stream classdesc serialVersionUID = -4844659850613636522, local class serialVersionUID = -4844659850613636533
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:2001)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1848)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2158)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1665)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:501)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:459)
	at com.haining820.serializable.SerializableDemo.deserialize(SerializableDemo.java:31)
	at com.haining820.serializable.SerializableDemo.main(SerializableDemo.java:39)
```

如果在定义一个可序列化的类时，没有人为显式地给它定义一个 serialVersionUID 的话，则 Java 运行时环境会根据该类的各方面信息自动地为它生成一个默认的 serialVersionUID，一旦像上面一样**更改了类的结构或者信息**，则类的 **serialVersionUID 也会跟着变化**，所以，为了 serialVersionUID 的确定性，写代码时还是建议凡是 implements Serializable 的类，都最好**人为显式地为它声明一个 serialVersionUID 明确值**。

## 1.2、Externalizable

还可以实现 Externalizable 接口，重写其中的 writeExternal 和 readExternal 实现序列化，如果在这两个方法中指定了要序列化的字段或要反序列化的字段，则一定会序列化或反序列化，无关乎字段有没有被 transient 修饰。（目前工作中没看到过这种序列化方式）

```java
public interface Externalizable extends java.io.Serializable {
    void writeExternal(ObjectOutput out) throws IOException;
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

## 1.3、对序列化进行约束

使用 readObject 对反序列化进行约束<font color='#0000ff'>（理论上来说使用 writeObject 方法也能对序列化进行约束）</font>，底层是反射。

比如在A 处序列化后，将流进行传输，但是在传输过程中被篡改了，在反序列化的时候就会被约束拦住<font color='#0000ff'>（感觉意义不大，与密码加密不同，篡改为什么会改成奇葩值呢？这个篡改完全可以篡改在约束允许的范围之内，使得反序列化的时候能够通过）</font>

**实体类**

{% spoiler 展开查看折叠代码 %}

```java
private void writeObject(ObjectOutputStream objectOutputStream) throws IOException {
    objectOutputStream.defaultWriteObject();
    // 手工检查反序列化后学生成绩的有效性，若发现有问题，即终止操作！
    if (10 > age || 20 < age) {
        throw new IllegalArgumentException("writeObject: 学生年龄只能在10到20之间！");
    }
}

private void readObject(ObjectInputStream objectInputStream) throws IOException, ClassNotFoundException {
    // 调用默认的反序列化函数
    objectInputStream.defaultReadObject();
    // 手工检查反序列化后学生成绩的有效性，若发现有问题，即终止操作！
    if (10 > age || 20 < age) {
        throw new IllegalArgumentException("readObject: 学生年龄只能在10到20之间！");
    }
}
```

{% endspoiler %}

**结果**

{% spoiler 展开查看折叠代码 %}

```java
// writeObject
Exception in thread "main" java.lang.IllegalArgumentException: writeObject: 学生年龄只能在10到20之间！
	at com.haining820.serializable.Student.writeObject(Student.java:46)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	...

// readObject
=========序列化成功！已经生成student.txt文件=========
Exception in thread "main" java.lang.IllegalArgumentException: readObject: 学生年龄只能在10到20之间！
	at com.haining820.serializable.Student.readObject(Student.java:55)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	...
```

{% endspoiler %}

## 1.4、单例模式在序列化的失效问题

使用 readResolve 方法避免，估计不会经常用到，详情可见置顶链接