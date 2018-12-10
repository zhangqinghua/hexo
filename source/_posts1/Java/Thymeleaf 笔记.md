---
title: Thymeleaf 专题

tags:
- Thymeleaf
- 专题

categories:
- Java

date: 2017-11-25
---

Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。它是一个开源的Java库，基于Apache License 2.0许可，由Daniel Fernández创建，该作者还是Java加密库Jasypt的作者。

## 遍历
```html
<input type="checkbox"
	th:each="tag, itar:${tags}"
	th:name="'tags['+${itar.index}+'].id'"
	th:value="${tag.id}"
	th:text="${tag.name}"
	th:checked="${image.tags.contains(tag)? 'true':'false'}">
```

## 三元表达式
```html
<input type="checkbox"
	th:each="tag, itar:${tags}"
	th:name="'tags['+${itar.index}+'].id'"
	th:value="${tag.id}"
	th:text="${tag.name}"
	th:checked="${image.tags.contains(tag)? 'true':'false'}">
```

## 日期格式化
只能用于Date类型的数据
```html
<td th:text="${#dates.format(joke.createTime, 'yyyy-MM-dd HH:mm:ss')}"/>
```

## include Error resolving template 加载模板错误
多了个/，例如`<th:replace='/base'></div>`，开发环境正常，服务器环境找不到，是因为开发环境和发布环境的处理是不一样的，一个走系统文件路径，一个走jar路径。
```java
// 要加载的模板，这个是找不到模板的写法，多了一个 /
templates//common/head.html
```

所以，开发环境得到的流路径：
```java
// 自动删除了多余的 / 所以能正常找到模板
/Users/alanwei/Documents/${project_path}/classes/templates/common/head.html
```

发布环境，因为在jar中找不到templates//common/head.html而挂掉