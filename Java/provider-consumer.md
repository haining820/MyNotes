---
title: 生产者消费者问题
date: 2022-09-03
tags: [生产者消费者]
toc: true
---

# 1、生产者和消费者问题

- 假设仓库中只能存放一件产品，生产者将生产出来的产品放入仓库，消费者将仓库中产品取走消费；
- 如果仓库中没有产品，生产者将产品放入仓库；如果仓库中有产品，停止生产并等待，直到仓库中的产品被消费者取走为止；
- 如果仓库中放有产品，则消费者将产品取走消费，否则停止消费并等待，直到仓库中再次放入产品为止。

<!--more-->

**这是一个线程同步问题，生产者和消费者共享同一个资源，并且生产者和消费者之间相互依赖，互为条件。**

- 对于生产者，没有生产产品之前，要通知消费者等待；而生产了产品之后，又需要马上通知消费者消费；
- 对于消费者，在消费之后，要通知生产者已经结束消费，需要生产新的产品以供消费；

- 在生产者消费者问题中 , 仅有 synchronized 是不够的，synchronized 可阻止并发更新同一个共享资源 , 实现了同步；但是 synchronized 不能用来实现不同线程之间的消息传递（通信）。这里要用到 `wait()` 和`notify()` 的相关方法。

Java 提供了几个方法解决线程之间的通信问题，**注意：这些方法均是 Object 类的方法，都只能在同步方法或者同步代码块中使用，否则会抛出异常 IllegalMonitorStateException。**

| 方法名               | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| `wait()`             | 表示线程一直等待，直到其他线程通知，与 `sleep()` 不同，会释放锁。 |
| `wait(long timeout)` | 指定等待的毫秒数                                             |
| `notify()`           | 唤醒一个处于等待状态的线程                                   |
| `notifyAll()`        | 唤醒**同一个对象**上所有调用`wait()`方法的线程，优先级别高的线程优先调度。 |

**生产者消费者 demo**

{% spoiler 展开查看折叠代码 %}


```java
// 测试：生产者消费者模式
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Producer(container).start();	// 开启生产者
        new Consumer(container).start();	// 开启消费者
    }
}

// 生产者
class Producer extends Thread {
    SynContainer container;
    public Producer(SynContainer container) {
        this.container = container;
    }
    @Override   // 生产
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            // 在这里输出生产情况和库存情况会有数据不一致的问题，所以在这里不进行输出
            // 因为push和pop是同时进行的，在输出之前已经进行了push/pop
            // 假设当前push完成后输出，在push完成到输出这段时间内可能会发生其他的push或pop，输出结果就会受到影响
        }
    }
}

// 消费者
class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer container) {
        this.container = container;
    }

    @Override   // 消费
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.pop();
        }
    }
}

// 产品
class Chicken {
    int id;
    public Chicken(int id) {
        this.id = id;
    }
}

// 缓冲区
class SynContainer {
    Chicken[] chickens = new Chicken[10];	// 容器大小为10
    int count = 0;	// 容器计数器
    // 生产者放入产品
    public synchronized void push(Chicken chicken) {
        while (count == 10) {   // 容器满了，等待消费者消费
            try {
                this.wait();    // 通知消费者消费，生产者等待
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 如果容器没满，生产者继续生产，将产品丢入容器
        chickens[count] = chicken;
        count++;
        System.out.println("生产: " + chicken.id + "-->" + count);
        // 通知消费者消费
        this.notifyAll();
    }

    public synchronized Chicken pop() {
        while (count == 0) {    // 容器为空，等待生产者生产
            try {
                this.wait();    // 通知生产者生产，消费者等待
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        count--;    // 消费者消费
        Chicken chicken = chickens[count];
        System.out.println("消费: " + chicken.id + "-->" + count);
        this.notifyAll();   // 消费完了，通知生产者生产
        return chicken;
    }

}
```

{% endspoiler %}

# 2、传统的生产者消费者问题（虚假唤醒）

{% spoiler 展开查看折叠代码 %}

```java
/**
 * 线程之间的通信：生产者消费者问题（传统方式处理）
 * 线程A、B交替对同一个变量num进行操作
 */
// 判断等待->业务->通知
class Resource {
    private int number = 0;

    public synchronized void increment() throws InterruptedException {
        while (number != 0) {	// if
            this.wait();	// 等待
        }
        number++;   // 生产
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，+1完毕
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {	// if
            this.wait()； // 等待
        }
        number--;   // 消费
        System.out.println(Thread.currentThread().getName() + "=>" + number);
        // 通知其他线程，-1完毕
        this.notifyAll();
    }
}

public class Test1 {
    public static void main(String[] args) {
        Resource resource = new Resource();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        /*new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread	(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();*/
    }
}
```

{% endspoiler %}

**存在的问题：**以上程序中只使用了A、B两个线程的话没有问题，但是如果使用四个甚至八个线程的话仍然会出现线程不安全的问题。这种情况叫做**虚假唤醒**。

**虚假唤醒：**当一个条件满足时，很多线程都被唤醒了，但是只有其中部分是有用的唤醒，其它的唤醒都是无用功。 比如买货，如果商店本来没有货物，突然新增一件商品，此时所有的线程都被唤醒了 ，但是只能一个人买，所以其他人都是假唤醒，获取不到对象的锁。

**解决方法：**等待应该总是出现在循环 `while` 中，而不是 `if` 中，因为 while 会一直进行判断，直到条件不满足才会继续向下执行，不使用一次性判断，就可以避免产生虚假唤醒的问题。

# 3、JUC 版的生产者消费者问题

| 传统三件套 | synchronized | `wait()`                   | `notifyAll()`  |
| ---------- | ------------ | -------------------------- | -------------- |
| **JUC**    | **Lock**     | **`await()`**（Condition） | **`signal()`** |

Lock 可以替换 synchronized 方法和语句的使用，Condition 取代了对象监视器方法的使用，原理相同，仍可以实现生产者消费者问题。

{% spoiler 展开查看折叠代码 %}

```java
/**
 * 线程之间的通信：生产者消费者问题（传统方式处理）
 * 线程A、B交替对同一个变量进行操作
 */
public class Test2 {
    public static void main(String[] args) {
        Resource2 resource = new Resource2();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    resource.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

// 判断等待->业务->通知
class Resource2 {
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void increment() throws InterruptedException {
        lock.lock();
        try {
            while (number != 0) {
                condition.await();	// 等待
            }
            number++;   // 生产
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            condition.signalAll();	// 通知其他线程，+1完毕
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                condition.await();	// 等待
            }
            number--;   // 消费
            System.out.println(Thread.currentThread().getName() + "=>" + number);
            condition.signalAll();	// 通知其他线程，-1完毕
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

{% endspoiler %}

问题：虽然实现了功能，但是线程的执行是随机的，和之前传统方式的解决效果是一样的，没什么区别。

<font size=4 style="font-weight:bold;background:yellow;">如何让 ABCD 四个线程按顺序执行？</font>

**使用 Condition 精准的通知和唤醒线程**，这是通过 JUC 的方式解决生产者消费者问题的一个优势。

{% spoiler 展开查看折叠代码 %}

```java
/**
 * A执行完调用B，B执行完调用C，C执行完调用A
 */
public class Test3 {
    public static void main(String[] args) {
        Resource3 resource3 = new Resource3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource3.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource3.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource3.printC();
            }
        }, "C").start();
    }
}

class Resource3 {
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number = 1;     // 1A 2B 3C

    public void printA() {
        lock.lock();
        try {
            while (number != 1) {
                // 等待
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "=>A");
            // 唤醒B
            number = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            while (number != 2) {
                // 等待
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName()+"=>B");
            // 唤醒C
            number = 3;
            condition3.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            while (number != 3) {
                // 等待
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName()+"=>B");
            // 唤醒A
            number = 1;
            condition1.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

{% endspoiler %}



# 4、使用阻塞队列实现生产者消费者问题

实现一个通过命令行读取货物的运输时间的生产者线程和3个快递员的消费者线程，当等待投递的货物超过3个时，生产者线程拒绝下一个货物的输入。如，用户输入1000，表示这个货物需要花费1000毫秒投递，如果有快递员线程空闲，则进行投递，并消耗相应时间。

<font size=4 style="font-weight:bold;background:yellow;">货物实体类</font>

```java
public class Goods {

    // 货物投递需要的毫秒数
    private int time;

    public Goods(int time) {
        this.time = time;
    }

    public int getTime() {
        return time;
    }

    public void setTime(int time) {
        this.time = time;
    }
}
```

<font size=4 style="font-weight:bold;background:yellow;">生产者和消费者接口</font>

```java
public interface IProducer {
    // 生产者接口，请求是输入时间参数（获取一个请求对象），返回是一个 Good
    Goods produce(int time);
}
```

```java
public interface IConsumer {
	// 消费者接口，消费一个货物
    void consume(Goods goods);
}
```

<font size=4 style="font-weight:bold;background:yellow;">生产者和消费者</font>

```java
@Component
public class Producer implements IProducer {
    @Override
    public Goods produce(int time) {
        return new Goods(time);
    }
}
```

```java
@Component
public class Consumer implements IConsumer {
    @Override
    public void consume(Goods goods) {
    }
}
```

<font size=4 style="font-weight:bold;background:yellow;">Solution</font>

```java
@Component
public class Solution {
    
    private static Logger logger = Logger.getLogger(Solution.class);
    
    // 声明阻塞队列长度为3
    private static LinkedBlockingQueue<Goods> blockingQueue = new LinkedBlockingQueue<>(3);

    public void produceLogger(Goods goods) {
        try {
            if (blockingQueue.size() == 3) {
                logger.info("队列满了!");
                logger.info(Thread.currentThread().getName() + "生产失败,当前仓库剩余->" + blockingQueue.size());
            } else {
                blockingQueue.put(goods);
                logger.info(Thread.currentThread().getName() + "生产成功,当前仓库剩余->" + blockingQueue.size());
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public void consumeLogger(Goods goods) {
        int time = goods.getTime();
        logger.info("线程" + Thread.currentThread().getName() + "花费" + time + "ms投递货物,剩余->" + blockingQueue.size());
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        logger.info("线程" + Thread.currentThread().getName() + "花费" + time + "ms投递完成,剩余->" + blockingQueue.size());
    }

    @Autowired
    private IProducer producer;

    @Autowired
    private IConsumer consumer;
    
    public void runPC() {
        new Thread(() -> {
            while (true) {
                Scanner sc = new Scanner(System.in);
                int time = sc.nextInt();
                Goods goods = producer.produce(time);
                produceLogger(goods);
            }
        }, "Producer").start();
        for (int i = 1; i <= 3; i++) {
            new Thread(() -> {
                while (true) {
                    Goods goods;
                    try {
                        goods = blockingQueue.take();
                        consumer.consume(goods);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                    consumeLogger(goods);
                }
            }, String.valueOf(i)).start();
        }
    }

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Solution solution = (Solution) context.getBean("solution");
        solution.runPC();
    }
}
```

