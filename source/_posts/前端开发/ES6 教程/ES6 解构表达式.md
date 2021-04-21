---
title: ES6 解构表达式

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:11
---
## 数组解构
```js
let arr = [1,2,3];
//以前我们想获取其中的值，只能通过角标。ES6 可以这样：
const [x,y,z] = arr;// x，y，z 将与 arr 中的每个位置对应来取值
// 然后打印
console.log(x,y,z);
```

## 对象解构
```js
const person = { name: "jack", age: 21,
language: ['java', 'js', 'css']
}
// 解构表达式获取值，将 person 里面每一个属性和左边对应赋值
const { name, age, language } = person;
// 等价于下面
// const name = person.name;
// const age = person.age;
// const language = person.language;
// 可以分别打印console.log(name); console.log(age);
console.log(language);

//扩展：如果想要将 name 的值赋值给其他变量，可以如下,nn 是新的变量名
const { name: nn, age, language } = person; console.log(nn);
console.log(age);
console.log(language);
```