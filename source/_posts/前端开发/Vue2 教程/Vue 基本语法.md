---
title: Vue 基本语法

categories:
- 前端开发
- Vue2 教程

date: 2021-04-06 00:00:03
---
#### 声明式渲染
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
   <meta charset="UTF-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Document</title>
   <script src="./node_modules/vue/dist/vue.js"></script>
</head>
 
<body>
   <div id="app">
      <h1>{{name}} 非常帅</h1>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         name: "zhangsan"
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125205.png)

#### 双向绑定
使用 `v-model` 属性可以实现双向绑定功能，模型变化，视图变化，反之亦然。

```html
<!DOCTYPE html>
<html lang="en">
 
<head>
   <meta charset="UTF-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Document</title>
   <script src="./node_modules/vue/dist/vue.js"></script>
</head>
 
<body>
   <div id="app">
      <h1>{{name}} 非常帅，有{{num}}个人为他点赞</h1>
      <input type="text" v-model="num">
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         name: "zhangsan",
         num: 1
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125139.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125244.png)

#### 事件处理
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
   <meta charset="UTF-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Document</title>
   <script src="./node_modules/vue/dist/vue.js"></script>
</head>
 
<body>
   <div id="app">
      <h1>{{name}} 非常帅，有{{num}}个人为他点赞</h1>
      <input type="text" v-model="num">
      <button v-on:click="num++">点赞</button>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         name: "zhangsan",
         num: 1
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125334.png)
#### 声明方法
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
   <meta charset="UTF-8">
   <meta http-equiv="X-UA-Compatible" content="IE=edge">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Document</title>
   <script src="./node_modules/vue/dist/vue.js"></script>
</head>
 
<body>
   <div id="app">
      <h1>{{name}} 非常帅，有{{num}}个人为他点赞</h1>
      <input type="text" v-model="num">
      <button v-on:click="num++">点赞</button>
      <button v-on:click="cancel">清空</button>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         name: "zhangsan",
         num: 1
      },
      methods: {
         cancel() {
            this.num = 0;
         }
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125413.png)

点击清空后，数字重置为 0。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406125539.png)

## 总结
1. 创建 Vue 实例，关联页面的模版；
1. 将自己的数据（data）渲染到关联的模版，响应式的；
1. 指令来简化对 DOM 的一些操作；
1. 声明方法来做更复杂的操作，methods 里面可以封装方法；