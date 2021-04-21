---
title: ES6 对象优化

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:14
---
## 新增的 API
ES6 给 Object 拓展了许多新的方法，如：
1. `keys(obj)`
   获取对象的所有 key 形成的数组。
1. `values(obj)`
   获取对象的所有 value 形成的数组。
1. `entries(obj)`
   获取对象的所有 key 和 value 形成的二维数组。格式：`[[k1,v1],[k2,v2],...]`。
1. `assign(dest, ...src)` 
   将多个 `src` 对象的值 拷贝到 `dest` 中。（第一层为深拷贝，第二层为浅拷贝）。

```js
const person = { 
   name: "jack", 
   age: 21,
   language: ['java', 'js', 'css']
}

// ["name", "age", "language"] 
console.log(Object.keys(person));

// ["jack", 21, Array(3)] 
console.log(Object.values(person));

// [Array(2), Array(2), Array(2)]
console.log(Object.entries(person));


const target = { a: 1 }; 
const source1 = { b: 2 }; 
const source2 = { c: 3 };

// Object.assign 方法的第一个参数是目标对象，后面的参数都是源对象。
Object.assign(target, source1, source2);

// {a: 1, b: 2, c: 3}
console.log(target)
```

## 声明对象简写
```js
const age = 23
const name = "张三"

// 传统
const person1 = { age: age, name: name } 
console.log(person1)

// ES6：属性名和属性值变量名一样，可以省略
const person2 = { age, name }

// {age: 23, name: "张三"}
console.log(person2) 
```

## 对象函数属性简写
```js
let person = {
        name: "jack",
        // 之前写法
        eat1: function(food) {
                console.log(this.name + " eating " + food);
        },
        // 新的写法
        eat3(food) {
                console.log(this.name + " eating " + food);
        },
        // 箭头函数 this 不能使用对象.属性
        eat2: food => console.log(person.name + " eating " + food),
}
 
person.eat1("banana");
person.eat2("apple");
person.eat3("orage");
```

## 对象扩展运算符
拓展运算符 `...` 用于取出参数对象所有可遍历属性然后拷贝到当前对象。

```js
// 1. 拷贝对象（深拷贝）
let person = {name: "Amy", age: 15};
let somone = {... person};
// {name: "Amy", age: 15}
console.log(someone); 
 
// 2. 合并对象
let age = {age: 15};
let name = {name: "Amy"};
// 如果两个对象的字段名重复，后面对象字段会覆盖前面对象的字段值
let person = {...age, ...name}; 
```