---
title: 注解与反射
date: 2022-08-14
tags: [注解,反射]
toc: true
---

# 注解 Annotation

## 1、什么是注解

- Annotation 是从 **JDK 5.0**开始引入的新技术。

<!--more-->

- Annotation的作用
  - 不是程序本身 , 可以对程序作出解释，这一点和注释（comment）没什么区别。
  - **可以被其他程序（比如:编译器等）读取。**
- Annotation的格式
  - 注解是以 ”@注释名“ 在代码中存在的，还可以添加一些参数值 , 例如：`@SuppressWarnings(value="unchecked")`
- Annotation在哪里使用?
  - 可以附加在package，class，method，field 等上面 , 相当于给他们添加了额外的辅助信息，我们可以通过反射机制实现对这些元数据的访问。



## 2、内置注解

- @Override
- 定义在 `java.lang.Override` 中，此注释只适用于修辞方法，表示一个方法声明打算重写超类中 的另一个方法声明。
- @Deprecated 定义在 `java.lang.Deprecated` 中，此注释可以用于修辞方法，属性，类，表示不鼓励程序员使用这样的元素，通常是因为它很危险或者存在更好的选择。
- @SuppressWarnings
  - 定义在`java.lang.SuppressWarnings`中，用来抑制编译时的警告信息。
  - 与前两个注释有所不同，你需要添加一个参数才能正确使用，这些参数都是已经定义好了的，我们选择性的使用就好了。
    - `@SuppressWarnings("all")`
    - `@SuppressWarnings("unchecked")`
    - `@SuppressWarnings(value={"unchecked","deprecation"})`
    - 等等 .....

```java
//测试内置注解
//所有类默认继承Object类
public class Test01 extends Object {
    
    //@Override 表示方法重写
    @Override
    public String toString() {
    	return super.toString();
    }
    
    //@Deprecated 表示方法过时了,不建议使用,可能存在问题,并不是不能使用!
    @Deprecated
    public static void stop(){
    	System.out.println("测试 @Deprecated");
    }
    
    //@SuppressWarnings 抑制警告,可以传参数
    @SuppressWarnings("all")
    public void sw(){
    	List list = new ArrayList();
    }
    public static void main(String[] args) {
    	stop();
    }
}
```



## 3、元注解

-  元注解的作用就是负责注解其他注解，Java定义了4个标准的meta-annotation类型,他们被用来提供 对其他annotation类型作说明。
- 这些类型和它们所支持的类在`java.lang.annotation`包中可以找到。( @Target ，@Retention，@Documented，@Inherited ) 
  - @Target：用于描述注解的使用范围。（即：被描述的注解可以用在什么地方）
  
    ```xml
    @Target:注解的作用目标
    @Target(ElementType.TYPE) ——接口、类、枚举、注解
    @Target(ElementType.FIELD) ——字段、枚举的常量
    @Target(ElementType.METHOD) ——方法
    @Target(ElementType.PARAMETER) ——方法参数
    @Target(ElementType.CONSTRUCTOR) ——构造函数
    @Target(ElementType.LOCAL_VARIABLE) ——局部变量
    @Target(ElementType.ANNOTATION_TYPE) ——注解
    @Target(ElementType.PACKAGE) ——包
    ```
  
  - @Retention：表示需要在什么级别保存该注释信息，用于描述注解的生命周期。
    
    （SOURCE < CLASS < RUNTIME）
    
    ```java
    RetentionPolicy.SOURCE:这种类型的Annotations只在源代码级别保留,编译时就会被忽略,在class字节码文件中不包含
    RetentionPolicy.CLASS:这种类型的Annotations编译时被保留,默认的保留策略,在class文件中存在,但JVM将会忽略,运行时无法获得
    RetentionPolicy.RUNTIME:这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用
    ```
    
  - @Document：说明该注解将被包含在javadoc中。
  
  - @Inherited：说明子类可以继承父类中的该注解。

```java
package com.annotation;
import java.lang.annotation.*;
//测试元注解
public class Test02 {
    @MyAnnotation
    public void test(){
    }
}

//定义一个注解
//Target 表示注解可以用在什么地方
@Target(value = {ElementType.METHOD,ElementType.TYPE})
//Retention 表示注解在什么地方有效
//SOURCE < CLASS < RUNTIME
@Retention(value = RetentionPolicy.RUNTIME)
//Documented 该注解将被包含在javadoc中
@Documented
//Inherited 子类可以继承父类中的该注解
@Inherited
@interface MyAnnotation{
	//测试作用域,了解@Retention的概念
}
```



## 4、自定义注解

-  使用 @interface自定义注解时，自动继承了`java.lang.annotation.Annotation`接口
- 分析
  - @ interface用来声明一个注解，格式 : public @ interface 注解名 { 定义内容 }
  - 其中的每一个方法实际上是声明了一个配置参数。
  - 方法的名称就是参数的名称。
  - 返回值类型就是参数的类型（返回值只能是基本类型：Class，String，enum）。
  - 可以通过default来声明参数的默认值。
  - 如果只有一个参数成员，一般参数名为value。
  - 注解元素必须要有值，我们定义注解元素时，经常使用空字符串，0作为默认值 .

```java
package com.annotation;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
//测试自定义注解
public class Test3 {
    //注解可以显式的赋值,如果没有默认值,就要给注解赋值
    @MyAnnotation2(age = 18,name = "秦疆",id = 001,schools = {"西工大"})
    public void test() {}
    
    //只有一个参数,默认名字一般是value,使用可省略不写
    @MyAnnotation3("aaa")
    public void test2() {}
}

@Target(value = {ElementType.TYPE,ElementType.METHOD})
@Retention(value = RetentionPolicy.RUNTIME)
@interface MyAnnotation2{
    //注解的参数:参数类型+参数名()
    String name() default "";
    int age();
    int id() default -1; //默认值为-1,代表不存在
    String[] schools() default {"西部开源","狂神说Java"};
}

@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation3{
    //参数类型+参数名称()
    String value();
}

```



# 反射Reflection

因为有反射机制，Java变成了一门动态语言

## 1、静态 VS 动态语言

- 动态语言
  - 是一类在运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是在运行时代码可以根据某些条件改变自身结构。 
  - 主要动态语言：Object-C、C#、JavaScript、PHP、Python等。

```javascript
//体现动态语言的代码
function test() {
    var x = "var a=3;var b=5;alert(a+b)";
    eval(x);
}
```

- 静态语言
  - 与动态语言相对应的，运行时结构不可变的语言就是静态语言。如Java、C、C++。
  - Java不是动态语言，但Java可以称之为“准动态语言”。即Java有一定的动态性，我们可以利用 反射机制获得类似动态语言的特性。Java的动态性让编程的时候更加灵活！

## 2、Java Reflection

Reflection（反射）是Java被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取 得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

```java
 Class c = Class.forName("java.lang.String")
```

==加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子， 透过这个镜子看到类的结构，所以，我们形象的称之为：反射。==

![image-20210403095001244](annotation-reflex\image-20210403095001244.png)

```java
//什么叫反射
public class test01 {
    public static void main(String[] args) throws ClassNotFoundException {
        //通过反射获取类的class对象
        Class c1 = Class.forName("com.haining820.reflection.User");
        System.out.println(c1);
        Class c2 = Class.forName("com.haining820.reflection.User");
        Class c3 = Class.forName("com.haining820.reflection.User");
        Class c4 = Class.forName("com.haining820.reflection.User");
        //一个类在内存中只有一个class对象
        //一个类被加载后,类的整个结构都会被封装在class对象中
        System.out.println(c2.hashCode());
        System.out.println(c3.hashCode());
        System.out.println(c4.hashCode());
        /**
         * class com.haining820.reflection.User
         * 460141958
         * 460141958
         * 460141958
         * hashCode的值是一样的,说明是同一个对象
         */
    }
}
//实体类pojo,entity
class User{
    private String name;
    private int id;
    private int age;
    public int height;
    
    private void test(){}   //测试后面的getDeclaredMethods方法
    
    //此处省略有参/无参构造,get/set方法,toString方法
}
```



**Java反射机制提供的功能**

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解 生成动态代理
- ... ...

**Java反射优点和缺点**

优点：可以实现动态创建对象和编译，体现出很大的灵活性 ! 

缺点：对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满 足我们的要求。这类操作总是慢于直接执行相同的操作。

## 3、反射相关的主要API

- java.lang.Class : 代表一个类
- java.lang.reflect.Method : 代表类的方法
- java.lang.reflect.Field : 代表类的成员变量
- java.lang.reflect.Constructor : 代表类的构造器
- .......

## 4、Class类

在Object类中定义了以下的方法，此方法将被所有子类继承

```java
 public final Class getClass();
```

以上的方法返回值的类型是一个Class类，此类是Java反射的源头，实际上所谓反射从程序的运行结果来看也很好理解，即：可以通过对象反射求出类的名称。

![image-20210403100205840](annotation-reflex\image-20210403100205840.png)

对象照镜子后可以得到的信息：某个类的属性、方法和构造器、某个类到底实现了哪些接口。对于每个 类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个结构 (class/interface/enum/annotation/primitive type/void/[])的有关信息。 

- Class 本身也是一个类
- Class 对象只能由系统建立对象
- 一个加载的类在 JVM 中只会有一个Class实例
- 一个Class对象对应的是一个加载到JVM中的一个.class文件
- 每个类的实例都会记得自己是由哪个 Class 实例所生成
- 通过Class可以完整地得到一个类中的所有被加载的结构
- **Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有先获得相应的Class对象**

**Class类的常用方法**

| 方法名                                  | 功能说明                                                     |
| --------------------------------------- | ------------------------------------------------------------ |
| **static ClassforName(String name)**    | 返回指定类名name的Class对象                                  |
| **Object newInstance()**                | 调用缺省构造函数，返回Class对象的一个实例                    |
| getName()                               | 返回此Class对象所表示的实体（类，接口，数组类或 void）的名称。 |
| Class getSuperClass()                   | 返回当前Class对象的父类的Class对象                           |
| Class[] getinterfaces()                 | 获取当前Class对象的接口                                      |
| ClassLoader getClassLoader()            | 返回该类的类加载器                                           |
| Constructor[] getConstructors()         | 返回一个包含某些Constructor对象的数组                        |
| Method getMothed(String name,Class.. T) | 返回一个Method对象，此对象的形参类型为paramType              |
| Field[] getDeclaredFields()             | 返回Field对象的一个数组                                      |

获取Class类的实例

a）若已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高。

```java
Class clazz = Person.class;
```

b）已知某个类的实例，调用该实例的getClass()方法获取Class对象

```java
 Class clazz = person.getClass();
```

c）已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法`forName()`获取，可能抛出 `ClassNotFoundException`

```java
 Class clazz = Class.forName("demo01.Student");
```

d）内置基本数据类型可以直接用 **类名.class**

e）还可以利用ClassLoader我们之后讲解

```java
public class test02 {

    public static void main(String[] args) throws ClassNotFoundException {
        Person person = new Student();
        System.out.println("这个人是:" + person.name);
        //方式一: 通过对象获得
        Class c1 = person.getClass();
        System.out.println(c1.hashCode());

        //方式二: forName获得
        Class c2 = Class.forName("com.haining820.reflection.Student");

        System.out.println(c2.hashCode());
        //方式三: 通过类名.class
        Class c3 = Student.class;
        System.out.println(c3.hashCode());

        //方式四: 基本内置类型的包装类都有一个TYPE属性
        Class c4 = Integer.class;
        System.out.println(c4.hashCode());
        System.out.println(c4);//class java.lang.Integer

        //获得父类的类型
        Class c5 = c1.getSuperclass();
        System.out.println(c5);//class com.haining820.reflection.Person

    }

}

/**
 * 这个人是:学生
 * 460141958
 * 460141958
 * 460141958    //hashCode值相等,其实都是同一个对象
 * 1163157884
 * class java.lang.Integer
 * class com.haining820.reflection.Person
 */


class Person{
    public String name;
    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }
}

class Student extends Person {
    public Student() {
        this.name = "学生";
    }
}

class Teacher extends Person {
    public Teacher() {
        this.name = "老师";
    }
}
```



哪些类型可以有Class对象？

- class：外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类。
- interface：接口
- []：数组
- enum：枚举
- annotation：注解@interface
- primitive type：基本数据类型
- void

```java
package com.haining820.reflection;

import java.lang.annotation.ElementType;

//所有类型的class对象
public class test03 {
    public static void main(String[] args) {
        Class c1 = Object.class;  //类
        Class c2 = Comparable.class;  //接口
        Class c3 = String[].class;  //一维数组
        Class c4 = int[][].class;  //二维数组
        Class c5 = Override.class;  //注解
        Class c6 = ElementType.class;  //枚举
        Class c7 = Integer.class;  //基本数据类型
        Class c8 = void.class;  //void
        Class c9 = Class.class;  //Class

        System.out.println(c1);
        System.out.println(c2);
        System.out.println(c3);
        System.out.println(c4);
        System.out.println(c5);
        System.out.println(c6);
        System.out.println(c7);
        System.out.println(c8);
        System.out.println(c9);

        /**
         *         class java.lang.Object
         *         interface java.lang.Comparable
         *         class [Ljava.lang.String;
         *         class [[I
         *         interface java.lang.Override
         *         class java.lang.annotation.ElementType
         *         class java.lang.Integer
         *         void
         *         class java.lang.Class
         */

        //只要元素类型与维度一样,就是同一个Class
        int[] a = new int[10];
        int[] b = new int[100];
        System.out.println(a.getClass().hashCode());
        System.out.println(b.getClass().hashCode());
    }
}

```



## 5、Java内存分析

<img src="annotation-reflex\image-20210403101146431.png" alt="image-20210403101146431" style="zoom: 67%;" />



### 5.1、类的加载过程

<img src="annotation-reflex\image-20210403101234510.png" alt="image-20210403101234510" style="zoom:67%;" />



### 5.2、类的加载与ClassLoader的理解

- 加载
  - 将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构， 然后生成一个代表这个类的`java.lang.Class`对象。
- 链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。
  - 验证：确保加载的类信息符合JVM规范，没有安全方面的问题
  - 准备：正式为类变量（static）分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配。
  - 解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。
- 初始化
  - 执行类构造器 `<clinit>()` 方法的过程。类构造器 `<clinit>()` 方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。（类构造器是构造类信息的，不是构造该类对象的构造器）
  - 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
  - 虚拟机会保证一个类的()方法在多线程环境中被正确加锁和同步。

```java
package com.haining820.reflection;

public class Test13 {
    public static void main(String[] args) {
        A a = new A();
        System.out.println(A.m);
    }
}

class A {
    static {
        System.out.println("A类静态代码块初始化");
        m = 300;
    }

    static int m = 100;

    public A(){
        System.out.println("A类的无参构造初始化");
    }
}

/*
	A类静态代码块初始化
    A类的无参构造初始化
    100
*/

/*
    执行顺序:
    1.加载到内存,会产生一个对应的class对象
    2.链接,链接结束后 m = 0
    3.初始化
        <clinit>(){
            System.out.println("A类静态代码块初始化");
            m = 300;
            m = 100;
            //先执行父类的静态代码块和静态变量初始化
            //静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。
        }
        最终 m = 100
 */
```



### 5.3、什么时候会发生类初始化？ 

- 类的主动引用（一定会发生类的初始化）
  - 当虚拟机启动，先初始化main方法所在的类
  - new一个类的对象
  - 调用类的静态成员（除了final常量）和静态方法
  - 使用java.lang.reflect包的方法对类进行反射调用
  - 当初始化一个类，如果其父类没有被初始化，则先会初始化它的父类
- 类的被动引用（不会发生类的初始化）
  - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静 态变量，不会导致子类初始化
  - 通过数组定义类引用，不会触发此类的初始化
  - 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

```java
package com.haining820.reflection;

public class test04 {
    static{
        System.out.println("main类被加载");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        //1.主动引用
//        Son son = new Son();
        /**
         * main类被加载
         * 父类被加载
         * 子类被加载
         */

        //反射也会产生主动引用
//        Class.forName("com.haining820.reflection.Son");
        /**
         * main类被加载
         * 父类被加载
         * 子类被加载
         */

        //不会产生类的引用的方法
//        System.out.println(Son.b);
        /**
         * main类被加载
         * 父类被加载
         * 2
         */

        //数组
//        Son[] sons = new Son[5];  //main类被加载

        System.out.println(Son.M);
        /**
         * main类被加载
         * 1
         */
    }
}

class Father{
    static int b = 2;
    static{
        System.out.println("父类被加载");
    }
}

class Son extends Father{
    static{
        System.out.println("子类被加载");
    }
    static int m = 100;
    static final int M = 1;

}
```

### 5.4、类加载器的作用

- 类加载的作用：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数 据结构，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入 口。
- 类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持 加载（缓存）一段时间。不过JVM垃圾回收机制可以回收这些Class对象

![image-20210403101952774](annotation-reflex\image-20210403101952774.png)

- 类加载器作用是用来把类(class)装载进内存的，JVM 规范定义了如下类型的类的加载器。

![image-20210403102024372](annotation-reflex\image-20210403102024372.png)

```java
package com.haining820.reflection;

public class test05 {
    public static void main(String[] args) throws ClassNotFoundException {
        //获取系统类的加载器
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);

        //获取系统类加载器的父类加载器-->扩展类加载器
        ClassLoader parent = classLoader.getParent();
        System.out.println(parent);

        //获取扩展类加载器的父类加载器-->根加载器(C/C++)
        ClassLoader grandparent = parent.getParent();
        System.out.println(grandparent);

        //测试当前类是哪个类加载器加载的
        ClassLoader loader = Class.forName("com.haining820.reflection.test05").getClassLoader();
        System.out.println(loader);

        //测试JDK内置类是哪个类加载器加载的
        loader = Class.forName("java.lang.Object").getClassLoader();
        System.out.println(loader);

        //如何获得系统加载类可以加载的路径
        String property = System.getProperty("java.class.path");
        System.out.println(property);

        //双亲委派机制

        /**
         * sun.misc.Launcher$AppClassLoader@14dad5dc
         * sun.misc.Launcher$ExtClassLoader@1b6d3586
         * null(无法直接获取)
         * sun.misc.Launcher$AppClassLoader@14dad5dc
         * null
         * E:\Java\jdk-1.8.0_60\jre\lib\charsets.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\deploy.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\access-bridge-64.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\cldrdata.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\dnsns.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\jaccess.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\jfxrt.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\localedata.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\nashorn.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\sunec.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\sunjce_provider.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\sunmscapi.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\sunpkcs11.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\ext\zipfs.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\javaws.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\jce.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\jfr.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\jfxswt.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\jsse.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\management-agent.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\plugin.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\resources.jar;
         * E:\Java\jdk-1.8.0_60\jre\lib\rt.jar;
         * E:\IDEA-project\reflection\out\production\reflection;
         * D:\Tools\IntelliJ IDEA 2020.1.4\lib\idea_rt.jar
         */
    }
}

```



## 6、创建运行时类的对象

**通过反射获取运行时类的完整结构**

Field、Method、Constructor、Superclass、Interface、Annotation

- 实现的全部接口
- 所继承的父类
- 全部的构造器
- 全部的方法
- 全部的Field
- 注解 
- ... ...

小结

- 在实际的操作中，取得类的信息的操作代码，并不会经常开发。 
- 一定要熟悉 java.lang.reflect 包的作用，反射机制。
- 如何取得属性、方法、构造器的名称，修饰符等。

```java
package com.haining820.reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Test06 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException {
        Class c1 = Class.forName("com.haining820.reflection.User");

        //获得类的名字
        System.out.println(c1.getName());  //获得包名+类名
        System.out.println(c1.getSimpleName());  //获得类名
        System.out.println("====================");
        
        Field[] fields = c1.getFields();  //只能找到public属性
        for (Field field : fields) {
            System.out.println(field);
        }
        System.out.println("====================");
        
        fields = c1.getDeclaredFields();  //能找到全部的属性
        for (Field field : fields) {
            System.out.println(field);
        }
        System.out.println("====================");
        
        /**
         * com.haining820.reflection.User
         * User
         * ====================
         * public int com.haining820.reflection.User.height
         * ====================
         * private java.lang.String com.haining820.reflection.User.name
         * private int com.haining820.reflection.User.id
         * private int com.haining820.reflection.User.age
         * public int com.haining820.reflection.User.height
         * ====================
         */

        
         
        //获得指定属性的值
        Field name = c1.getDeclaredField("name");
        System.out.println(name); 
        /**
         *private java.lang.String com.haining820.reflection.User.name
         */

        

        //获得类的方法
        Method[] methods = c1.getMethods();  //获得本类及其父类的全部public方法
        for (Method method : methods) {
            System.out.println("getMethods: "+method);
        }
        System.out.println("========================================");
        Method[] declaredMethods = c1.getDeclaredMethods(); //获得本类所有方法,public,private
        for (Method declaredMethod : declaredMethods) {
            System.out.println("getDeclaredMethods: "+declaredMethod);
        }
        /**
         * getMethods: public java.lang.String com.haining820.reflection.User.toString()
         * getMethods: public java.lang.String com.haining820.reflection.User.getName()
         * getMethods: public int com.haining820.reflection.User.getId()
         * getMethods: public void com.haining820.reflection.User.setName(java.lang.String)
         * getMethods: public void com.haining820.reflection.User.setAge(int)
         * getMethods: public int com.haining820.reflection.User.getAge()
         * getMethods: public void com.haining820.reflection.User.setId(int)
         * getMethods: public final void java.lang.Object.wait() throws java.lang.InterruptedException
         * getMethods: public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
         * getMethods: public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
         * getMethods: public boolean java.lang.Object.equals(java.lang.Object)
         * getMethods: public native int java.lang.Object.hashCode()
         * getMethods: public final native java.lang.Class java.lang.Object.getClass()
         * getMethods: public final native void java.lang.Object.notify()
         * getMethods: public final native void java.lang.Object.notifyAll()
         * ========================================
         * getDeclaredMethods: public java.lang.String com.haining820.reflection.User.toString()
         * getDeclaredMethods: public java.lang.String com.haining820.reflection.User.getName()
         * getDeclaredMethods: public int com.haining820.reflection.User.getId()
         * getDeclaredMethods: public void com.haining820.reflection.User.setName(java.lang.String)
         * getDeclaredMethods: public void com.haining820.reflection.User.setAge(int)
         * getDeclaredMethods: public int com.haining820.reflection.User.getAge()
         * getDeclaredMethods: public void com.haining820.reflection.User.setId(int)
         * getDeclaredMethods: private void com.haining820.reflection.User.test()  这是私有方法！
         */
        
        
        
        //获得指定的方法
        Method getName = c1.getMethod("getName", null);
        Method setName = c1.getMethod("setName", String.class);  //通过参数类型判断哪个具体的方法(),可能会有重载
        System.out.println(getName);
        System.out.println(setName);
        /**
         * public java.lang.String com.haining820.reflection.User.getName()
         * public void com.haining820.reflection.User.setName(java.lang.String)
         */

        
        
        //获取类的构造方法
        Constructor[] constructors = c1.getConstructors();  //public
        for (Constructor constructor : constructors) {
            System.out.println(constructor);
        }
        Constructor[] declaredConstructors = c1.getDeclaredConstructors();  //全部
        for (Constructor declaredConstructor : declaredConstructors) {
            System.out.println("#: "+declaredConstructor);
        }
        /**
         * public com.haining820.reflection.User()
         * public com.haining820.reflection.User(java.lang.String,int,int)
         * #: public com.haining820.reflection.User()
         * #: public com.haining820.reflection.User(java.lang.String,int,int)
         */

        
        
        //获取指定的构造器
        Constructor constructor = c1.getConstructor(String.class, int.class, int.class);
        System.out.println("获取指定的构造器:"+constructor); 
         /**
         * 获取指定的构造器:public com.haining820.reflection.User(java.lang.String,int,int)
         */
    }
}

```



## 7、有了Class对象，能做什么?

- 创建类的对象：调用Class对象的newInstance()方法 
  - 类必须有一个无参数的构造器。 
  - 类的构造器的访问权限需要足够。
- 思考？难道没有无参的构造器就不能创建对象了吗？只要在操作的时候明确的调用类中的构造器， 并将参数传递进去之后，才可以实例化操作。
- 步骤如下： 
  - 通过Class类的getDeclaredConstructor(Class … parameterTypes)取得本类的指定形参类型 的构造器
  - 向构造器的形参中传递一个对象数组进去，里面包含了构造器中所需的各个参数。
  - 通过Constructor实例化对象

调用指定的方法

- 通过反射，调用类中的方法，通过Method类完成。

  - 通过Class类的`getMethod(String name,Class…parameterTypes)`方法取得一个Method对 象，并设置此方法操作时所需要的参数类型。 

  - 之后使用`Object invoke(Object obj, Object[] args)`进行调用，并向方法中传递要设置的obj对 象的参数信息。

    ![image-20210403102703470](annotation-reflex\image-20210403102703470.png)

  - Object 对应原方法的返回值，若原方法无返回值，此时返回 null 

  - 若原方法若为静态方法，此时形参Object obj可为 null 

  - 若原方法形参列表为空，则Object[] args为 null 

  - 若原方法声明为private,则需要在调用此invoke()方法前，显式调用方法对象的 setAccessible(true)方法，将可访问private的方法。

```java
package com.haining820.reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

//通过反射动态的创建对象
public class Test07 {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        //获得Class对象
        Class c1 = Class.forName("com.haining820.reflection.User");

        //构造一个对象
//        User user = (User)c1.newInstance();  //本质上是调用了User的无参构造器
//        System.out.println(user);


        //通过构造器创建对象
//        Constructor constructor = c1.getDeclaredConstructor(String.class, int.class, int.class);
//        User user2 = (User)constructor.newInstance("yuhaiyang", 001, 18);
//        System.out.println(user2);

        //通过反射调用普通构造方法
        User user3 = (User)c1.newInstance();
        //通过反射获取setName方法
        Method setName = c1.getDeclaredMethod("setName", String.class);
//        user3.setName("于海洋");
        setName.invoke(user3,"于海洋777");  //invoke,激活
        System.out.println(user3.getName());

        System.out.println("**************************************");
        //通过反射获取getName方法
        Method getName = c1.getDeclaredMethod("getName", null);
        String user3Name = (String)getName.invoke(user3,null);
        System.out.println(user3Name);

        //通过反射操作属性
        System.out.println("==================================");
        User user4 = (User)c1.newInstance();
        Field name = c1.getDeclaredField("name");
        //不能直接操作私有属性,需要关闭程序的安全检测
        //属性或者方法.setAccessible(true)
        name.setAccessible(true);  //取消属性的安全检测
        name.set(user4,"于海洋666");
        System.out.println(user4.getName());


    }
}

```



## 8、setAccessible

- Method和Field、Constructor对象都有setAccessible()方法。
- setAccessible作用是启动和禁用访问安全检查的开关。
- 参数值为true则指示反射的对象在使用时应该取消Java语言访问检查。
- 提高反射的效率。如果代码中必须用反射，而该句代码需要频繁的被调用，那么请设置为true。
- 使得原本无法访问的私有成员也可以访问
- 参数值为false则指示反射的对象应该实施Java语言访问检查

```java
package com.haining820.reflection;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

//分析性能问题
public class Test10_1 {
    //普通方式调用
    public static void test011(){
        User user = new User();

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            user.getName();
        }
        long endTime = System.currentTimeMillis();

        System.out.println("普通方法,10亿次"+(endTime-startTime)+"ms");
    }

    //反射方式调用

    public static void test022() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException {
        User user = new User();
        Class c1 = user.getClass();
        Method getName = c1.getDeclaredMethod("getName", null);

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user,null);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("反射方式调用,10亿次"+(endTime-startTime)+"ms");
    }


    //反射方式调用,关闭检测


    public static void test033() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException {
        User user = new User();
        Class c1 = user.getClass();
        Method getName = c1.getDeclaredMethod("getName", null);
        getName.setAccessible(true);

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user,null);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("反射方式调用,关闭检测,10亿次"+(endTime-startTime)+"ms");
    }


    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        test011();
        test022();
        test033();
        /**
         * 普通方法,10亿次3ms
         * 反射方式调用,10亿次1870ms
         * 反射方式调用,关闭检测,10亿次1290ms
         */
    }
}
```



## 9、反射操作泛型

- Java采用泛型擦除的机制来引入泛型，Java中的泛型仅仅是给编译器javac使用的,确保数据的安全性 和免去强制类型转换问题，但是，一旦编译完成 , 所有和泛型有关的类型全部擦除 
- 为了通过反射操作这些类型，Java新增了 ParameterizedType，GenericArrayType，TypeVariable 和 WildcardType 几种类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型。 
- ParameterizedType：表示一种参数化类型，比如Collection
- GenericArrayType：表示一种元素类型是参数化类型或者类型变量的数组类型 
- TypeVariable：是各种类型变量的公共父接口
- WildcardType：代表一种通配符类型表达式

```java
public class Test11 {

    //通过反射获取泛型
    public void test01(Map<String,User> Map, List<User> list){
        System.out.println("test01");
    }
    public Map<String,User> test02(){
        System.out.println("test02");
        return null;
    }

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = Test11.class.getMethod("test01", Map.class, List.class);
        Type[] genericParameterTypes = method.getGenericParameterTypes();  //参数类型
        for (Type genericParameterType : genericParameterTypes) {
//            System.out.println("#"+genericParameterType);
            if(genericParameterType instanceof ParameterizedType){//如果是参数化类型的话
                //获得真实的参数信息
                Type[] actualTypeArguments = ((ParameterizedType) genericParameterType).getActualTypeArguments();
                for (Type actualTypeArgument : actualTypeArguments) {
                    System.out.println(actualTypeArgument);
                    //可以将泛型中的参数信息打印出来
                }
            }
        }
        /**
         * #java.util.Map<java.lang.String, com.haining820.reflection.User>
         * #java.util.List<com.haining820.reflection.User>
         */
        /**
         * class java.lang.String
         * class com.haining820.reflection.User
         * class com.haining820.reflection.User
         */

        method = Test11.class.getMethod("test02");
        Type genericReturnType = method.getGenericReturnType();
        if(genericReturnType instanceof ParameterizedType){//如果是参数化类型的话
            //获得真实的参数信息
            Type[] actualTypeArguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
            for (Type actualTypeArgument : actualTypeArguments) {
                System.out.println(actualTypeArgument);
                //可以将泛型中的参数信息打印出来
            }
        }
        /**
         * class java.lang.String
         * class com.haining820.reflection.User
         */

    }

}
```



## 10、反射操作注解

- getAnnotations
- getAnnotation



## 11、练习ORM

- 了解什么是ORM ? 

  Object relationship Mapping --> 对象关系映射

![image-20210403103129359](annotation-reflex\image-20210403103129359.png)

- 类和表结构对应

- 属性和字段对应

- 对象和记录对应

要求 : 利用注解和反射完成类和表结构的映射关系

```java
package com.haining820.reflection;

import java.lang.annotation.*;
import java.lang.reflect.Field;

//练习反射操作注解
public class Test12 {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("com.haining820.reflection.Student2");
        //通过反射获得注解
        Annotation[] annotations = c1.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
            //@com.haining820.reflection.TableYu(value=db_student)
        }

        //获得注解value的值
        TableYu tableYu = (TableYu)c1.getAnnotation(TableYu.class);
        String value = tableYu.value();
        System.out.println(value);  //db_student

        //获得指定类的注解
        Field name = c1.getDeclaredField("name");
        FieldYu annotation = name.getAnnotation(FieldYu.class);
        System.out.println(annotation.columnName());
        System.out.println(annotation.type());
        System.out.println(annotation.length());
        /**
         * db_name
         * varchar
         * 3
         */
        System.out.println("==================");
        Field id = c1.getDeclaredField("id");
        FieldYu annotation_id = id.getAnnotation(FieldYu.class);
        System.out.println(annotation_id.columnName());
        System.out.println(annotation_id.type());
        System.out.println(annotation_id.length());
        /**
         * db_id
         * int
         * 10
         */

        System.out.println("==================");
        Field age = c1.getDeclaredField("age");
        FieldYu annotation_age = age.getAnnotation(FieldYu.class);
        System.out.println(annotation_age.columnName());
        System.out.println(annotation_age.type());
        System.out.println(annotation_age.length());
        /**
         * db_age
         * int
         * 10
         */

    }

}

@TableYu("db_student")
class Student2 {
    @FieldYu(columnName = "db_id", type = "int", length = 10)
    private int id;
    @FieldYu(columnName = "db_age", type = "int", length = 10)
    private int age;
    @FieldYu(columnName = "db_name", type = "varchar", length = 3)
    private String name;

   //此处省略有参/无参构造,get/set方法,toString方法
}

//类名的注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TableYu{
    String value();
}

//属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface FieldYu{
    String columnName();
    String type();
    int length();
}
```

