---
title: SpringBoot 事务
categories:
- 后端开发
- SpringBoot 手册

date: 2021-06-14
---

#### 获取事务Id
SpringBoot无法直接获取事务Id，要判断是否同一个事务，可以通过数据库连接Id来确认。

```java
@Transactional
public void printConnect() {
    System.err.println("MerchantGroupController.test123 -------------------> Transactional");
    Map<Object, Object> map = TransactionSynchronizationManager.getResourceMap();
    for (Object key: map.keySet()) {
        System.err.println("key: " + key);
        System.err.println("value: " +  map.get(key));
    }
    System.err.println("MerchantGroupController.test123 -------------------> Transactional");
}
```

下面以一个 demo 来测试一下。

```java
@Transactional
public ClassA.method() {
   printConnect();
   ClassB.method();
   ClassC.method();
}


public ClassB.method() {

}

public ClassC.method() {

}

=====================
key: {
	CreateTime:"2021-06-14 14:53:04",
	ActiveCount:1,
	PoolingCount:0,
	CreateCount:1,
	DestroyCount:0,
	CloseCount:1,
	ConnectCount:2,
	Connections:[
	]
}
value: org.springframework.jdbc.datasource.ConnectionHolder@d5d122d3


key: {
	CreateTime:"2021-06-14 14:53:04",
	ActiveCount:1,
	PoolingCount:0,
	CreateCount:1,
	DestroyCount:0,
	CloseCount:1,
	ConnectCount:2,
	Connections:[
	]
}
value: org.springframework.jdbc.datasource.ConnectionHolder@d5d122d3


key: {
	CreateTime:"2021-06-14 14:53:04",
	ActiveCount:2,
	PoolingCount:0,
	CreateCount:2,
	DestroyCount:0,
	CloseCount:1,
	ConnectCount:3,
	Connections:[
	]
}
value: org.springframework.jdbc.datasource.ConnectionHolder@7bf06f97
```

可以看到 `ClassB.method` 和 `ClassA.method` 的数据库连接是同一个，所以他们属于同一个事务，能查询到未提交的数据。`ClassC.method` 开启了一个新的事务，他们的数据库连接不是同一个。