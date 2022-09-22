---
title: java源码
date: 2022-08-14
tags: [Java,源码]
toc: true
---

# Java 中的集合容器简介

## 1、Java 中常用的容器类介绍

- Collection 接口和 Map 接口是所有集合框架的父接口。

- Collection 集合的子接口有 List、Set、Queue 三种子接口，比较常用的是 List 接口 和 Set 接口。

  List 接口 的实现类主要有：ArrayList（线程不安全）、LinkedList、Stack 以及 Vector（线程安全）等；

  Set 接口 的实现类主要有：HashSet、TreeSet、LinkedHashSet 等。

  Queue 接口：BlockingQueue

- Map 接口 的实现类主要有：HashMap、TreeMap、Hashtable、LinkedHashMap、ConcurrentHashMap 以及 Properties 等。

<!--more-->

## 2、List，Set，Map三者的区别？这三个接口存取元素时，各有什么特点？

**List**

- 一个有序（元素存入集合的顺序和取出的顺序一致）容器。
- 元素可以重复，可以插入多个 null 元素，元素都有索引。

**Set**

- 一个无序（存入和取出顺序有可能不一致）容器。
- 不可以存储重复元素，只允许存入一个 null 元素，必须保证元素唯一性。

**Map**

- 一个键值对集合，存储 key 和 value 之间的映射。 
- key 无序，唯一；value 允许重复。从 Map 集合中检索元素时，只要给出键对象，就会返回对应的值对象。

> 注意：Map 接口不是 Collection 的子接口。Map 和 Collection 是同级的概念。

## 3、集合框架底层数据结构

**Collection**

- List
  - Arraylist： Object数组
  - Vector： Object数组
  - LinkedList： 双向循环链表

- Set
  - HashSet（无序，唯一）：基于 HashMap 实现的，底层采用 HashMap 来保存元素。
  
    ```java
    // 源码中HashSet的构造函数
    public HashSet() {
        map = new HashMap<>();
    }
    // 增加元素，本质就是map，只是key无法重复！
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    ```
  
  - LinkedHashSet： LinkedHashSet 继承与 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 Hashmap 实现一样，不过还是有一点点区别。
  
  - TreeSet（有序，唯一）： 红黑树（自平衡的排序二叉树）

**Map**

- HashMap：
  - JDK1.8 之前 HashMap 由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。
  - JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。
- LinkedHashMap
  - LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构，即由数组和链表或红黑树组成。
  - 另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- HashTable
  - 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。
- TreeMap： 红黑树

# ArrayList

## 1、ArrayList的扩容机制？

ArrayList 不是线程安全的。

说到扩容机制首先要从构造方法说起，在源码中 ArrayList 的构造有三种方法，如果没有给定初始容量会创建一个空数组；给定初始容量了则会创建一个给定容量的数组；还可以构造包含指定 collection 元素的 ArrayList。构造完成后，向列表中添加元素，如果元素过多就会发生溢出现象，这个时候就需要扩容。

与添加元素有关的是 ArrayList 源码中的 add 方法，add 方法实际上就是为数组赋值，但是在赋值之前会调用 **ensureCapacityInternal** 方法进行与扩容有关的操作。

- 在 **ensureCapacityInternal** 方法中将传入参数和默认容量进行比较，调用`Math.max()`方法选取较大值作为新的容量。
- 调用 **ensureCapacityInternal** 方法就一定会调用 **ensureExplicitCapacity** 方法，这个方法用于判断是否需要进行扩容操作，参数就是之前比较后的 minCapacity 。如果 `minCapacity - elementData.length > 0`（即 minCapacity 大于当前数组的长度）就会调用 **grow** 方法进行扩容操作。 

> 比如在初始化的时候选取无参构造，这时候初始化的是一个空数组，在添加元素时调用 **ensureCapacityInternal** 方法时在内部就会将 1 和 DEFAULT_CAPACITY 进行比较（DEFAULT_CAPACITY是一个全局变量，是默认的初始容量，大小为10），会将 minCapacity 设置为 10 ；然后进入 **ensureExplicitCapacity(minCapacity)** 方法判断 `minCapacity - elementData.length > 0` （10 - 1 > 0）成立；然后进入 **grow(minCapacity)** 方法进行扩容。
>
> 当再次添加数据时，minCapacity 为2，3，4 ... 直至10 都不会满足扩容条件，都不会调用 **grow** 方法，当添加第11个元素时，会再次进入 **grow** 方法进行扩容。

- **grow** 方法是 ArrayList 进行扩容的核心方法，首先取 oldCapacity 为原数组的长度，然后设置 newCapacity 为oldCapacity 本身再加上采用位运算将其右移一位的结果（`int newCapacity = oldCapacity + (oldCapacity >> 1);`位运算效率高，采用位运算会加快运行的速度），实际上扩容后的容量就是扩容前的 1.5 倍。

  然后对扩容后的容量与最小所需容量进行比较，如果还是不够的话会直接将扩容后的 newCapacity 设置为最小所需容量。

  然后对最小所需容量与 MAX_ARRAY_SIZE （MAX_ARRAY_SIZE 的值为 Integer.MAX_VALUE - 8）进行比较，如果比MAX_ARRAY_SIZE 还大的话会进入 **hugeCapacity** 方法。否则的话，就进入最终的`Arrays.copyOf()`方法进行数组的扩容操作。

- `Arrays.copyOf()`方法调用的是本地方法`System.arraycopy()`，原数组对象仍是原数组对象，不变，该拷贝不会影响原来的数组。`Arrays.copyOf()`方法传回的数组是新的数组对象，改变传回数组中的元素值，不会影响原来的数组；该方法的第二个参数是指定要建立的新数组长度，如果新数组的长度超过原数组的长度，则多出来的部分为数组的默认初始值 0 。

- `System.arraycopy()`方法有五个参数：源数组、源数组中的起始位置、目标数组、目标数组中的起始位置、要复制的数组元素的数量，通过这五个参数的规定实现数组复制的功能。

> ArrayList 中大量调用了`System.arraycopy()` 和 `Arrays.copyOf()`这两个方法，`grow()`、`add(int index, E element)`、`toArray()` 等方法中都用到了该方法。
>
> `System.arraycopy()`需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置。 `Arrays.copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

-  在**hugeCapacity** 方法中，比较 minCapacity 和 MAX_ARRAY_SIZE，如果 minCapacity 大于最大容量 MAX_ARRAY_SIZE，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

-  ArrayList 源码中有一个**ensureCapacity**方法，该方法在 ArrayList 内部没有被调用过，是提供给用户调用的。向 ArrayList 添加大量元素之前可以先使用**ensureCapacity** 方法，可以减少在增加元素时的重新分配次数。



**从源码上来看**

1. 源码中ArrayList 的构造方式，ArrayList有三种构造方法。


- `ArrayList() `

  构造一个初始容量为 10 的空列表。 

- `ArrayList(int initialCapacity) `

  构造一个具有指定初始容量的空列表。 

- `ArrayList(Collection<? extends E> c) `

  构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。 

  ```java
  // 这是数组默认的初始容量
  private static final int DEFAULT_CAPACITY = 10;
  // 出现在需要用到空数组的地方，其中一处就是使用自定义初始容量构造方法时，如果指定初始容量为0的时候就会返回。
  private static final Object[] EMPTY_ELEMENTDATA = new Object[0];
  // 使用默认构造方法时候返回的空数组，如果第一次添加数据的话那么数组长度扩容为 DEFAULT_CAPACITY=10。
  private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];
  
  // 默认构造
  public ArrayList() {
      this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
  }
  
  // 自定义初始化容量
  public ArrayList(int initialCapacity) {
      if (initialCapacity > 0) {
          this.elementData = new Object[initialCapacity];
      } else {
          if (initialCapacity != 0) {
              throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
          }
          this.elementData = EMPTY_ELEMENTDATA;
      }
  }
  
  // 构造包含指定元素的ArrayList
  public ArrayList(Collection<? extends E> c) {
      this.elementData = c.toArray();
      if ((this.size = this.elementData.length) != 0) {
          if (this.elementData.getClass() != Object[].class) {
              this.elementData = Arrays.copyOf(this.elementData, this.size, Object[].class);
          }
      } else {
          this.elementData = EMPTY_ELEMENTDATA;
      }
  }
  ```

  **以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10。** 

2. 扩容操作的具体的实现

- 先看 ArrayList 的 add 方法的实现。

  ```java
  // 将指定的元素追加到此列表的末尾
  public boolean add(E e) {
      //添加元素之前，先调用ensureCapacityInternal方法
      this.ensureCapacityInternal(this.size + 1);
      //可以看出ArrayList添加元素的实质就相当于为数组赋值
      this.elementData[this.size++] = e;
      return true;
  }
  ```

- ArrayList的扩容核心在于 `ensureCapacityInternal()` 方法**（ensureCapacityInternal，确保内部容量）**。

  ```java
  private void ensureCapacityInternal(int minCapacity) {
      // 假设初始的ArrayList为空，当要add第1个元素时var1为1，在Math.max()方法比较后，var1为10。
      if (this.elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
          // 将传入参数与DEFAULT_CAPACITY取最大值处理
          minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
      }
      this.ensureExplicitCapacity(minCapacity);
  }
  ```

- `ensureExplicitCapacity()` 方法**（ensureExplicitCapacity，确保明确容量）**。

  ```java
  // 判断是否需要扩容
  private void ensureExplicitCapacity(int minCapacity) {
      modCount++;
      if (minCapacity - this.elementData.length > 0) {
          // 调用grow方法进行扩容
          grow(minCapacity);
      }
  }
  ```

  - 当 add 第1个元素到 ArrayList 时，`elementData.length`为 0（因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为10。此时，`minCapacity- elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法进行扩容。
  - 当add第2个元素时，minCapacity 为 2，此时elementData.length 在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 `grow(minCapacity)` 方法。添加第3、4 ...到第10个元素时，依然不会执行grow方法，数组容量都为10。
  - 直到添加第11个元素，minCapacity（大小增至11）比elementData.length（为10）要大，要调用grow方法进行扩容。

- `grow()` 方法，ArrayList扩容的核心方法。

  ```java
      /**
       * 要分配的最大数组大小
       */
      private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
  
      private void grow(int minCapacity) {
          // oldCapacity为旧容量，newCapacity为新容量
          // 将oldCapacity 右移一位，其效果相当于oldCapacity/2
          // 位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍。
          int oldCapacity = elementData.length;
          int newCapacity = oldCapacity + (oldCapacity >> 1);
          // 然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量。
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
         // 如果新容量大于 MAX_ARRAY_SIZE，就会执行hugeCapacity()方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
         // 如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则新容量大小则为 MAX_ARRAY_SIZE 即为 Integer.MAX_VALUE - 8。
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // minCapacity is usually close to size, so this is a win:
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
  
  ```

- `hugeCapacity()` 方法

  从上面 `grow()` 方法源码可以看出： 如果新容量大于 MAX_ARRAY_SIZE，进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

  ```java
  private static int hugeCapacity(int minCapacity) {
      if (minCapacity < 0) // overflow
          throw new OutOfMemoryError();
      // 对minCapacity和MAX_ARRAY_SIZE进行比较
      // 若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
      // 若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
      // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
      return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
  }
  ```

  ## 6、HashMap 的死锁问题  

  HashMap 多线程操作会造成链表的循环，在put过程中的resize方法在调用transfer方法的时候导致的死锁。

  JDK1.8后 HashMap 的确不会因为多线程put导致死循环，但是依然有其他的弊端，比如数据丢失等等。因此多线程情况下还是建议使用concurrenthashmap。

# HashMap

## 1、初始化的静态全局 final 变量

```java
// HashMap 的默认初始容量，必须是2的幂次方。1左移四位，0000 0001 -> 0001 0000 (1->16)
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
// HashMap 的最大容量（1左移30位）
static final int MAXIMUM_CAPACITY = 1 << 30;
// 装载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

## 2、三种构造函数

```java
// HashMap 的构造函数1
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不合法
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    // 初始容量大于最大容量，直接采用最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 装载因子不合法
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    // 设置装载因子
    this.loadFactor = loadFactor;
    // 对initialCapacity进行处理，获取大于它的并且最接近的2的幂次方的值
    this.threshold = tableSizeFor(initialCapacity);
}

// 声明初始容量的构造函数2，其实是调用的是构造函数1，装载因子采用默认装载因子。
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 不声明初始容量，默认构造。
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}


```

## 3、tableSizeFor 方法

该函数的作用就是对给定的容量 cap 进行处理，返回一个比 cap 大且最接近的2的幂次方整数。（这也从代码层面解释了 HashMap 的 capacity 为什么始终为2的幂次方）

```java
// |= 按位或后在赋值给左边
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;	// n = n | n>>>1
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

假定传入的 cap 为 10 ，模拟一下内部计算的流程。

```java
// 假设传入10，则tableSizeFor(10) = 16，以下是内部位运算的具体流程。
cap = 10
n = cap - 1 = 9		0000 1001  (9)
n |= n>>>1			0000 1001 -> n = 0000 1001 | 0000 0100 = 0000 1101
n |= n>>>2			0000 1101 -> n = 0000 1101 | 0000 0011 = 0000 1111
n |= n>>>4			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n |= n>>>8			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n |= n>>>16			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n = n + 1			0000 1111 -> n = n + 1 = 0001 0000  (16)
```

### (1) 为什么 cap 在传入后要减 1？

如果 cap 已经是2的幂， 没有执行这个减1操作，执行完后面的无符号右移操作之后，返回值将会是这个 cap 的2倍。减1的话则会返回 cap 本身。现在假设传入的 cap 初始值为8，是2的幂次方，来看一下 cap 减1和不减1的计算过程。

```java
// 减1
0000 1000  (8)
n = cap - 1 = 7 	0000 0111  (7)
n |= n>>>1			0000 0111 -> n = 0000 0111 | 0000 0011 = 0000 0111
n |= n>>>2			0000 0111 -> n = 0000 0111 | 0000 0001 = 0000 0111
n |= n>>>4			0000 0111 -> n = 0000 0111 | 0000 0000 = 0000 0111
n |= n>>>8			0000 0111 -> n = 0000 0111 | 0000 0000 = 0000 0111
n |= n>>>16			0000 0111 -> n = 0000 0111 | 0000 0000 = 0000 0111
n = n + 1			0000 0111 -> n = n + 1 = 0000 1000  (8)		// 最终结果为8
// 不减1
0000 1000  (8)
n |= n>>>1			0000 1000 -> n = 0000 1000 | 0000 0100 = 0000 1100
n |= n>>>2			0000 1100 -> n = 0000 1100 | 0000 0011 = 0000 1111
n |= n>>>4			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n |= n>>>8			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n |= n>>>16			0000 1111 -> n = 0000 1111 | 0000 0000 = 0000 1111
n = n + 1			0000 1111 -> n = n + 1 = 0001 0000  (16)	// 最终结果为16
```

### (2) **原反补码的计算**

正数：原码、反码和补码都一样。

负数：

- 原码 -> 反码：符号位不变，数值位按位取反。
- 原码 -> 补码：符号位不变，数值位按位取反，末位再加1。

### (3) Java 中 `>>` 与 `>>>` 的区别

- `>>` 是带符号右移（相当于除以2），计算机都是通过数字的补码进行运算，所以 `>>` 的操作对象为补码。

  **规则：正数右移高位补 0，负数右移高位补 1。**

  ```java
  4的二进制源码: 0000 0000 0000 0000 0000 0000 0000 0100  // 4
  4>>1:        0000 0000 0000 0000 0000 0000 0000 0010  // 2
      
  -4的二进制源码: 1000 0000 0000 0000 0000 0000 0000 0100  // -4
  -4的补码:      1111 1111 1111 1111 1111 1111 1111 1100
  -4>>1:        1111 1111 1111 1111 1111 1111 1111 1110
  再转换成原码:   1000 0000 0000 0000 0000 0000 0000 0010  // -2
  ```

- `>>>` 是不带符号右移，**无论负数还是正数，高位都用0补齐**。

  ```java
  4的二进制源码: 0000 0000 0000 0000 0000 0000 0000 0100  // 4
  4>>>1:       0000 0000 0000 0000 0000 0000 0000 0010  // 2
  
  // 负数补码右移后高位补0,变成正数,原码补码都一样。
  -4的二进制源码: 1000 0000 0000 0000 0000 0000 0000 0100  // -4
  -4的补码:      1111 1111 1111 1111 1111 1111 1111 1100
  -4>>>1:       0111 1111 1111 1111 1111 1111 1111 1110
  再转换成原码:   0111 1111 1111 1111 1111 1111 1111 1110  // 2147483646
  ```

## 4、put 方法

- 首先来看 put 函数部分的源码，调用的是 putval 函数。

> 该部分源码的注释：Associates the specified value with the specified key in this map. If the map previously contained a mapping for the key, the oldvalue is replaced.（将指定值与此映射中的指定键相关联。如果映射以前包含该键的映射，则会替换旧值。）
>
> ```java
> public class Test {
>     public static void main(String[] args) {
>         Map<String,Object> map = new HashMap<>();
>         map.put("key1",111);
>         map.put("key2",222);
>         map.put("key3",333);
>         System.out.println(map.toString());
>         map.put("key1","aaa");
>         System.out.println(map.toString());
>     }
> }
> /*
> 运行结果：
> 	{key1=111, key2=222, key3=333}
> 	{key1=aaa, key2=222, key3=333}
> */
> ```

将这个 key 的 hash 值，这个 key 本身，还有 value 作为参数传入到 putVal 方法中。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

- hash 方法

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);	// 为了更好的均匀散列表的下标
}
```

- putVal 方法

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; 
    Node<K,V> p; 
    int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e;
        K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```







# 常见面试题

## 1、HashMap如何解决哈希冲突  

1. 使用链地址法（使用散列表）来链接拥有相同hash值的数据；

2. 使用2次扰动函数（hash函数）来降低哈希冲突的概率，使得数据分布更平均；

   > 主要是因为如果使用hashCode取余，那么相当于参与运算的只有hashCode的低位，高位是没有起到任何作用的，所以我们的思路就是让hashCode取值出的高位也参与运算，进一步降低hash碰撞的概率，使得数据分布更平均，我们把这样的操作称为扰动。

3. 引入红黑树（JDK1.8）进一步降低遍历的时间复杂度，使得遍历更快。

## 2、HashMap 和 Hashtable 的区别

HashMap（JDK 1.2）几乎可以等价于 Hashtable（一开始就有），HashMap 和 Hashtable 都实现了 Map 接口，但决定用哪一个之前先要弄清楚它们之间的分别。

- 继承的父类不同

  HashMap 和 Hashtable 不仅作者不同，而且连父类也是不一样的。HashMap 是继承自 AbstractMap 类，而 Hashtable 是继承自Dictionary 类。

  Dictionary 类是一个已经被废弃的类（见其源码中的注释），父类都被废弃，自然也没人用它的子类 Hashtable 了。不过它们都实现了同时实现了 map、Cloneable（可复制）、Serializable（可序列化）这三个接口。

- 是否线程安全

  HashMap 不是线程安全的，Hashtable 是线程安全的（Hashtable 内部的方法基本都使用了 synchronized 关键字修饰），多个线程可以共享一个Hashtable；

  注意：现在 Hashtable 在我们的开发中很少使用，如果要保证线程安全，推荐使用ConcurrentHashMap（JDK 1.5），ConcurrentHashMap 的效率比 Hashtable 要高好多倍。因为 ConcurrentHashMap 使用了分段锁，并不对整个数据进行锁定。

- 效率问题

  由于 Hashtable 是线程安全的，所以在单线程环境下它比 HashMap 要慢。如果只需要单一线程，那么使用 HashMap 性能要好过Hashtable。

- 对于Null Key 和 Null Value 的支持

  - HashMap 中，null 可以作为 key，但是这样的 key 只能有一个；可以有一个或者多个键对应的 value 为 null，当 get() 方法返回 null值时，可能是 HashMap 中没有该键，也可能使该键所对应的值为 null。因此，在 HashMap 中不能由 get() 方法来判断 HashMap 中是否存在某个键， 而应该用 containsKey() 方法来判断。

  - Hashtable既不支持 Null key 也不支持 Null value。Hashtable 的 put() 方法的注释中有说明。如果 put 使用 null，那么就会抛出空指针异常。

- 初始容量和每次扩充容量的大小不同

  **不给定初始值**

  - HashMap 创建的时候如果不指定容量大小，初始容量大小为16，之后每次扩充，容量变为原来的2倍；
  - Hashtable 创建的时候如果不指定容量大小，初始容量大小为11，之后每次扩充，容量会变为 2n + 1；
  - HashMap 创建的时候给定初始容量大小，HashMap 会将其扩充为2的幂次方大小（HashMap 中的 `tableSizeFor()` 方法保证）。

  **给定初始值**

  - 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。Hashtable 会尽量使用素数、奇数。而 HashMap 则总是使用2的幂作为哈希表的大小。
  - 之所以会有这样的不同，是因为 Hashtable 和 HashMap 设计时的侧重点不同。Hashtable 的侧重点是哈希的结果更加均匀，使得哈希冲突减少。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀。
  - 而 HashMap 则更加关注 hash 的计算效率问题。在取模计算时，如果模数是2的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。HashMap 为了加快 hash 的速度，将哈希表的大小固定为了2的幂。当然这引入了哈希分布不均匀的问题，所以 HashMap 为解决这问题，又对 hash 算法做了一些改动。这从而导致了 Hashtable 和 HashMap 的计算 hash 值的方法不同。

- 底层数据结构

  -  JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间；
  -  Hashtable 没有这样的机制。

- 计算 hash 值的方法

  为了得到元素的位置，首先需要根据元素的 KEY计算出一个hash值，然后再用这个hash值来计算得到最终的位置。

  - Hashtable 直接使用对象的 hashCode。hashCode 是 JDK 根据对象的地址或者字符串或者数字算出来的int类型的数值。然后再使用除留余数发来获得最终的位置。Hashtable 在计算元素的位置时需要进行一次除法运算，而除法运算是比较耗时的。
  - HashMap 为了提高计算效率，将哈希表的大小固定为了2的幂，这样在取模预算时，不需要做除法，只需要做位运算。位运算比除法的效率要高很多。HashMap 的效率虽然提高了，但是 hash 冲突却也增加了。因为它得出的 hash 值的低位相同的概率比较高，为了解决这个问题，HashMap 重新根据 hashcode 计算 hash 值后，又对 hash 值做了一些运算来打散数据。使得取得的位置更加分散，从而减少了 hash 冲突。当然为了高效，HashMap只做了一些简单的位处理。从而不至于把使用2的幂次方带来的效率提升给抵消掉。

> 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。







# Guava

Guava 是谷歌提供的一个核心 Java 类库，其中包括**新的集合类型、不可变集合、图库，以及用于并发、I/O、Hash、缓存、字符串等的实用工具。**它在谷歌中的大多数 Java 项目中被广泛使用，也被许多其他公司广泛使用，熟练掌握这些工具类可以快速解决日常开发中的一些问题。

## 1、Guava 不可变类型集合

## 2、Guava 中新的集合类型

### 2.1、Multiset

> 问题引入：现在需要统计一个 list 中每个元素出现的次数
>
> 传统方式：定义一个 map，遍历 list，将 list 中元素的出现次数记录在 map 中，但是有了 Guava，特定的集合类可以帮助解决这个问题。

MultiSet：虽然名字中带有 set，但是可以储存重复元素，可以认为是 ArrayList 和 Map 的结合体。

MultiSet 是一个接口，JDK 中的各种 map 都有对应的实现。

|        Map        |      Multiset      | 是否支持 null 元素 |
| :---------------: | :----------------: | :----------------: |
|      HashMap      |    HashMultiset    |         是         |
|      TreeMap      |    TreeMultiset    |         是         |
|   LinkedHashMap   | LinkedHashMultiset |         是         |
| ConcurrentHashMap | ConcurrentMultiset |         否         |
|   ImmutableMap    | ImmutableMultiset  |         否         |





















































































































