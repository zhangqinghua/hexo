---
title: ES6 函数优化

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:13
---

## 函数参数默认值
在 ES6 以前，我们无法给一个函数参数设置默认值，只能采用变通写法：

```js
function add(a, b) {
   // 判断 b 是否为空，为空就给默认值 1 
   b = b || 1;
   return a + b;
}

// 传一个参数
console.log(add(10));
```

现在可以这么写：直接给参数写上默认值，没传就会自动使用默认值：

```js
function add2(a , b = 1) {
   return a + b;
}
// 传一个参数
console.log(add2(10));
```

## 不定参数
不定参数用来表示不确定参数个数，形如，`...变量名`，由 `...` 加上一个具名参数标识符组成。具名参数只能放在参数列表的最后，并且有且只有一个不定参数：

```js
function fun(...values) { 
      console.log(values.length)
}

// 2
fun(1, 2)	      
// 4
fun(1, 2, 3, 4)   
```

## 箭头函数

```js
var print = function (obj) {
	console.log(obj);
}

// 箭头函数
var print = obj => console.log(obj);

print(100);
```

多个参数：

```js
var sum = function (a, b) { 
   return a + b;
}

// 简写为：当只有一行语句，并且需要返回结果时，可以省略 {} , 结果会自动返回。
var sum2 = (a, b) => a + b;

// 测 试 调 用 20
console.log(sum2(10, 10));

// 代码不止一行，可以用`{}`括起来
var sum3 = (a, b) => { 
   c = a + b;
   return c;
};

// 测 试 调 用 30
console.log(sum3(10, 20));
```

## 实战：箭头函数结合解构表达式
需求，声明一个对象，hello 方法需要对象的个别属性：
```js
// 以前的方式
const person = {
   name: "jack", age: 21,
   language: ['java', 'js', 'css']
}

function hello(person) {
   console.log("hello," + person.name)
}

// 现在的方式
var hello2 = ({ name }) => { console.log("hello," + name) };
// 测试
hello2(person);
```