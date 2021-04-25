---
title: 并发编程-AQS 机制

categories:
- 后端开发
- Java 并发编程

date: 2020-06-13
---
从是什么、有什么、为什么、如何用、底层原理等几个维度分析 AQS 机制。

#### AQS 简介
AQS 是 `AbstractQueuedSynchronizer` 这个抽象类的简称，翻译过来就是抽象队列同步器。AQS 为不同场景提供了实现锁及同步机制的基本框架，为同步状态的原子性管理、线程的阻塞、线程的解除阻塞及排队管理提供了一种通用的机制。

AQS 将线程封装到一个 Node 里面，并维护一个 CHL Node FIFO 队列，它是一个非阻塞的 FIFO 队列，也就是说在并发条件下往此队列做插入或移除操作不会阻塞，是通过自旋锁和 CAS 保证节点插入和移除的原子性，实现无锁快速插入。

![](https://upload-images.jianshu.io/upload_images/53727-ae36db58241c256b.png?imageMogr2/auto-orient/strip|imageView2/2/w/852/format/webp)



#### AQS 基本使用
当我们谈论 AQS 时，更多的是指那些利用 AQS 实现的同步工具类。比如 `Semaphore`、`CountDownLatch`、`ReentrantLock` 等。

#### AQS 横向对比

#### AQS 底层原理

#### 附：CountDownLatch
`CountDownLatch` 闭锁可以让一些线程阻塞直到另一些线程完成一系列操作后才被唤醒，主要有 `await` 和 `countDown` 两个方法：
1. 当一个或多个线程调用 `await` 方法时，调用线程会被阻塞；
1. 其它线程调用 `countDown` 方法会将计数器减 1（调用 `countDown` 方法的线程不会被阻塞）。当计数器的值变为 0 时，因调用 `await` 方法被阻塞的线程会被唤醒，继续执行；

来看下面一个例子，多个线程没有执行完毕，主线程就结束了：

```java
for (int i = 1; i <= 6; i++) {
   new Thread(() -> System.out.println(Thread.currentThread().getName() + "离开了教室"), "同学" + i).start();
}
System.out.println("===============班长最后走人");

---------------------------------------------------------打印信息----------------------------------------------------------
同学1离开了教室
===============班长最后走人
同学4离开了教室
同学3离开了教室
同学2离开了教室
同学6离开了教室
同学5离开了教室
```

加上了 `CountDownLatch` 后，主线程会等待其它线程执行完毕才执行：

```java
CountDownLatch countDownLatch = new CountDownLatch(5);
for (int i = 1; i <= 6; i++) {
   new Thread(() -> {
         System.out.println(Thread.currentThread().getName() + "离开了教室");
         countDownLatch.countDown();
   }, "同学" + i).start();
}
countDownLatch.await();
System.out.println("===============班长最后走人");

---------------------------------------------------------打印信息----------------------------------------------------------
同学1离开了教室
同学3离开了教室
同学2离开了教室
同学4离开了教室
同学5离开了教室
同学6离开了教室
===============班长最后走人
```

#### 附：CyclicBarrier
`CyclicBarrier` 栅栏类似于闭锁，它能阻塞一组线程直到某个事件的发生。栅栏与闭锁的关键区别在于，所有的线程必须同时到达栅栏位置，才能继续执行。闭锁用于等待事件，而栅栏用于等待其他线程。

男生版：集齐 7 颗龙珠才能召唤神龙。

女生版：人到齐了才能开会。

```java
CyclicBarrier barrier = new CyclicBarrier(7, () -> System.out.println("=========召唤神龙"));
for (int i = 1; i <= 7; i++) {
   final int tempI = i;
   new Thread(() -> {
         System.out.println("收集到了第" + tempI + "颗龙珠");
         try {
            barrier.await();
         } catch (InterruptedException e) {
            e.printStackTrace();
         } catch (BrokenBarrierException e) {
            e.printStackTrace();
         }
         System.out.println("第" + tempI + "颗龙珠开始召唤");
   }).start();
}

---------------------------------------------------------打印信息----------------------------------------------------------
收集到了第1颗龙珠
收集到了第4颗龙珠
收集到了第5颗龙珠
收集到了第3颗龙珠
收集到了第6颗龙珠
收集到了第2颗龙珠
收集到了第7颗龙珠
=========召唤神龙
第4颗龙珠开始召唤
第1颗龙珠开始召唤
第3颗龙珠开始召唤
第5颗龙珠开始召唤
第7颗龙珠开始召唤
第6颗龙珠开始召唤
第2颗龙珠开始召唤
```

#### 附：Semaphore
Semaphore 信号量主要用于两个目的：一个是用于多个共享资源的互诉使用，另一个用于并发线程数的控制。

来一个争停车位的例子：

```java
// 1. 模拟3个停车位
Semaphore semaphore = new Semaphore(3);

// 2. 模拟6个人抢车位
for (int i = 1; i <= 6; i++) {
   new Thread(() -> {
         try {
            int time = new Random().nextInt(5) + 3;

            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + "抢到车位，休息" + time + "秒");

            TimeUnit.SECONDS.sleep(time);
            System.out.println(Thread.currentThread().getName() + "离开了车位");
         } catch (InterruptedException e) {
            e.printStackTrace();
         } finally {
            semaphore.release();
         }
   }, "T" + i).start();
}

---------------------------------------------------------打印信息----------------------------------------------------------
T3抢到车位，休息6秒
T2抢到车位，休息4秒
T4抢到车位，休息3秒
T4离开了车位
T6抢到车位，休息5秒
T2离开了车位
T1抢到车位，休息5秒
T3离开了车位
T5抢到车位，休息7秒
T6离开了车位
T1离开了车位
T5离开了车位
```