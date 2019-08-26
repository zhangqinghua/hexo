---
title: JPA 笔记

tags:
- JPA

categories:
- Java

date: 2017-12-06
---

JPA 笔记

## 异常
- save the transient instance before flushing
	```java
	@Override
	public Image save(Image image) throws Exception {
	    // 如果一个list的一个元素为null，则会报错，需要将其移除
	    for (Tag tag : image.getTags()) {
	        if (tag.getId() == null) {
	            image.getTags().remove(tag);
	        }
	    }
	    return super.save(image);
	}
	```

## 自动维护创建时间和更新时间
在数据库里设置默认值current_timestamp可以维护创建时间，设置on update current_timestamp 可以维护更新时间。
```java
@Column(name = "create_time",insertable = false,updatable = false,columnDefinition="TIMESTAMP DEFAULT CURRENT_TIMESTAMP")
private Date createTime;
@Column(name = "update_time",insertable = false,updatable = false,columnDefinition="TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP")
private Date updateTime;
```