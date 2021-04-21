---
title: ES6 声明变量

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:10
---
## let 声明变量
```js
// var 声明的变量往往会越域
// let 声明的变量有严格局部作用域
{
   var a = 1; let b = 2;
}
console.log(a);	// 1
console.log(b);	// ReferenceError: b is not defined

// var 可以声明多次
// let 只能声明一次
var m = 1 var m = 2 let n = 3
// let n = 4 console.log(m)	// 2
console.log(n)	// Identifier 'n' has already been declared

// var 会变量提升
// let 不存在变量提升console.log(x);	// undefined var x = 10;
console.log(y);	//ReferenceError: y is not defined let y = 20;
```

## const 声明常量（只读变量）
```js
// 1. 声明之后不允许改变
// 2. 一但声明必须初始化，否则会报错
const a = 1;
a = 3; //Uncaught TypeError: Assignment to constant variable.
```