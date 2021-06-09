---
title: SpringBoot 事务源码

categories:
- 后端开发
- SpringBoot 源码

date: 2021-06-09
---
## 正常流程
#### 1. 依赖和业务代码
pom.xml 依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.13.RELEASE</version>
    </parent>

    <groupId>org.zhangqinghua.learn</groupId>
    <artifactId>springboot-learning</artifactId>
    <packaging>pom</packaging>
    <version>0.0.1</version>
    <modules>
        <module>springboot-sourcecode</module>
    </modules>
</project>
```

业务代码：

```java
@RestController
@RequestMapping("/api/test")
public class TestController {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @GetMapping("/demo")
    @Transactional
    public String demo() {
        System.out.println("demo...");
        long id = System.currentTimeMillis();
        String token = "token" + new Random().nextInt(10000);
        String sql = "insert into sys_user_token (user_id,token) values (" + id + ",'" + token + "')";
        jdbcTemplate.update(sql);
        return "HelloWorld";
    }
}
```

执行链：

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609231304.png)

#### 2. ReflectiveMethodInvocation 动态代理转入事务拦截器
```java
@Nullable
public Object proceed() throws Throwable {
   // 2. 第二次进来，执行业务方法
   if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
      return this.invokeJoinpoint();
   } 
   // 1. 第一次进来，被@Transactional注解修饰，执行事务逻辑
   else {
      Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
      // 1.1 其它注解类型
      if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
            InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
            Class<?> targetClass = this.targetClass != null ? this.targetClass : this.method.getDeclaringClass();
            return dm.methodMatcher.matches(this.method, targetClass, this.arguments) ? dm.interceptor.invoke(this) : this.proceed();
      } 
      // 1.2 @Transactional注解类型
      else {
            return ((MethodInterceptor)interceptorOrInterceptionAdvice).invoke(this);
      }
   }
}
```

#### 3. TransactionInterceptor 事务拦截器
```java
/**
 * 2. 准备执行事务逻辑
 */
@Override
@Nullable
public Object invoke(MethodInvocation invocation) throws Throwable {
   // org.zhangqinghua.transactional.api.TestController
   Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

   // param1: public java.lang.String org.zhangqinghua.transactional.api.TestController.demo()
   // param2: org.zhangqinghua.transactional.api.TestController
   // param3: unknow
   return invokeWithinTransaction(invocation.getMethod(), targetClass, invocation::proceed);
}
```

#### 4. TransactionAspectSupport 事务切面

```java
/**
 * General delegate for around-advice-based subclasses, delegating to several other template
 * methods on this class. Able to handle {@link CallbackPreferringPlatformTransactionManager}
 * as well as regular {@link PlatformTransactionManager} implementations and
 * {@link ReactiveTransactionManager} implementations for reactive return types.
 * @param method the Method being invoked
 * @param targetClass the target class that we're invoking the method on
 * @param invocation the callback to use for proceeding with the target invocation
 * @return the return value of the method, if any
 * @throws Throwable propagated from the target invocation
 */
@Nullable
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
      final InvocationCallback invocation) throws Throwable {

   // 1. 不知道是什么东西 If the transaction attribute is null, the method is non-transactional.
   TransactionAttributeSource tas = getTransactionAttributeSource();
   // 2. 应该是事务传播类型、事务隔离 PROPAGATION_REQUIRED,ISOLATION_DEFAULT
   final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
   // 3. 决定使用哪个事务管理器？
   final TransactionManager tm = determineTransactionManager(txAttr);

   // 4. 这里应该是响应式事务处理，现在没有用到
   if (this.reactiveAdapterRegistry != null && tm instanceof ReactiveTransactionManager) {
      ReactiveTransactionSupport txSupport = this.transactionSupportCache.computeIfAbsent(method, key -> {
         if (KotlinDetector.isKotlinType(method.getDeclaringClass()) && KotlinDelegate.isSuspend(method)) {
            throw new TransactionUsageException(
                  "Unsupported annotated transaction on suspending function detected: " + method +
                  ". Use TransactionalOperator.transactional extensions instead.");
         }
         ReactiveAdapter adapter = this.reactiveAdapterRegistry.getAdapter(method.getReturnType());
         if (adapter == null) {
            throw new IllegalStateException("Cannot apply reactive transaction to non-reactive return type: " +
                  method.getReturnType());
         }
         return new ReactiveTransactionSupport(adapter);
      });
      return txSupport.invokeWithinTransaction(
            method, targetClass, invocation, txAttr, (ReactiveTransactionManager) tm);
   }

   // 5. 这里应该是跟数据源有关了，debug看到ptm.dataSource="HikariDataSource (HikariPool-1)"
   PlatformTransactionManager ptm = asPlatformTransactionManager(tm);
   // 6. 这里应该是切入点 org.zhangqinghua.transactional.api.TestController.demo
   final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

   // 7. 事务逻辑
   if (txAttr == null || !(ptm instanceof CallbackPreferringPlatformTransactionManager)) {
      // 7.1 创建一个事务 Standard transaction demarcation with getTransaction and commit/rollback calls.
      TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

      // 7.2 执行业务逻辑，retVal是返回值：HelloWorld
      Object retVal;
      try {
         // This is an around advice: Invoke the next interceptor in the chain.
         // This will normally result in a target object being invoked.
         // 这里再回到 ReflectiveMethodInvocation.proceed执行业务方法
         retVal = invocation.proceedWithInvocation();
      }
      // 7.3 业务处理有异常，需要处理（回滚？）
      catch (Throwable ex) {
         // target invocation exception
         completeTransactionAfterThrowing(txInfo, ex);
         throw ex;
      }
      // 7.4 清理事务
      finally {
         cleanupTransactionInfo(txInfo);
      }

      // 7.5 返回结果处理（不知道有什么用）
      if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
         // Set rollback-only in case of Vavr failure matching our rollback rules...
         TransactionStatus status = txInfo.getTransactionStatus();
         if (status != null && txAttr != null) {
            retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
         }
      }

      // 7.6 字面意思是等返回值返回以后再提交事务
      commitTransactionAfterReturning(txInfo);
      return retVal;
   }

   // 8. 这里应该是跟事务无关的，没有执行到
   else {
      Object result;
      final ThrowableHolder throwableHolder = new ThrowableHolder();

      // It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.
      try {
         result = ((CallbackPreferringPlatformTransactionManager) ptm).execute(txAttr, status -> {
            TransactionInfo txInfo = prepareTransactionInfo(ptm, txAttr, joinpointIdentification, status);
            try {
               Object retVal = invocation.proceedWithInvocation();
               if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
                  // Set rollback-only in case of Vavr failure matching our rollback rules...
                  retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
               }
               return retVal;
            }
            catch (Throwable ex) {
               if (txAttr.rollbackOn(ex)) {
                  // A RuntimeException: will lead to a rollback.
                  if (ex instanceof RuntimeException) {
                     throw (RuntimeException) ex;
                  }
                  else {
                     throw new ThrowableHolderException(ex);
                  }
               }
               else {
                  // A normal return value: will lead to a commit.
                  throwableHolder.throwable = ex;
                  return null;
               }
            }
            finally {
               cleanupTransactionInfo(txInfo);
            }
         });
      }
      catch (ThrowableHolderException ex) {
         throw ex.getCause();
      }
      catch (TransactionSystemException ex2) {
         if (throwableHolder.throwable != null) {
            logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
            ex2.initApplicationException(throwableHolder.throwable);
         }
         throw ex2;
      }
      catch (Throwable ex2) {
         if (throwableHolder.throwable != null) {
            logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
         }
         throw ex2;
      }

      // Check result state: It might indicate a Throwable to rethrow.
      if (throwableHolder.throwable != null) {
         throw throwableHolder.throwable;
      }
      return result;
   }
}
```

#### 附：determineTransactionManager 返回一个事务管理器
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609232233.png)

```java
/**
 * Determine the specific transaction manager to use for the given transaction.
 */
@Nullable
protected TransactionManager determineTransactionManager(@Nullable TransactionAttribute txAttr) {
   // 1. （略过）Do not attempt to lookup tx manager if no tx attributes are set
   if (txAttr == null || this.beanFactory == null) {
      return getTransactionManager();
   }

   // 2. （略过）选取指定的事务管理器
   String qualifier = txAttr.getQualifier();
   if (StringUtils.hasText(qualifier)) {
      return determineQualifiedTransactionManager(this.beanFactory, qualifier);
   }
   // 3. （略过）选取指定的事务管理器
   else if (StringUtils.hasText(this.transactionManagerBeanName)) {
      return determineQualifiedTransactionManager(this.beanFactory, this.transactionManagerBeanName);
   }
   // 4. （执行）返回当前的事务管理器
   else {
      TransactionManager defaultTransactionManager = getTransactionManager();
      // 4.1 （略过）重新生成一个事务管理器
      if (defaultTransactionManager == null) {
         defaultTransactionManager = this.transactionManagerCache.get(DEFAULT_TRANSACTION_MANAGER_KEY);
         if (defaultTransactionManager == null) {
            defaultTransactionManager = this.beanFactory.getBean(TransactionManager.class);
            this.transactionManagerCache.putIfAbsent(
                  DEFAULT_TRANSACTION_MANAGER_KEY, defaultTransactionManager);
         }
      }
      return defaultTransactionManager;
   }
}
```

#### 附：asPlatformTransactionManager 强转成为PlatformTransactionManager事务管理器
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609232915.png)

```java
@Nullable
private PlatformTransactionManager asPlatformTransactionManager(@Nullable Object transactionManager) {
   // 1. 直接强转返回了
   if (transactionManager == null || transactionManager instanceof PlatformTransactionManager) {
      return (PlatformTransactionManager) transactionManager;
   }
   else {
      throw new IllegalStateException(
            "Specified transaction manager is not a PlatformTransactionManager: " + transactionManager);
   }
}
```

#### 附：createTransactionIfNecessary 创建一个事务
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609233406.png)

```java
/**
 * Create a transaction if necessary based on the given TransactionAttribute.
 * <p>Allows callers to perform custom TransactionAttribute lookups through
 * the TransactionAttributeSource.
 * @param txAttr the TransactionAttribute (may be {@code null})
 * @param joinpointIdentification the fully qualified method name
 * (used for monitoring and logging purposes)
 * @return a TransactionInfo object, whether or not a transaction was created.
 * The {@code hasTransaction()} method on TransactionInfo can be used to
 * tell if there was a transaction created.
 * @see #getTransactionAttributeSource()
 */
protected TransactionInfo createTransactionIfNecessary(@Nullable PlatformTransactionManager tm,
      @Nullable TransactionAttribute txAttr, final String joinpointIdentification) {

   // 1. 没有特别指定的话，使用方法名作为事务名称 If no name specified, apply method identification as transaction name.
   if (txAttr != null && txAttr.getName() == null) {
      txAttr = new DelegatingTransactionAttribute(txAttr) {
         @Override
         public String getName() {
            return joinpointIdentification;
         }
      };
   }

   // 2. 从事务管理器中创建一个新的事务
   TransactionStatus status = null;
   if (txAttr != null) {
      if (tm != null) {
         status = tm.getTransaction(txAttr);
      }
      else {
         if (logger.isDebugEnabled()) {
            logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
                  "] because no transaction manager has been configured");
         }
      }
   }
   return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
}
```

#### 附：tm.getTransaction 获取一个事务
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609233535.png)

```java
/**
 * This implementation handles propagation behavior. Delegates to
 * {@code doGetTransaction}, {@code isExistingTransaction}
 * and {@code doBegin}.
 * @see #doGetTransaction
 * @see #isExistingTransaction
 * @see #doBegin
 */
@Override
public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
      throws TransactionException {

   // 1. （略过）Use defaults if no transaction definition given.
   TransactionDefinition def = (definition != null ? definition : TransactionDefinition.withDefaults());

   Object transaction = doGetTransaction();
   boolean debugEnabled = logger.isDebugEnabled();

   // 2. （略过）返回一个已经存在的事务
   if (isExistingTransaction(transaction)) {
      // Existing transaction found -> check propagation behavior to find out how to behave.
      return handleExistingTransaction(def, transaction, debugEnabled);
   }

   // 3. （略过）校验之类的 Check definition settings for new transaction.
   if (def.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
      throw new InvalidTimeoutException("Invalid transaction timeout", def.getTimeout());
   }

   // 4. （略过）No existing transaction found -> check propagation behavior to find out how to proceed.
   if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
      throw new IllegalTransactionStateException(
            "No existing transaction found for transaction marked with propagation 'mandatory'");
   }
   // 5. （执行）开启一个事务，并返回
   else if (def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
         def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
         def.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
      SuspendedResourcesHolder suspendedResources = suspend(null);
      if (debugEnabled) {
         logger.debug("Creating new transaction with name [" + def.getName() + "]: " + def);
      }
      try {
         return startTransaction(def, transaction, debugEnabled, suspendedResources);
      }
      catch (RuntimeException | Error ex) {
         resume(null, suspendedResources);
         throw ex;
      }
   }
   else {
      // Create "empty" transaction: no actual transaction, but potentially synchronization.
      if (def.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT && logger.isWarnEnabled()) {
         logger.warn("Custom isolation level specified but no actual transaction initiated; " +
               "isolation level will effectively be ignored: " + def);
      }
      boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
      return prepareTransactionStatus(def, null, true, newSynchronization, debugEnabled, null);
   }
}
```

#### 附：startTransaction
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609233916.png)

```java
/**
 * Start a new transaction.
 */
private TransactionStatus startTransaction(TransactionDefinition definition, Object transaction,
      boolean debugEnabled, @Nullable SuspendedResourcesHolder suspendedResources) {
   // 1. true
   boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
   // 2. 生成一个事务
   DefaultTransactionStatus status = newTransactionStatus(
         definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
   // 3. 开始是一个事务
   doBegin(transaction, definition);
   prepareSynchronization(status, definition);
   return status;
}
```

#### 附：DataSourceTransactionManager.doBegin
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609234150.png)

```java
protected void doBegin(Object transaction, TransactionDefinition definition) {
   // 1. 不知道干嘛用的，强转吧
   DataSourceTransactionManager.DataSourceTransactionObject txObject = (DataSourceTransactionManager.DataSourceTransactionObject)transaction;
   Connection con = null;

   try {
      // 2. （执行）从数据源获取一个连接，并设置在当前事务的ConnectionHolder上
      if (!txObject.hasConnectionHolder() || txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
            Connection newCon = this.obtainDataSource().getConnection();
            if (this.logger.isDebugEnabled()) {
               this.logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
            }

            txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
      }

      // 3. 给这个连接设置一下属性
      txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
      con = txObject.getConnectionHolder().getConnection();
      Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
      txObject.setPreviousIsolationLevel(previousIsolationLevel);
      txObject.setReadOnly(definition.isReadOnly());

      // 4. 禁用自动提交（这样子事务才能起作用）
      if (con.getAutoCommit()) {
            txObject.setMustRestoreAutoCommit(true);
            if (this.logger.isDebugEnabled()) {
               this.logger.debug("Switching JDBC Connection [" + con + "] to manual commit");
            }

            con.setAutoCommit(false);
      }

      this.prepareTransactionalConnection(con, definition);
      txObject.getConnectionHolder().setTransactionActive(true);
      int timeout = this.determineTimeout(definition);
      if (timeout != -1) {
            txObject.getConnectionHolder().setTimeoutInSeconds(timeout);
      }

      // 5. 将这个链接绑定在当前线程上（好像是）
      if (txObject.isNewConnectionHolder()) {
            TransactionSynchronizationManager.bindResource(this.obtainDataSource(), txObject.getConnectionHolder());
      }

   } catch (Throwable var7) {
      if (txObject.isNewConnectionHolder()) {
            DataSourceUtils.releaseConnection(con, this.obtainDataSource());
            txObject.setConnectionHolder((ConnectionHolder)null, false);
      }

      throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", var7);
   }
}
```

#### 附：DataSourceTransactionManager.prepareTransactionalConnection
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609234524.png)

```java
protected void prepareTransactionalConnection(Connection con, TransactionDefinition definition) throws SQLException {
   // 没有执行到，我靠
   if (this.isEnforceReadOnly() && definition.isReadOnly()) {
      Statement stmt = con.createStatement();
      Throwable var4 = null;

      try {
            stmt.executeUpdate("SET TRANSACTION READ ONLY");
      } catch (Throwable var13) {
            var4 = var13;
            throw var13;
      } finally {
            if (stmt != null) {
               if (var4 != null) {
                  try {
                        stmt.close();
                  } catch (Throwable var12) {
                        var4.addSuppressed(var12);
                  }
               } else {
                  stmt.close();
               }
            }

      }
   }
}
```

#### 附：TransactionSynchronizationManager.bindResource 将连接绑定在当前线程上
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210609234846.png)

```java
private static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal<>("Transactional resources");

/**
 * Bind the given resource for the given key to the current thread.
 * @param key the key to bind the value to (usually the resource factory)
 * @param value the value to bind (usually the active resource object)
 * @throws IllegalStateException if there is already a value bound to the thread
 * @see ResourceTransactionManager#getResourceFactory()
 */
public static void bindResource(Object key, Object value) throws IllegalStateException {
   // 1. 就是上面那个key
   Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
   Assert.notNull(value, "Value must not be null");

   // 2. 不知道干嘛用的
   Map<Object, Object> map = resources.get();
   // set ThreadLocal Map if none found
   if (map == null) {
      map = new HashMap<>();
      resources.set(map);
   }

   // 3. null key应该就是连接池了，value是连接
   Object oldValue = map.put(actualKey, value);
   // 4. 没有执行到 Transparently suppress a ResourceHolder that was marked as void...
   if (oldValue instanceof ResourceHolder && ((ResourceHolder) oldValue).isVoid()) {
      oldValue = null;
   }
   // 5. 没有执行到
   if (oldValue != null) {
      throw new IllegalStateException("Already value [" + oldValue + "] for key [" +
            actualKey + "] bound to thread [" + Thread.currentThread().getName() + "]");
   }
   // 6. 没有执行到
   if (logger.isTraceEnabled()) {
      logger.trace("Bound value [" + value + "] for key [" + actualKey + "] to thread [" +
            Thread.currentThread().getName() + "]");
   }
}
```