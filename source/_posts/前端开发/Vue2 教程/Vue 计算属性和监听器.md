---
title: Vue 计算属性和监听器

categories:
- 前端开发
- Vue2 教程

date: 2021-04-06 00:00:05
---

## 计算属性
某些结果是基于之前数据实时计算出来的，我们可以利用计算属性来完成。

```html
<div id="app">
   <ul>
      <li>西游记：价格{{xyjPrice}}，数量：
         <input type="number" v-model="xyjNum">
      </li>
      <li>水浒传：价格{{shzPrice}}，数量：
         <input type="number" v-model="shzNum">
      </li>
      <li>总价：{{totalPrice}}</li>
   </ul>
</div>
<script src="../node_modules/vue/dist/vue.js"></script>
<script type="text/javascript"> let app = new Vue({
      el: "#app", data: {
         xyjPrice: 56.73,
         shzPrice: 47.98,
         xyjNum: 1,
         shzNum: 1
      },
      computed: {
         totalPrice() {
            return this.xyjPrice * this.xyjNum + this.shzPrice * this.shzNum;
         }
      },
   })
</script>
```

效果：只要依赖的属性发生变化，就会重新计算这个属性：

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406141623.png)

## 侦听
`watch` 可以让我们监控一个值的变化。从而做出相应的反应。

```html
<div id="app">
   <ul>
      <li>西游记：价格{{xyjPrice}}，数量：
         <input type="number" v-model="xyjNum">
      </li>
      <li>水浒传：价格{{shzPrice}}，数量：
         <input type="number" v-model="shzNum">
      </li>
      <li>总价：{{totalPrice}}</li>
      {{msg}}
   </ul>
</div>
<script src="../node_modules/vue/dist/vue.js"></script>
<script type="text/javascript"> let app = new Vue({
      el: "#app", data: {
         xyjPrice: 56.73,
         shzPrice: 47.98,
         xyjNum: 1,
         shzNum: 1,
         msg: "",
      },
      computed: {
         totalPrice() {
            return this.xyjPrice * this.xyjNum + this.shzPrice * th
         }
      },
      watch: {
         xyjNum(newVal, oldVal) {
            if (newVal >= 3) {
               this.msg = "西游记没有更多库存了"; this.xyjNum = 3;
            } else {
               this.msg = "";
            }
         }
      }
   })
</script>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406142235.png)

## 过滤器
过滤器不改变真正的 `data`，而只是改变渲染的结果，并返回过滤后的版本。在很多不同的情况下，过滤器都是有用的，比如尽可能保持 API 响应的干净，并在前端处理数据的格式。

示例：展示用户列表性别显示男女。

```html
<body>
   <div id="app">
      <table>
         <tr v-for="user in userList">
            <td>{{user.id}}</td>
            <td>{{user.name}}</td>
            <!-- 使用代码块实现，有代码侵入 -->
            <td>{{user.gender===1? "男":"女"}}</td>
         </tr>
      </table>
   </div>
</body>
<script src="../node_modules/vue/dist/vue.js"></script>
<script>

   let app = new Vue({
      el: "#app", 
      data: {
         userList: [
            { id: 1, name: 'jacky', gender: 1 },
            { id: 2, name: 'peter', gender: 0 }
         ]
      }
   });
</script>
```

> 过滤器常用来处理文本格式化的操作。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式。

#### 局部过滤器
注册在当前 Vue 实例中，只有当前实例能用。

```html
<body>
   <div id="app">
      <!-- | 管道符号：表示使用后面的过滤器处理前面的数据 -->
      <td>{{user.gender | genderFilter}}</td>
   </div>
</body>
<script src="../node_modules/vue/dist/vue.js"></script>
<script>
   let app = new Vue({
      el: "#app", data: {
         userList: [
            { id: 1, name: 'jacky', gender: 1 },
            { id: 2, name: 'peter', gender: 0 }
         ]
      },
      // filters 定义局部过滤器，只可以在当前 vue 实例中使用
      filters: {
         genderFilter(gender) {
            return gender === 1 ? '男~' : '女~'
         }
      }
   });
</script>
```

#### 全局过滤器
任何 vue 实例都可以使用。

```html
<script>
   // 在创建 Vue 实例之前全局定义过滤器：
   Vue.filter('capitalize', function (value) {
      return value.charAt(0).toUpperCase() + value.slice(1)
   })
</script>
```