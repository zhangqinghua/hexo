---
title: ES6 数组操作

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:15
---
数组中新增了 `map` 和 `reduce` 方法。

## map
接收一个函数，将原数组中的所有元素用这个函数处理后放入新数组返回。

```js
let arr = ["1", "20", "-5", "3"];

console.log(arr);

arr = arr.map(s => parseInt(s));

console.log(arr);
```

## reduce
reduce 为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：初始值（或者上一次回调函数的返回值），当前元素值，当前索引，调用 `reduce` 的数组。

`callback` （执行数组中每个值的函数）包含四个参数：
1. `previousValue` 
   上一次调用回调返回的值，或者是提供的初始值。
2. `currentValue` 
   数组中当前被处理的元素。
3. `index` 
   当前元素在数组中的索引。
4. `array` 
   调用 `reduce` 的数组。

示例：

```js
const arr = [1, 20, -5, 3];

// 没 有 初 始 值 ： 19 
console.log(arr.reduce((a, b )=> a + b));

// 没 有 初 始 值 ： -300 
console.log(arr.reduce((a, b) => a * b));

// 指 定 初 始 值 ： 20
console.log(arr.reduce((a, b) => a + b, 1));

// 指 定 初 始 值 ： 0
console.log(arr.reduce((a, b) => a * b, 0));
```