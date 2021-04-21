---
title: 尚硅谷Java大厂面试题全集

categories:
- 面试题

date: 2021-01-11
---
#### 线程池如何使用？
单个线程，主要特点如下：
1. 创建一个单线程化的线程池，它只会用一唯一的工作线程来执行任务，保证所有任务按照指定顺序执行。
1. newSingleThreadExecutor 将 corePollSize 和 maximumPollSize 都设置为 1，它使用的是 LinkedBlockQueue。
1. 适用一个任务一个任务执行的场景。

```java
public static ExecutorService newSingleThreadExecutor() {
   return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0L, 
                                                                         TimeUnit.MILLISECONDS, 
                                                                         new LinkedBlockingQueue<Runnable>()));
}

ExecutorService executor = Executors.newSingleThreadExecutor();
for (int i = 0; i < 3; i++) {
    executor.execute(() -> System.out.println(Thread.currentThread().getName() + "\t 办理业务"));
}
executor.shutdown();

pool-1-thread-1	 办理业务
pool-1-thread-1	 办理业务
pool-1-thread-1	 办理业务
```

固定线程数
1. 执行长期任务，性能好很多。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
   return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}

ExecutorService executor = Executors.newFixedThreadPool(2);
for (int i = 0; i < 3; i++) {
    executor.execute(() -> System.out.println(Thread.currentThread().getName() + "\t 办理业务"));
}
executor.shutdown();

pool-1-thread-2	 办理业务
pool-1-thread-1	 办理业务
pool-1-thread-2	 办理业务
```

无限制线程数，主要特点如下：
1. 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
1. 任务来了就创建线程，当线程空闲超过 60 秒，就销毁线程。
1. 适用执行很多短期异步的小程序或者负载较轻的服务。

```java
public static ExecutorService newCachedThreadPool() {
   return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}

ExecutorService executor = Executors.newCachedThreadPool();
for (int i = 0; i < 3; i++) {
    executor.execute(() -> System.out.println(Thread.currentThread().getName() + "\t 办理业务"));
}
executor.shutdown();

pool-1-thread-2	 办理业务
pool-1-thread-3	 办理业务
pool-1-thread-1	 办理业务
```
#### 为什么用线程池，优势？
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放进队列，然后在线程创建后启动这些任务。如果线程数量超过了最大数量，则在队列中排队等候。等其它线程执行完毕，再从队列中取出任务来执行。

线程池的主要特点为：线程复用、控制最大并发数、管理线程。

线程池的好处有：
1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的销毁。
1. 提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行。
1. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性。使用线程池可以进行统一的分配，调优和监控。

#### 线程池有几个重要的参数？
1. corePoolSize 线程池中的常驻核心线程数；
1. maximumPollSize 线程池能够容纳同时执行的最大线程数，此值必须大于等于1；
1. keepAliveTime 多余的空闲线程的存活时间。当前线程池数量超过 corePoolSize 时，当空闲时间达到 keepAliveTime 值时，多余空闲线程会被销毁只剩下 corePoolSize 个线程为止；
1. unit keepAliveTime 的单位；
1. workQueue 任务队列，被提交但尚未被执行的任务；
1. threadFactory 表示生成线程池中工作线程的线程工厂，用于创建线程，一般默认即可；
1. handler 拒绝策略，表示当队列满了并且工作线程大于等于线程池的最大线程数时如何拒绝新的任务；

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {}
```
## 蚂蚁花呗一面（一个小时）
#### Java 容器有哪些？哪些是同步容器？哪些是并发容器？
#### ArrayList 和 LinkedList 的插入和访问的时间复杂度？
#### Java 反射原理，注解原理？
#### 新生代分为几个区？使用什么算法进行垃圾回收？为什么使用这个算法？
#### HashMap 在什么情况下会扩容，或者有哪些操作会导致扩容？
#### HashMap push 方法的执行过程？
#### HashMap 检测到 hash 冲突后，将元素插入在链表的末尾还是开头？
#### 1.8 还采用了红黑树，讲讲红黑树的特性，为什么人家一定要用红黑树而不是 AV了，B 树之类的？
#### https 和 http 的区别，也没有用过其它安全传输手段？
#### 线程池的工作原理，几个重要的参数，然后给了具体几个参数分析线程池会怎么做，最后问阻塞队列的作用是什么？
#### Linux 怎么查看系统负载情况？
#### 请详细描述 Spring MVC 处理请求全流程？
#### Spring 一个 bean 装配的过程？
#### 讲一讲 AtomicInteger、为什么要用 CAS 而不是 synchronized？

## 美团一面经验
#### 最近做的比较熟悉的项目是哪个，画一下项目技术架构图
#### JVM 老年代和新生代的比例？
#### YGC 和 FGC 发生的具体场景
#### jstack、jmap、jutil 分别的意义？如何线上排查 JVM 的相关问题？
#### 线程池的构造类的方法的 5 个参数的具体意义？
#### 单机上一个线程池正在处理服务，如果忽然断电怎么办（正在处理的和阻塞队列里面的请求怎么处理）
#### 使用无界阻塞队列会出现什么问题？
#### 接口如何处理重复请求？