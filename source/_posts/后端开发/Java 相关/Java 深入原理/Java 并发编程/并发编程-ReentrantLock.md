---
title: 并发编程-ReentrantLock

categories:
- Java 并发编程

date: 2020-01-01 00:00:04
---
前面我们已经学习过了 `synchronized`，这个关键字可以确保对象在并发访问中的原子性、可见性和有序性，这个关键字的底层交由了JVM 通过 C++ 来实现，既然是 JVM 实现，就依赖于 JVM，程序员就无法在 Java 层面进行扩展和优化，肯定就灵活性不高，比如程序员在使用时就无法中断一个正在等待获取锁的线程，或者无法在请求一个锁时无限的等待下去。

基于这样一个背景，Doug Lea 构建了一个在内存语义上和 `synchronized` 一样效果的 Java 类，同时还扩展了其他一些高级特性，比如定时的锁等待、可中断的锁等待和公平性等，这个类就是 `ReentrantLock`。

> 在 JDK1.5 里面，ReentrantLock 的性能是明显优于 synchronized 的，但是在 JDK1.6 里面，synchronized 做了优化，他们之间的性能差别已经不明显了。

## 基本使用
#### 普通的线程锁
这种用法和 `synchronized` 效果是一样的，但是必须显示的声明 `lock` 和 `unlock`。

```java
ReentrantLock lock = new ReentrantLock();
try {
   lock.lock();
   //……
}finally {
   lock.unlock();
}
```

#### 带限制的锁
体可查看 github 链接里面的 ReentrantLockTest。

```java
public boolean tryLock()                              // 尝试获取锁,立即返回获取结果 轮询锁
public boolean tryLock(long timeout, TimeUnit unit)   //尝试获取锁,最多等待 timeout 时长 超时锁
public void lockInterruptibly()                       //可中断锁,调用线程 interrupt 方法,则锁方法抛出 InterruptedException  中断锁
```

#### 等待/通知模型
内置队列存在一些缺陷，每个内置锁只能关联一个条件队列(_WaitSet)，这导致多个线程可能会在同一个条件队列上等待不同的条件谓词，如果每次使用 `notify` 唤醒条件队列，可能会唤醒错误的线程导致唤醒失败，但是如果使用 `notifyAll` 的话，能唤醒到正确的线程，因为所有的线程都会被唤醒，这也带来一个问题，就是不应该被唤醒的在被唤醒后发现不是自己等待的条件谓词转而又被挂起。

这样的操作会带来系统的资源浪费，降低系统性能。这个时候推荐使用显式的 `Lock` 和 `Condition` 来替代内置锁和条件队列，从而控制多个条件谓词的情况，达到精确的控制线程的唤醒和挂起。具体后面再来分析下JVM的内置锁、条件队列模型和显式的 `Lock`、`Condition` 模型，实际上在 AQS 里面也提到了 `Lock`、`Condition` 模型。

## 和 synchronized 比较
1. 原始构成
   sychronized 是关键字属于 JVM 层面。
   monitorenter + monitorexit（底层通过monitor对象来完成，其实）

   Lock 是具体类，是 api 层面。

1. 使用方法
   synchronized 不需要用户去手动释放锁，当 synchronized 代码执行完后系统会自动让线程释放对锁的占用。
   ReentrantLock 则需要用户去手动释放锁，若没有主动释放锁，就有可能导致死锁现象。

1. 加锁是否公平
   synchronized  非公平锁。
   ReentrantLock 两者都可以，默认非公平锁。可在构造方法传入 boolean 值，true 为公平锁，false 为非公平锁。

1. 等待是否可中断
   synchronzied 不可中断，除非正常运行完成或异常抛出。
   ReentranLock 可中断（1）设置超时时间 （2）lockInterruptiry() 放代码块中。

1. 锁绑定多个条件
   synchronized  没有
   ReentrantLock 用来实现分组唤醒需要唤醒的线程，可以精确唤醒，而不是像 synchronized 要么随机唤醒一个线程要么唤醒全部线程。

## 源码原理解析
#### 可重入性原理
在 `synchronized` 一文中，我们认为 `synchronized` 是一种重量级锁，它的实现对应的是 C++ 的 `ObjectMonitor`，代码如下：

```c++
ObjectMonitor() {
   _header       = NULL;
   _count        = 0;    //记录线程获取锁的次数
   _waiters      = 0;
   _recursions   = 0;    //锁的重入次数
   _object       = NULL;
   _owner        = NULL; //指向持有ObjectMonitor对象的线程
   _WaitSet      = NULL; //等待条件队列 类似AQS的ConditionObject
   _WaitSetLock  = 0 ;
   _Responsible  = NULL ;
   _succ         = NULL ;
   _cxq          = NULL ;
   FreeNext      = NULL ;
   _EntryList    = NULL ; //同步队列 类似AQS的CLH队列
   _SpinFreq     = 0 ;
   _SpinClock    = 0 ;
   OwnerIsThread = 0 ;
   _previous_owner_tid = 0;
}
```

从代码中可以看到 `synchronized` 实现的锁的重入依赖于 JVM，JVM 为每个对象的锁关联一个计数器 `_count` 和一个所有者线程 `_owner`，当计数器为 0 的时候就认为锁没有被任何线程持有，当线程请求一个未被持有的锁时，JVM 就记下锁的持有者，并将计数器的值设置为 1，如果是同一个线程再次获取这个锁，计数器的值递增，而当线程退出时，计数器的值递减，直到计数器为 0 时，锁被释放。

`ReentrantLock` 实现了在内存语义上的 `synchronized`，固然也是支持可重入的，那么 `ReentrantLock` 是如何支持的呢，让我们以非公平锁的实现看下 `ReentrantLock` 的可重入，代码如下：

```java
final boolean nonfairTryAcquire(int acquires) {
   final Thread current = Thread.currentThread();//当前线程
   int c = getState();
   if (c == 0) {//表示锁未被抢占
         if (compareAndSetState(0, acquires)) {//获取到同步状态
            setExclusiveOwnerThread(current); //当前线程占有锁
            return true;
         }
   }
   else if (current == getExclusiveOwnerThread()) {//线程已经占有锁了 重入
         int nextc = c + acquires;//同步状态记录重入的次数
         if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
         setState(nextc);
         return true;
   }
   return false;
}

protected final boolean tryRelease(int releases) {
   int c = getState() - releases; //既然可重入 就需要释放重入获取的锁
   if (Thread.currentThread() != getExclusiveOwnerThread())
         throw new IllegalMonitorStateException();
   boolean free = false;
   if (c == 0) {
         free = true;//只有线程全部释放才返回true
         setExclusiveOwnerThread(null); //同步队列的线程都可以去获取同步状态了
   }
   setState(c); 
   return free;
}
```

看到这也就明白了上文说的 `ReentrantLock` 类使用 AQS 同步状态来保存锁重复持有的次数。当锁被一个线程获取时，`ReentrantLock` 也会记录下当前获得锁的线程标识，以便检查是否是重复获取，以及当错误的线程试图进行解锁操作时检测是否存在非法状态异常。

#### 获取和释放锁
如下是获取和释放锁的方法：

```java
public void lock() {
   sync.lock();//获取锁
}
public void unlock() {
   sync.release(1); //释放锁
}
```

获取锁的时候依赖的是内部类 `Sync` 的 `lock()` 方法，该方法又有 2 个实现类方法，分别是非公平锁 `NonfairSync` 和公平锁 `FairSync`，具体咱们下一小节分析。再来看下释放锁，释放锁的时候实际调用的是 AQS 的 `release` 方法，代码如下：

```java
public final boolean release(int arg) {
   if (tryRelease(arg)) {//调用子类的tryRelease 实际就是Sync的tryRelease
      Node h = head;//取同步队列的头节点
      if (h != null && h.waitStatus != 0)//同步队列头节点不为空且不是初始状态
            unparkSuccessor(h);//释放头节点 唤醒后续节点
      return true;
   }
   return false;
}
```

`Sync` 的 `tryRelease` 就是上一小节的重入释放方法，如果是同一个线程，那么锁的重入次数就依次递减，直到重入次数为 0，此方法才会返回 true，此时断开头节点唤醒后续节点去获取 AQS 的同步状态。

#### 公平锁和非公平锁
公平锁还是非公平锁取决于 `ReentrantLock` 的构造方法，默认无参构造方法是 `NonfairSync`，含参构造方法，入参 `true` 为 `FairSync`，入参 `false` 为 `NonfairSync`。

```java
public ReentrantLock() {
   sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
   sync = fair ? new FairSync() : new NonfairSync();
}
```

再分别来看看非公平锁和公平锁的实现。

```java
static final class NonfairSync extends Sync {
   private static final long serialVersionUID = 7316153563782823691L;

   /**
   * Performs lock.  Try immediate barge, backing up to normal
   * acquire on failure.
   */
   final void lock() {
      if (compareAndSetState(0, 1))//通过CAS来获取同步状态 也就是锁
            setExclusiveOwnerThread(Thread.currentThread());//获取成功线程占有锁
      else
            acquire(1);//获取失败 进入AQS同步队列排队等待 执行AQS的acquire方法 
   }

   protected final boolean tryAcquire(int acquires) {
      return nonfairTryAcquire(acquires);
   }
}
```

在 AQS 的 `acquire` 方法中先调用子类 `tryAcquire`，也就是 `nonfairTryAcquire`，见 2.1 小节。可以看出非公平锁中，抢到AQS的同步状态的未必是同步队列的首节点，只要线程通过 CAS 抢到了同步状态或者在 `acquire` 中抢到同步状态，就优先占有锁，而相对同步队列这个严格的FIFO队列来说，所以会被认为是非公平锁。

```java
static final class FairSync extends Sync {
   private static final long serialVersionUID = -3000897897090466540L;

   final void lock() {
      acquire(1);//严格按照AQS的同步队列要求去获取同步状态
   }

   /**
   * Fair version of tryAcquire.  Don't grant access unless
   * recursive call or no waiters or is first.
   */
   protected final boolean tryAcquire(int acquires) {
      final Thread current = Thread.currentThread();//获取当前线程
      int c = getState();
      if (c == 0) {//锁未被抢占
            if (!hasQueuedPredecessors() &&//没有前驱节点
               compareAndSetState(0, acquires)) {//CAS获取同步状态
               setExclusiveOwnerThread(current);
               return true;
            }
      }
      else if (current == getExclusiveOwnerThread()) {//锁已被抢占且线程重入
            int nextc = c + acquires;//同步状态为重入次数
            if (nextc < 0)
               throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
      }
      return false;
   }
}
```

公平锁的实现直接调用 AQS 的 `acquire` 方法，`acquire` 中调用 `tryAcquire`。和非公平锁相比，这里不会执行一次 CAS，接下来在 `tryAcquire` 去抢占锁的时候，也会先调用 `hasQueuedPredecessors` 看看前面是否有节点已经在等待获取锁了，如果存在则同步队列的前驱节点优先。

```java
public final boolean hasQueuedPredecessors() {
   // The correctness of this depends on head being initialized
   // before tail and on head.next being accurate if the current
   // thread is first in queue.
   Node t = tail; // Read fields in reverse initialization order 尾节点
   Node h = head;//头节点
   Node s;
   return h != t &&//头尾节点不是一个 即队列存在排队线程
      ((s = h.next) == null || s.thread != Thread.currentThread());//头节点的后续节点为空或者不是当前线程
}
```

虽然公平锁看起来在公平性上比非公平锁好，但是公平锁为此付出了大量线程切换的代价，而非公平锁在锁的获取上不能保证公平，就有可能出现锁饥饿，即有的线程多次获取锁而有的线程获取不到锁，没有大量的线程切换保证了非公平锁的吞吐量。


#### 多线程之间按顺序执行，实现 A -> B -> C 三个线程启动，要求如下：
1. A 打印 5 次，B 打印 10 次，C 打印 15 次；
1. A 打印 5 次，B 打印 10 次，C 打印 15 次；
1. 来 10 轮；

```java
// 标志位 A:1 B:2 C:3
private int number = 1;
// 锁
private Lock lock = new ReentrantLock();
// 唤醒条件
private Condition c1 = lock.newCondition();
private Condition c2 = lock.newCondition();
private Condition c3 = lock.newCondition();

public void print5() {
    lock.lock();
    try {
        // 1. 判断，避免虚假唤醒
        while (number != 1) {
            c1.await();
        }
        // 2. 打印5次
        System.out.print(Thread.currentThread().getName() + " \t");
        for (int i = 0; i < 5; i++) {
            System.out.print(i + " ");
        }
        System.out.println();
        // 3. 通知
        number = 2;
        c2.signal();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}

public void print10(){
   // ...
}

public void print15(){
   // ...
}

public static void main(String[] args) {
    ShareResource shareResource = new ShareResource();
    new Thread(() -> {
        for (int i = 0; i < 10; i++) {
            shareResource.print5();
        }
    }, "A").start();
    new Thread(() -> {
        for (int i = 0; i < 10; i++) {
            shareResource.print10();
        }
    }, "B").start();
    new Thread(() -> {
        for (int i = 0; i < 10; i++) {
            shareResource.print15();
        }
    }, "C").start();
}
```

打印结果：

```
A 	0 1 2 3 4 
B 	0 1 2 3 4 5 6 7 8 9 
C 	0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 
A 	0 1 2 3 4 
B 	0 1 2 3 4 5 6 7 8 9 
C 	0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 
...
```