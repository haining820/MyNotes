## 函数式接口 Supplier

`java.util.function.Supplier<T>` 接口仅包含一个无参的方法：`T get()`

用来获取一个泛型参数指定类型的对象数据，`Supplier<T>` 接口被称之为生产型接口，传入一个泛型 T，将返回一个泛型 T。

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



## 函数式接口 Consumer

> https://blog.csdn.net/weixin_44230693/article/details/113847162

`java.util.function.Consumer` 接口则正好与 Supplier 接口相反，它不是生产一个数据，而是消费一个数据， 其数据类型由泛型决定。

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

如果一个方法的参数和返回值全都是 Consumer 类型，在消费数据的时候，可以首先做一个操作，然后再做一个操作，实现组合。由 Consumer 接口中的 default 方法 `andThen()` 可以实现。



## 泛型

> https://blog.csdn.net/weixin_42071874/article/details/82993262

方法的泛型优先级高于类的泛型

![泛型](D:\imgs\tempnote\泛型.png)