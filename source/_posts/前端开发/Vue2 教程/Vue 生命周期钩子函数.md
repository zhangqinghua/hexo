---
title: Vue 生命周期钩子函数

categories:
- 前端开发
- Vue2 教程

date: 2021-04-06 00:00:07
---
## 生命周期
每个 Vue 实例在被创建时都要经过一系列的初始化过程 ：创建实例，装载模板，渲染模板等等。Vue 为生命周期中的每个状态都设置了钩子函数（监听函数）。每当 Vue 实例处于不同的生命周期时，对应的函数就会被触发调用。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406150932.png)

## 钩子函数
1. `beforeCreated`
   我们在用 Vue 时都要进行实例化，因此，该函数就是在 Vue 实例化时调用，也可以将他理解为初始化函数比较方便一点，在 Vue1.0 时，这个函数的名字就是 `init`。

1. `created`
   在创建实例之后进行调用。

1. `beforeMount`
   页面加载完成，没有渲染。如：此时页面还是 `{{name}}`。

1. `mounted`
   我们可以将他理解为原生 js 中的 `window.onload=function({.,.})`。
   
   它的功能就是： 在 DOM 文档渲染完毕之后将要执行的函数，该函数在 Vue1.0 版本中名字为 `compiled`。 此时页面中的 `{{name}}` 已被渲染成张三。

1. `beforeUpdate`
   组件更新之前。

1. `updated`
   组件更新之后。

1. `beforeDestroy`
   该函数将在销毁实例前进行调用 。

1. `destroyed`
   函数将在销毁实例时进行调用。

示例：

```html
<body>
   <div id="app">
      <span id="num">{{num}}</span>
      <button v-on:click="num++">赞！</button>
      <h2>
         {{name}}，非常帅！！！有{{num}}个人点赞。
      </h2>
   </div>
</body>
<script src="../node_modules/vue/dist/vue.js"></script>
<script>
   let app = new Vue({
      el: "#app", data: {
         name: "张三", num: 100
      },
      methods: {
         show() {
            return this.name;
         },
         add() {
            this.num++;
         }
      },
      beforeCreate() {
         console.log("=========beforeCreate=============");
         console.log("数据模型未加载：" + this.name, this.num);
         console.log(" 方 法 未 加 载 ：" + this.show());
         console.log("html 模板未加载：" + document.getElementById("num"));
      },
      created: function () {
         console.log("=========created=============");
         console.log("数据模型已加载：" + this.name, this.num);
         console.log(" 方 法 已 加 载 ：" + this.show());
         console.log("html 模板已加载：" + document.getElementById("num"));
         console.log("html 模板未渲染：" + document.getElementById("num").innerText);
      },
      beforeMount() {
         console.log("=========beforeMount=============");
         console.log("html 模板未渲染：" + document.getElementById("num").innerText);
      },
      mounted() {
         console.log("=========mounted=============");
         console.log("html 模板已渲染：" + document.getElementById("num").innerText);
      },
      beforeUpdate() {
         console.log("=========beforeUpdate=============");
         console.log(" 数 据 模 型 已 更 新 ：" + this.num);
         console.log("html 模板未更新：" + document.getElementById("num").innerText);
      },
      updated() {
         console.log("=========updated=============");
         console.log("数据模型已更新：" + this.num);
         console.log("html 模板已更新：" + document.getElementById("num").innerText);
      }
   });
</script>
```