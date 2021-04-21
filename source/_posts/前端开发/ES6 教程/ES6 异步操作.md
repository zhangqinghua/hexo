---
title: ES6 异步操作

categories:
- 前端开发
- ES6 教程

date: 2021-04-06 00:00:16
---
在 JavaScript 的世界中，所有代码都是单线程执行的。由于这个“缺陷”，导致 JavaScript 的所有网络操作，浏览器事件，都必须是异步执行。异步执行可以用回调函数实现。一旦有一连串的 Ajax 请求 a, b, c, d... 后面的请求依赖前面的请求结果，就需要层层嵌套。

这种缩进和层层嵌套的方式，非常容易造成上下文代码混乱，我们不得不非常小心翼翼处理内层函数与外层函数的数据，一旦内层函数使用了上层函数的变量，这种混乱程度就会加剧 总之，这种层叠上下文的层层嵌套方式，着实增加了神经的紧张程度。

我们可以通过 `Promise` 解决以上问题。

假设我们要查询一个用户的课程分数，需要经过以下步骤：
1. 查询当前用户信息；
2. 按照当前用户 id 查询课程；
3. 按照当前课程 id 查询分数；

## Ajax 的写法
```js
$.ajax({
        url: "mock/user.json",
        success(data) {
                console.log("查询用户：" + data);
                $.ajax({
                        url: "mock/user_corse_${data.id}.json",
                        success(data) {
                                console.log("用户课程：" + data);
                                $.ajax({
                                        url: "mock/corse_score.json",
                                        success(data) {
                                                console.log("课程分数：" + data);
                                        },
                                        error(error) {
                                                console.log("查询课程分数出现异常了");
                                        }
                                });
                        },
                        error(error) {
                                console.log("查询用户课程出现异常了");
                        }
                });
        },
        error(error) {
                console.log("查询用户信息出现异常了");
        }
});
```

## Promoise 写法
```js
new Promoise((resolve, reject) => {
   $.ajax({
      url: "mock/user.json",
      success(data) {
         console.log("查询数据成功：" + data);
         resolve(data);
      },
      error(error) {
         reject(error);
      }
   });
}).then((obj, resolve, reject) => {
   $.ajax({
      url: "mock/user_corse_${data.id}.json",
      success(data) {
         console.log("查询数据成功：" + data);
         resolve(data);
      },
      error(error) {
         reject(error);
      }
   });
}).then((obj, resolve, reject) => {
   $.ajax({
      url: "mock/corse_score.json",
      success(data) {
         console.log("查询数据成功：" + data);
         resolve(data);
      },
      error(error) {
         reject(error);
      }
   });
}).catch(error => {
   console.log("查询数据失败：" + error);
});
```

## 改进写法
```js
function get(url, data) {
   return new Promise((resolve, reject) => {
      $.ajax({
         url: url,
         data: data,
         success: function (data) {
            resolve(data);
         },
         error: function (err) {
            reject(err);
         }
      });
   });
}
 
get(`mock/user.json`).then(data => {
   console.log("用户查询成功：" + data);
   return get(`mock/user_corse_${data.id}.json`);
}).then(data => {
   console.log("课程查询成功：" + data);
   return get(`mock/corse_score_${data.id}.json`);
}).then(data => {
   console.log("分数查询成功：" + data);
}).catch(err => {
   console.log("查询数据失败：" + error);
});
```

通过比较，我们知道了 `Promise` 的扁平化设计理念，也领略了这种上层设计带来的好处。我们的项目中会使用到这种异步处理的方式。