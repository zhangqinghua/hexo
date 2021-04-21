---
title: ES6 字符串拓展

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:12
---
## 新增的  API
ES6 为字符串扩展了几个新的 API：
1. `includes()`
   返回布尔值，表示是否找到了参数字符串。
1. `startsWith()`
   返回布尔值，表示参数字符串是否在原字符串的头部。
1. `endsWith()`
   返回布尔值，表示参数字符串是否在原字符串的尾部。

```js
let str = "hello.vue"; 
console.log(str.startsWith("hello"));//true 
console.log(str.endsWith(".vue"));//true 
console.log(str.includes("e"));//true
console.log(str.includes("hello"));//true
```

## 字符串模板
模板字符串相当于加强版的字符串，用反引号 `，除了作为普通字符串，还可以用来定义多行字符串，还可以在字符串中加入变量和表达式。

多行字符串：

```js
let ss = `
         <div>
         <span>hello world<span>
         </div>
         ` 
console.log(ss)
```

字符串插入变量和表达式。变量名写在 ${} 中，${} 中可以放入 JavaScript 表达式：

```js
let name = "张三"; 
let age = 18;
let info = `我是${name}，今年${age}了`;
console.log(info)；
```

字符串中调用函数：

```js
function fun() {
   return "这是一个函数"
}

// O(∩_∩)O 哈哈~，这是一个函数
let sss = `O(∩_∩)O 哈 哈 ~，${fun()}`; console.log(sss); 
```