---
title: Vue 组件化

categories:
- 前端开发
- Vue2 教程

date: 2021-04-06 00:00:06
---
在大型应用开发的时候，页面可以划分成很多部分。往往不同的页面，也会有相同的部分。例如可能会有相同的头部导航。

但是如果每个页面都独自开发，这无疑增加了我们开发的成本。所以我们会把页面的不同部分拆分成独立的组件，然后在不同页面就可以共享这些组件，避免重复开发。

在 Vue 里，所有的 Vue 实例都是组件。例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。

## 全局组件
我们通过 Vue 的 `component` 方法来定义一个全局组件。

```html
<div id="app">
   <!--使用定义好的全局组件-->
   <counter></counter>
</div>

<script src="../node_modules/vue/dist/vue.js"></script>
<script type="text/javascript">
   // 定义全局组件，两个参数：1，组件名称。2，组件参数
   Vue.component("counter", {
      template: '<button v-on:click="count++">你点了我 {{ count }} 次，我记住了.</button>',
      data() {
         return {
            count: 0
         }
      }
   });

   let app = new Vue({
      el: "#app"
   });
</script>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406144753.png)

1. 组件其实也是一个 Vue 实例，因此它在定义时也会接收：`data`、`methods`、生命周期函数等；
1. 不同的是组件不会与页面的元素绑定，否则就无法复用了，因此没有 `el` 属性；
1. 但是组件渲染需要 HTML 模板，所以增加了 `template` 属性，值就是 HTML 模板；
1. 全局组件定义完毕，任何 Vue 实例都可以直接在 HTML 中通过组件名称来使用组件了；
1. `data` 必须是一个函数，不再是一个对象；

## 组件的复用
定义好的组件，可以任意复用多次：

```html
<div id="app">
   <!--使用定义好的全局组件-->
   <counter></counter>
   <counter></counter>
   <counter></counter>
</div>
```

> 组件的 data 属性必须是函数！
> 一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝；

## 局部组件
一旦全局注册，就意味着即便以后你不再使用这个组件，它依然会随着 Vue 的加载而加载。因此，对于一些并不频繁使用的组件，我们会采用局部注册。

我们先在外部定义一个对象，结构与创建组件时传递的第二个参数一致：

```html
<script type="text/javascript">
   const counter = {
      template: '<button v-on:click="count++">你点了我 {{ count }} 次，我记住了.</button>',
      data() {
         return {
            count: 0
         }
      }
   };
</script>
```

然后在 Vue 中使用它：

```html
<script type="text/javascript">
   let app = new Vue({
      el: "#app", 
      components: {
         counter: counter // 将定义的对象注册为组件
      }
   });
</script>
```

`components` 就是当前 vue 对象子组件集合：
1. 其 `key` 就是子组件名称；
1. 其值就是组件对象名；
