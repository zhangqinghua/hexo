---
title: 分布式事务 TCC

categories:
- 分布式事务

date: 2021-03-16 00:00:04
---
本文主要介绍 TCC 的原理，以及从代码的角度上分析如何实现的；不涉及具体使用示例。本文分析的是 Github 中开源项目 [tcc-transaction](https://github.com/changmingxie/tcc-transaction) 的代码。当然 Github 上有多个 TCC 项目，但是他们原理相近，所以不过多介绍，有兴趣的小伙伴自行阅读源码。

## 项目架构
tcc-transaction 的架构由以下部分组成：
1. 一个完整的业务活动由一个主业务服务与若干从业务服务组成；
1. 主业务服务负责发起并完成整个业务活动。
1. 从业务服务提供TCC型业务操作。
1. 业务活动管理器控制业务活动的一致性，它登记业务活动中的操作，并在业务活动提交时进行 confirm 操作，在业务活动取消时进行 cancel 操作；

TCC 和 2PC/3PC 很像，不过 TCC 的事务控制都是业务代码层面的，而 2PC/3PC 则是资源层面的。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210317003130.png)

TCC 事务其实主要包含两个阶段：Try 阶段、Confirm/Cancel 阶段。

从 TCC 的逻辑模型上我们可以看到，TCC 的核心思想是，Try 阶段检查并预留资源，确保在 Confirm 阶段有资源可用，这样可以最大程度的确保 Confirm 阶段能够执行成功。

1. Try-尝试执行业务
   完成所有业务检查(一致性)

   预留必须业务资源(准隔离性)
1. Confirm-确认执行业务
   真正执行业务。

   不作任何业务检查。

   只使用 Try 阶段预留的业务资源。

   Confirm 操作必须保证幂等性。
1. Cancel-取消执行业务
   释放Try阶段预留的业务资源。

   Cancel操作必须保证幂等性。

## 示例分析
下面通过一个示例来讨论 TCC 事务：Tom 需要给 Tracy 转 10 元，当使用 TCC 解决这种事务时，应该如何去做呢？

#### 转账面临的主要问题
我们考虑一下这个转账过程面临的问题：
1. 需要确保 Tom 账户余额不少于10元；
1. 需要确保账户余额的正确性，例如：假设 Tom 只有 10 元钱，但是 Tom 同时给 Tracy、Angle 转账 10 元；Tom 给其他人转账时，也可能收到其他人转过来的钱，此时账户的余额不能出现错乱（Tracy 账户也面临过类似的问题）；
1. 当并发量比较大时，要能够确保性能；

#### TCC 解决问题的思路
TCC 解决分布式事物的思路是，一个大事务拆解成多个小事务。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/002.png)

#### TCC 处理逻辑
使用 TCC 事务时，伪代码如下所示：

```java
@Compensable(confirmMethod = "transferConfirm", cancelMethod = "transferCancel")
@Transactional
public void transferTry(long fromAccountId, long toAccountId, int amount) {
    //检查Tom账户
    //锁定Tom账户
    //锁定Tracy账户
}

@Transactional
public void transferConfirm(long fromAccountId, long toAccountId, int amount) {
    //tom账户-10元
    //tracy账户+10元
}

@Transactional
public void transferCancel(long fromAccountId, long toAccountId, int amount) {
    //解除Tom账户锁定
    //接触Tracy账户锁定
}
```

在 Try 逻辑中需要确保 Tom 账户的余额足够，并锁定需要使用的资源（Tom、Tracy 账户）；如果这一步操作执行成功（没有出现异常），那么将执行 Confirm 方法，如果执行失败，那么将执行 Cancel 方法。注意 Confirm、Cancel 需要做好幂等。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/006.png)

## 原理分析
在上面的 TCC 事务中，转账操作其实涉及六次操作，实际项目中，在任何一个步骤都可能失败，那么当任何一个步骤失败时，TCC 框架是如何做到数据一致性的呢？

#### 整体流程图
以下为 TCC 的处理流程图，它可以确保不管是在 Try 阶段，还是在 Confirm/Cancel 阶段都可以确保数据的一致性。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/007.png)

从流程图上可以看到，TCC 依赖于一条事务处理记录，在开始 TCC 事务前标记创建此记录，然后在 TCC 的每个环节持续更新此记录的状态，这样就可以知道事务执行到那个环节了，当一次执行失败，进行重试时同样根据此数据来确定当前阶段，并判断应该执行什么操作。

因为存在失败重试的逻辑，所以 Cancel、Commit 方法必须实现幂等。其实在分布式开发中，凡是涉及到写操作的地方都应该实现幂等。

#### TCC 核心处理逻辑
因为使用了 `@Compensable` 注解，所以当调用 `transferTry` 方法前，首先进入代理类中。在 TCC 中有两个 `Interceptor` 会对 `@Compensable` 标注的方法生效，他们分别是：`CompensableTransactionInterceptor`（TCC 主要逻辑在此 `Interceptor` 中完成）、`ResourceCoordinatorInterceptor`（处理资源相关的事宜）。

`CompensableTransactionInterceptor#interceptCompensableMethod` 是 TCC 的核心处理逻辑。`interceptCompensableMethod` 封装请求数据，为 TCC 事务做准备，源码如下：

```java
public Object interceptCompensableMethod(ProceedingJoinPoint pjp) throws Throwable {
    Method method = CompensableMethodUtils.getCompensableMethod(pjp);
    Compensable compensable = method.getAnnotation(Compensable.class);
    Propagation propagation = compensable.propagation();
    TransactionContext transactionContext = FactoryBuilder.factoryOf(compensable.transactionContextEditor())
                                                          .getInstance()
                                                          .get(pjp.getTarget(), method, pjp.getArgs());
    boolean asyncConfirm = compensable.asyncConfirm();
    boolean asyncCancel = compensable.asyncCancel();
    boolean isTransactionActive = transactionManager.isTransactionActive();
    if (!TransactionUtils.isLegalTransactionContext(isTransactionActive, propagation, transactionContext)) {
        throw new SystemException("no active compensable transaction while propagation is mandatory for method " + method.getName());
    }
    MethodType methodType = CompensableMethodUtils.calculateMethodType(propagation, isTransactionActive, transactionContext);
    switch (methodType) {
        case ROOT:
            return rootMethodProceed(pjp, asyncConfirm, asyncCancel);
        case PROVIDER:
            return providerMethodProceed(pjp, transactionContext, asyncConfirm, asyncCancel);
        default:
            return pjp.proceed();
    }
}
```

`rootMethodProceed` 是 TCC 和核心处理逻辑，实现了对 Try、Confirm、Cancel 的执行，源码如下：

```java
private Object rootMethodProceed(ProceedingJoinPoint pjp, boolean asyncConfirm, boolean asyncCancel) throws Throwable {
    Object returnValue = null;
    Transaction transaction = null;
    try {
        transaction = transactionManager.begin();
        try {
           returnValue = pjp.proceed();
        } catch (Throwable tryingException) {
            if (isDelayCancelException(tryingException)) {
               transactionManager.syncTransaction();
            } else {
               logger.warn(String.format("compensable transaction trying failed. transaction content:%s", JSON.toJSONString(transaction)), tryingException);
               transactionManager.rollback(asyncCancel);
            }
            throw tryingException;
        }
       transactionManager.commit(asyncConfirm);
    } finally {
        transactionManager.cleanAfterCompletion(transaction);
    }
    return returnValue;
}
```

在这个方法中我们看到，首先执行的是 `@Compensable` 注解标注的方法（Try），如果抛出异常，那么执行 rollback 方法（Cancel），否则执行 `commit` 方法（ Confirm ）。

#### 异常处理流程
考虑到在 Try、Cancel、Confirm 过程中都可能发生异常，所以在任何一步失败时，系统都能够要么回到最初（未转账）状态，要么到达最终（已转账）状态。下面讨论一下 TCC 代码层面是如何保证一致性的。

**Begin**
在前面的代码中，可以看到执行 try 之前，TCC 通过 `transactionManager.begin()` 开启了一个事务，这个 begin 方法的核心是：
1. 创建一个记录，用于记录事务执行到那个环节了；
1. 注册当前事务到 `TransactionManager` 中，在 Confirm、Cancel 过程中可以使用此 `Transaction` 来 commit 或者 rollback；

`TransactionManager#begin`方法：

```java
public Transaction begin() {
   Transaction transaction = new Transaction(TransactionType.ROOT);
   transactionRepository.create(transaction);
   registerTransaction(transaction);
   return transaction;
}
```

`CachableTransactionRepository#create` 创建一个用于标识事务执行环节的记录，然后将 `transaction` 放到缓存中区。代码如下：

```java
@Override
public int create(Transaction transaction) {
    int result = doCreate(transaction);
    if (result > 0) {
        putToCache(transaction);
    }
    return result;
}
```

`CachableTransactionRepository` 有多个子类（`FileSystemTransactionRepository`、`JdbcTransactionRepository`、`RedisTransactionRepository`、`ZooKeeperTransactionRepository`），通过这些类可以实现记录 db、file、redis、zk 等的解决方案。

**commit/rollback**
在commit、rollback中，都有这样一行代码，用于更新事务状态：

```java
transactionRepository.update(transaction);
```

这行代码将当前事务的状态标记为 commit/rollback，如果失败会抛出异常，不会执行后续的 Confirm/Cancel 方法；如果成功，才会执行 Confirm/Cancel 方法。

**Scheduler**
如果在 try/commit/rollback 过程中失败了，请求( `transferTry` 方法)将会立即返回，TCC 在这里引入了重试机制，即通过定时程序查询执行失败的任务，然后进行补偿操作。具体见：`TransactionRecovery#startRecover` 查询所有异常事务，然后逐个进行处理。注意重试操作有一个最大重试次数的限制，如果超过最大重试次数，此事务将会被忽略。

```java
public void startRecover() {
    List<Transaction> transactions = loadErrorTransactions();
   recoverErrorTransactions(transactions);
}

private List<Transaction> loadErrorTransactions() {
    long currentTimeInMillis = Calendar.getInstance().getTimeInMillis();
    TransactionRepository transactionRepository = transactionConfigurator.getTransactionRepository();
    RecoverConfig recoverConfig = transactionConfigurator.getRecoverConfig();
    return transactionRepository.findAllUnmodifiedSince(new Date(currentTimeInMillis - recoverConfig.getRecoverDuration() * 1000));
}

private void recoverErrorTransactions(List<Transaction> transactions) {
    for (Transaction transaction : transactions) {
        if (transaction.getRetriedCount() > transactionConfigurator.getRecoverConfig().getMaxRetryCount()) {
           logger.error(String.format("recover failed with max retry count,will not try again. txid:%s, status:%s,retried count:%d,transaction content:%s", transaction.getXid(), transaction.getStatus().getId(), transaction.getRetriedCount(), JSON.toJSONString(transaction)));
            continue;
        }
        if (transaction.getTransactionType().equals(TransactionType.BRANCH)
                && (transaction.getCreateTime().getTime() +
               transactionConfigurator.getRecoverConfig().getMaxRetryCount() *
                       transactionConfigurator.getRecoverConfig().getRecoverDuration() * 1000
                > System.currentTimeMillis())) {
            continue;
        }
        try {
           transaction.addRetriedCount();
            if (transaction.getStatus().equals(TransactionStatus.CONFIRMING)) {
               transaction.changeStatus(TransactionStatus.CONFIRMING);
                transactionConfigurator.getTransactionRepository().update(transaction);
                transaction.commit();
               transactionConfigurator.getTransactionRepository().delete(transaction);
            } else if (transaction.getStatus().equals(TransactionStatus.CANCELLING)
                    || transaction.getTransactionType().equals(TransactionType.ROOT)) {
               transaction.changeStatus(TransactionStatus.CANCELLING);
               transactionConfigurator.getTransactionRepository().update(transaction);
                transaction.rollback();
               transactionConfigurator.getTransactionRepository().delete(transaction);
            }
        } catch (Throwable throwable) {
            if (throwable instanceof OptimisticLockException
                    || ExceptionUtils.getRootCause(throwable) instanceof OptimisticLockException) {
               logger.warn(String.format("optimisticLockException happened while recover. txid:%s, status:%s,retried count:%d,transaction content:%s", transaction.getXid(), transaction.getStatus().getId(), transaction.getRetriedCount(), JSON.toJSONString(transaction)), throwable);
            } else {
               logger.error(String.format("recover failed, txid:%s, status:%s,retried count:%d,transaction content:%s", transaction.getXid(), transaction.getStatus().getId(), transaction.getRetriedCount(), JSON.toJSONString(transaction)), throwable);
            }
        }
    }
}
```

## 优点缺点
目前解决分布式事务的方案中，最稳定可靠的方案有：TCC、2PC/3PC、最终一致性。这三种方案各有优劣，有自己的适用场景。下面我们简单讨论一下 TCC 主要的优缺点。

TCC 的主要优点有：
1. 因为 Try 阶段检查并预留了资源，所以 Confirm 阶段一般都可以执行成功；
1. 资源锁定都是在业务代码中完成，不会 block 住 DB，可以做到对 DB 性能无影响；
1. TCC 的实时性较高，所有的 DB 写操作都集中在 Confirm 中，写操作的结果实时返回（失败时因为定时程序执行时间的关系，略有延迟）;

TCC 的主要缺点有：
1. 因为事务状态管理，将产生多次 DB 操作，这将损耗一定的性能，并使得整个TCC事务时间拉长；
1. 事务涉及方越多，Try、Confirm、Cancel 中的代码就越复杂，可复用性就越底（这一点主要是相对最终一致性方案而言的）。另外涉及方越多，这几个阶段的处理时间越长，失败的可能性也越高；