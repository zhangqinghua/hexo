---
title: 并发编程-CAS 机制

categories:
- 后端开发
- Java 并发编程

date: 2020-06-13
---
从是什么、有什么、为什么、如何用、底层原理等几个维度分析 CAS 机制。

#### CAS 简介
CAS 的全称是 Compare And Swap（比较与交换），是基于硬件原语实现的，能够在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。

#### CAS 拥有功能
CAS 使用了 3 个基本操作数：内存地址 V，旧的预期值 A，要修改的新值 B。更新一个变量的时候，只有当变量的预期值 A 和内存地址 V 当中的实际值相同时，才会将内存地址 V 对应的值修改为 B。

![](https://img2018.cnblogs.com/blog/1775037/202001/1775037-20200106164315461-658325570.jpg)

#### CAS 出现背景
从 Java5 开始引入了对 CAS 机制的底层的支持，在这之前需要开发人员编写相关的代码才可以实现 CAS。

#### CAS 基本使用
JDK1.5 之后的 `java.util.concurrent.atomic` 包里，多了一批原子处理类。`AtomicBoolean`、`AtomicInteger`、`AtomicLong`、`AtomicReference`。主要用于在高并发环境下的高效程序处理，来帮助我们简化同步处理。

#### CAS 优点缺点
CAS 的优点：
1. 可以保证变量操作的原子性；
1. 并发量不是很高的情况下，使用 CAS 机制比使用锁机制效率更高；
1. 在线程对共享资源占用时间较短的情况下，使用 CAS 效率也会较高。

CAS 虽然很高效的解决原子操作，但是 CAS 仍然存在三大问题：

1. ABA 问题
   因为 CAS 需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新。但是如果一个值原来是 A，变成了 B，又变成了 A，那么使用 CAS 进行检查时会发现它的值没有发生变化，但是实际上却变化了。

   ABA 问题的一个解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加 1，那么 A->B->A 就会变成 1A->2B->3A。

   另外从 Java1.5 开始 JDK 提供了 `AtomicStampedReference` 来解决 ABA 问题。这个类的 `compareAndSet` 方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

1. 循环时间长开销大
   在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给 CPU 带来很大的压力。

1. 只能保证一个共享变量的原子操作
   CAS 机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证 3 个变量共同进行原子性的更新时，就不得不使用 `synchronized` 了。

   另外从 Java1.5 开始 JDK 提供了 `AtomicReference` 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作。

#### CAS 底层原理
CAS 并发原语体现在 Java 语言中就是 `sun.misc.Unsafe` 类中的各个方法。调用 `Unsafe` 类中的 CAS 方法，JVM 会帮我们实现出 CAS 汇编指令。这是一种完全依赖于硬件的功能，通过它实现了原子操作。

再次强调，由于 CAS 是一种系统原语，属于操作系统用语范凑，是由若干条指令组成的，用于完成某个功能的一个过程。并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说 CAS 是一条 CPU 的原子指令，不会造成所谓的数据不一致问题。

`AtomicInteger` 的自增方法就使用到了 `Unsafe` CAS 操作。

```java
// 1. AtomicInteger 自增操作
public final int getAndIncrement() {
   return unsafe.getAndAddInt(this, valueOffset, 1);
}

/**
 * 2. Unsafe 原子新增，循环比较交换。
 *
 * @param atomicInteger AtomicInteger 对象本身
 * @param valueoffset   内存地址偏移量
 * @param incr          需要新增的数量
 */
public final int getAndAddInt(Object atomicInteger, long valueoffset, int incr) {
   int var5;                                                                         // 期望值
   do {
      var5 = this.getIntVolatile(atomicInteger, valueoffset);                        // 通过主内存找出真实的值
   } while (!this.compareAndSwapInt(atomicInteger, valueoffset, var5, var5 + incr)); // 期望值一致的，则交换。否则循环刷新
   return var5;
}

// 3. 使用到了C++实现
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

#### concurrent 包

#### 附：AtomicInteger
```java
AtomicInteger i = new AtomicInteger(0);
// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
System.out.println(i.getAndIncrement());
// 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
System.out.println(i.incrementAndGet());
// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
System.out.println(i.decrementAndGet());
// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
System.out.println(i.getAndDecrement());
// 获取并加值（i = 0, 结果 i = 5, 返回 0）
System.out.println(i.getAndAdd(5));
// 加值并获取（i = 5, 结果 i = 0, 返回 0）
System.out.println(i.addAndGet(-5));
// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.getAndUpdate(p -> p - 2));
// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.updateAndGet(p -> p + 2));
// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
// getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
```

#### 附：Unsafe 类
`Unsafe` 类是 CAS 的核心类，由于 Java 方法无法直接访问底层系统，需要通过本地（`native`）方法来访问，`Unsafe` 类相当于一个后门，基于该类可以直接操作特定内存的数据。

`Unsafe` 类存在于 sum.misc 包中，其内部方法操作可以像 C 的指针一样直接操作内存。Java 中 CAS 操作的执行都依赖于 `Unsafe` 类方法。

> 注意 Unsafe 类中的所有方法都是 native 修饰的，也就是说 Unsafe 类中的方法都直接调用操作系统底层资源执行相应任务。

`Unsafe` 类提供了硬件级别的原子操作，主要提供了以下功能：

1. 通过 `Unsafe` 类可以分配内存，可以释放内存；

1. 可以定位对象某字段的内存位置，也可以修改对象的字段值，即使它是私有的；

1. 挂起与恢复；

1. CAS 操作；

#### 分配和释放内存
类中提供的 3 个本地方法 `allocateMemory`、`reallocateMemory`、`freeMemory` 分别用于分配内存，扩充内存和释放内存，与 C 语言中的 3 个方法对应。

```java
public native long allocateMemory(long l);
public native long reallocateMemory(long l, long l1);
public native void freeMemory(long l);
```

#### 定位对象某字段的内存位置、修改对象的字段值
1. 字段的定位
   Java 中对象的字段的定位可能通过 `staticFieldOffset` 方法实现，该方法返回给定 field 的内存地址偏移量，这个值对于给定的 filed 是唯一的且是固定不变的。

   `getLong` 方法获取对象中 offset 偏移地址对应的 `long` 型 field 的值。

   `getIntVolatile` 方法获取对象中 offset 偏移地址对应的整型 field 的值，支持 volatile load 语义。
   
1. 数组元素定位
   `Unsafe` 类中有很多以 BASE_OFFSET 结尾的常量，比如 ARRAY_INT_BASE_OFFSET，ARRAY_BYTE_BASE_OFFSET 等，这些常量值是通过 `arrayBaseOffset` 方法得到的。
   
   `arrayBaseOffset` 方法是一个本地方法，可以获取数组第一个元素的偏移地址。

   `Unsafe` 类中还有很多以 `INDEX_SCALE` 结尾的常量，比如 ARRAY_INT_INDEX_SCALE ， ARRAY_BYTE_INDEX_SCALE等，这些常量值是通过arrayIndexScale方法得到的。
   
   `arrayIndexScale` 方法也是一个本地方法，可以获取数组的转换因子，也就是数组中元素的增量地址。
   
   `arrayBaseOffset` 与 `arrayIndexScale` 配合使用，可以定位数组中每个元素在内存中的位置。

```java
public final class Unsafe {
    public static final int ARRAY_INT_BASE_OFFSET;
    public static final int ARRAY_INT_INDEX_SCALE;

    public native long getLong(Object obj, long l);
    public native int  getIntVolatile(Object obj, long l);
    public native long staticFieldOffset(Field field);

    public native int arrayBaseOffset(Class class1);
    public native int arrayIndexScale(Class class1);

    static 
    {
        ARRAY_INT_BASE_OFFSET = theUnsafe.arrayBaseOffset([I);
        ARRAY_INT_INDEX_SCALE = theUnsafe.arrayIndexScale([I);
    }
}
```
#### 挂起与恢复
将一个线程进行挂起是通过 `park` 方法实现的，调用 `park` 后，线程将一直阻塞直到超时或者中断等条件出现。`unpark` 可以终止一个挂起的线程，使其恢复正常。整个并发框架中对线程的挂起操作被封装在 `LockSupport` 类中，`LockSupport` 类中有各种版本 `pack` 方法，但最终都调用了 `Unsafe.park()` 方法。

```java
public class LockSupport {
    public static void unpark(Thread thread) {
        if (thread != null)
            unsafe.unpark(thread);
    }

    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(false, 0L);
        setBlocker(t, null);
    }

    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            unsafe.park(false, nanos);
            setBlocker(t, null);
        }
    }

    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(true, deadline);
        setBlocker(t, null);
    }

    public static void park() {
        unsafe.park(false, 0L);
    }

    public static void parkNanos(long nanos) {
        if (nanos > 0)
            unsafe.park(false, nanos);
    }

    public static void parkUntil(long deadline) {
        unsafe.park(true, deadline);
    }
}
```

#### CAS 操作
`Unsafe` 类的 CAS 操作是通过 `compareAndSwapXXX` 方法实现的。CAS 操作有 3 个操作数，内存值 M，预期值 E，新值 U，如果 M==E，则将内存值修改为 B，否则啥都不做。

```java
/**
 * 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
 * 
 * @param   obj 需要更新的对象
 * @param   offset obj中整型field的偏移量
 * @param   expect 希望field中存在的值
 * @param   update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
 * @return  如果field的值被更改返回true
 */
public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);
```

#### 附：我们知道 ArrayList 是线程不安全的，请编码写一个不安全的案例并给出解决方案。
1. 故障现象 
   java.util.ConcurrentModificationException。
1. 导致原因 
   并发修改导致的，一个线程正在写入，另一个线程过来抢夺，导致数据不一致。
1. 解决方案 
   new Vector()。
   Collections.synchronizedList(new ArrayList())。
   new CopyOnWriteArrayList()。
1. 优化建议
1. 衍生：SET、MAP。

直接使用 ArrayList，会导致异常：

```java
List<Integer> list = new ArrayList<>();

for (int i = 0; i < 4; i++) {
    final int temp = i;
    new Thread(() -> {
        list.add(temp);
        System.out.println(list);
    }, i + "").start();
}

[0, 1]
[0, 1, 2, 3]
[0, 1, 2, 3]
Exception in thread "0" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:911)
	at java.util.ArrayList$Itr.next(ArrayList.java:861)
	at java.util.AbstractCollection.toString(AbstractCollection.java:461)
	at java.lang.String.valueOf(String.java:3450)
	at java.io.PrintStream.println(PrintStream.java:821)
	at test4.Demo.lambda$main$0(Demo.java:14)
	at test4.Demo$$Lambda$1/000000000000000000.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:823)
```

使用 Collections.synchronizedList：

```java
List<Integer> list = Collections.synchronizedList(new ArrayList<>());

for (int i = 0; i < 4; i++) {
   final int temp = i;
   new Thread(() -> {
         list.add(temp);
         System.out.println(list);
   }, i + "").start();
}

[0, 1]
[0, 1, 2]
[0, 1, 2, 3]
[0, 1, 2, 3]
```

使用 Vector，正常，但是效率慢：

```java
List<Integer> list = new Vector<>();

for (int i = 0; i < 4; i++) {
   final int temp = i;
   new Thread(() -> {
         list.add(temp);
         System.out.println(list);
   }, i + "").start();
}

[0, 1]
[0, 1, 2, 3]
[0, 1, 2, 3]
[0, 1, 2]
```

CopyOnWrite 即写时复制的容器。往一个容器添加元素的时候，不直接往当前容器 ojbect[] 添加，而是先将当前容器 object[] 进行 copy，复制出一个新的容器 object[] newElemenets，然后往新的容器里添加元素。添加完元素之后，再将原容器的引用指向新的容器。

这样做的好处是可以对容器进行并发的读而不需要加锁，因为当前容器不会添加任何元素。所以 CopyOnWrite 容器也是一种读写分离的思想，读和写不同的容器。

```java
List<Integer> list = new CopyOnWriteArrayList<>();
for (int i = 0; i < 4; i++) {
   final int temp = i;
   new Thread(() -> {
         list.add(temp);
         System.out.println(list);
   }, i + "").start();
}

[0]
[0, 1, 2]
[0, 1, 2, 3]
[0, 1]
```

CopyOnWriteArraylist 核心源码：
```java
public boolean add(E e) {
   final ReentrantLock lock = this.lock;
   lock.lock();
   try {
      Object[] elements = getArray();
      int len = elements.length;
      Object[] newElements = Arrays.copyOf(elements, len + 1);
      newElements[len] = e;
      setArray(newElements);
      return true;
   } finally {
      lock.unlock();
   }
}
```

#### CPU 并发原语
不会。

#### AtomicInteger
`AtomicInteger` 是一个提供原子操作的 `Integer` 的类。在 Java 语言中，`++i` 和 `i++` 操作并不是线程安全的，在使用的时候，不可避免的会用到 `synchronized` 关键字。而 `AtomicInteger` 则通过一种线程安全的加减操作接口。

`AtomicInteger` 提供了下面几个接口:

```java
public final int get();                      //获取当前的值
public final int getAndSet(int newValue);    //获取当前的值，并设置新的值
public final int getAndAdd(int delta);       //获取当前的值，并加上预期的值
public final int getAndIncrement();          //获取当前的值，并自增
public final int getAndDecrement();          //获取当前的值，并自减
```

下面通过两个简单的例子来看一下 `AtomicInteger` 的优势在哪。

普通线程同步：

```java
class Test2 {
   private volatile int count = 0;

   // 若要线程安全执行执行count++，需要加锁
   public synchronized void increment() {
      count++; 
   }

   public int getCount() {
      return count;
   }
}
```

使用 `AtomicInteger`：

```java
class Test2 {
   private AtomicInteger count = new AtomicInteger();

   public void increment() {
      count.incrementAndGet();
   }
   // 使用AtomicInteger之后，不需要加锁，也可以实现线程安全。
   public int getCount() {
      return count.get();
   }
}
```

使用 `AtomicInteger` 是非常的安全的.而且因为 `AtomicInteger` 由硬件提供原子操作指令实现的。在非激烈竞争的情况下，开销更小，速度更快。

#### AtomicReference
跟 `AtomicInteger` 类似，里面封装的是一个对象。

#### AtomicStampedReference
由于ABA问题带来的隐患，各种乐观锁的实现中通常都会用版本戳version来对记录或对象标记，避免并发操作带来的问题。在Java中，`AtomicStampedReference` 也实现了这个作用，它通过包装 `[E,Integer]` 的元组来对对象标记版本戳 stamp，从而避免 ABA 问题。

```java
class Test2 {
   private static AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<Integer>(100, 0);

   public void increment() {
      atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
   }

   // 使用AtomicInteger之后，不需要加锁，也可以实现线程安全。
   public int getCount() {
      return count.get();
   }
}
```