---
title: ES6 模块化

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:17
---

模块化就是把代码进行拆分，方便重复利用。类似 Java 中的导包，要使用一个包，必须先导包。

模块功能主要由两个命令构成：`export` 和 `import`：

1. `export` 命令用于规定模块的对外接口；

2. `import` 命令用于导入其他模块提供的功能；

## user.js
```js
var name = "Jack";
var age = 21;

function add(a, b) {
        return a + b;
}

export(name, age, add);
```

## hello.js
```js
export const util = {
	sum(a, b) {
		return a + b;
	}
}
```

## main.js
```js
import util from "./hello.js";
import {name, add} from "./user.js";

console.log("name: " + name);

console.log(add(10, 20));

console.log(util.sum(10, 20));
```

> export 不仅可以导出对象，一切 JS 变量都可以导出，例如：基本类型变量、函数、数组、对象。
