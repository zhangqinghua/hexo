---
title: Problems

categories:
- 多线程高并发编程

tags:
- thread

date: 2020-01-22
---

#### 写一个固定容量的同步容器，拥有put和get方法，以及getCount方法。能够支持2个生产者线程以及10个消费者线程的阻塞调用

使用`wait`和`notify`或`notifyAll`来实线。

```java
public class MyContainer<T> {

    private int MAX = 10; // 最多10个元素

    private int count = 0;

    private final static LinkedList<T> list = new LinkedList<>();

    public synchronized void put(T t) {
        // 为什么用while不用if
        while (lists.size == MAX) {
            this.wait();
        }

        lists.add(t);
        ++count;

        // 通知消费者线程进行消费
        // 为什么不用notify
        this.notifyAll();
    }

    public synchronized T get() {
        T t = null;
        while (lists.size() == 0) {
            this.wait();
        }

        t = lists.removeFirst();
        count--;

        // 通知生产者进行生产
        this.notifyAll();
        return t;
    }

    public static void main(String[] args) {
        MyContainer<String> c = new MyContainer<>();
        // 启动消费者线程
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 5; j++) {
                    System.out.println(c.get())
                }
            }, "c" + i).start();
        }

        TimeUnit.SECONDES.sleep(2);

        // 启动生产者线程
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; i < 25; j++) {
                    c.put(Thread.currentThread().getName() + " " + j);
                }
            }, "p" + i).start();
        }
    }
}
```

为什么用`while`不用`if`呢？以消费者为例：
1. 假如有25个线程去获取元素，但是这是元素数量为0，大家陷入等待状态，释放锁（因为方法是`synchronized`修饰的，释放锁表示其它线程也可以继续访问了，相当于25个线程都调了这个方法，都在等待）
1. 生产者添加了一个元素，唤醒了所有的线程。