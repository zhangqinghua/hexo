---
title: SpringBoot 自定义注解
categories:
- SpringBoot

date: 2018-05-22
---

SpringBoot 自定义注解方式

## 自定义注解

1. 创建注解
    ```java
    @Target({ ElementType.METHOD })  
    @Retention(RetentionPolicy.RUNTIME)  
    public @interface SystemLogAnnotation {  
        String value();  
    }
    ```

1. 编写注解类切面
    ```java
    @Slf4j
    @Aspect
    @Component
    public class TransactionalAspect {

        @AfterReturning(pointcut = "@annotation(org.springframework.transaction.annotation.Transactional)()", returning = "obj")
        public Object Interceptor(Object obj) {
            try {
                if (obj != null && obj.getClass().getName().equals(ResponseData.class.getName())) {
                    if (!((ResponseData) obj).isSusscess()) {
                        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
                        log.warn("返回失败，事务回滚");
                    }
                }
            } catch (Exception e) {
                log.error("返回失败，异常：", e);
            }
            return obj;
        }
    }

1. 使用注解
    ```java
    @Override
    @Transactional
    public ResponseData<Boolean> update(TalentDTO talentDTO) {
        if (talentDTO == null) return ResponseData.builderError(false, "修改人才失败，dto为空");
    }
    ```

## 自定义事务注解

使用`@Transactional`事务后，根据方法的返回值决定回滚还是提交事务。 

1. 事务监听
    ```java
    @Slf4j
    @Aspect
    @Component
    public class TransactionalAspect {

        @AfterReturning(pointcut = "@annotation(org.springframework.transaction.annotation.Transactional)()", returning = "obj")
        public Object Interceptor(Object obj) {
            try {
                if (obj != null && obj.getClass().getName().equals(ResponseData.class.getName())) {
                    if (!((ResponseData) obj).isSusscess()) {
                        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
                        log.warn("返回失败，事务回滚");
                    }
                }
            } catch (Exception e) {
                log.error("返回失败，异常：", e);
            }
            return obj;
        }
    }
    ```

1. 使用注解
    ```java
    @Override
    @Transactional
    public ResponseData<Boolean> update(TalentDTO talentDTO) {
        if (talentDTO == null) return ResponseData.builderError(false, "修改人才失败，dto为空");
    }
    ```

## 操作日志注解