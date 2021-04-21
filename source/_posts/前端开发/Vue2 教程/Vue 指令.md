---
title: Vue 指令

categories:
- 前端开发
- Vue2 教程

date: 2021-04-06 00:00:04
---
#### v-text、v-html

#### v-bind
`v-bind` 用于给 HTML 标签的属性绑定值。

```html
<body>
   <div id="app">
      <!-- 动态链接 -->
      <a v-bind:href="link">gogogo</a>
      <!-- 动态样式，只有绑定的值为 true 时才显示 -->
      <span v-bind:class="{active: isActive, 'text-danger': hasError}">你好</span>
      <!-- 动态样式，另外一种，直接显示 -->
      <span v-bind:style="{color: mycolor, fontSize: mysize}">你好</span>
   </div>
</body>
 
<script>
   new Vue({
      el: "#app",
      data: {
         link: "http://www.baidu.com",
         isActive: true,
         hasError: false,
         mycolor: red,
         mysize: "16px"
      }
   });
</script>
</html>
```

#### v-model
`v-model` 用于表单项的双向绑定。

```html
<body>
   <div id="app">
      精通的语言：<br>
      <input type="checkbox" v-model="language" value="Java">Java<br>
      <input type="checkbox" v-model="language" value="PHP">PHP<br>
      <input type="checkbox" v-model="language" value="Python">Python<br>
      选中了：{{language.join(",")}}
   </div>
</body>
 
<script>
   new Vue({
      el: "#app",
      data: {
         language: []
      }
   });
</script>
</html>
```

#### v-on
`v-on` 用于绑定事件。

```html
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

在事件处理程序过程中调用 `event.preventDefault()` 或者 `event.stopPropagation()` 是非常常见的需求。尽管我们可以在方法中轻松实现这一点，但更好的方式是：方法只是纯粹的数据逻辑，而不是去处理 DOM 事件的细节。

为了解决这个问题，Vue.js 为 `v-on` 提供了事件修饰符。修饰符是由 `.` 开头的指令后缀来表示的：

1. `.stop` 阻止事件冒泡到父元素；
1. `.prevent` 阻止默认事件的发生；
1. `.capture` 使用事件捕捉模式；
1. `.self` 只有元素自身触发事件才执行。（冒泡或捕捉的都不执行）；
1. `.once`  只执行一次；

```html
<body>
   <div id="app">
      <h1>{{name}} 非常帅，有{{num}}个人为他点赞</h1>
      <input type="text" v-model="num">
      <button v-on:click="num++">
         <!-- 触发事件后，不要传递到上面 -->
         <button v-on:click.stop="cancel">清空</button>
      </button>
      
      <!-- 点击链接后，默认行为是跳转到对应的页面 -->
      <a href="http://www.baidu.com"></a>
      <!-- 点击链接后，阻止跳转事件 -->
      <a href="http://www.baidu.com" v-on:click:prevent></a>
      <!-- 可以链式调用，阻止冒泡传递 -->
      <a href="http://www.baidu.com" v-on:click:prevent.stop="alert(123)"></a>
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

在监听键盘事件时，我们经常需要检查常用的键值。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符：

```html
<!-- 只有在 keycode 是 13 时调用 submit 方法 -->
<input v-on:keyup.13="submit">
```

记住所有的 keycode 比较困难，所以 Vue 为最常用的按键提供了别名：

```html
<!-- 同上 -->
<input v-on:keyup.enter="submit">
```

全部的按键别名：
1. `.enter`
1. `.tab`
1. `.esc`
1. `.space`
1. `.up`
1. `.down`
1. `.left`
1. `.right`
1. `.delete` 捕捉删除和退格键

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器（组合按钮）： 
1. `.ctrl`
1. `.alt`
1. `.shift`

```html
<!-- 同时按下 alt + C 按键时触发 clear 方法 -->
<input v-on:keyup.alt.67="clear">

<!-- Ctrl + 鼠标左键触发 -->
<input v-on:keyup.ctrl.click="clear">
```

如果希望在HTML中直接调用Vue的变量，只需加上 `$data.xxx` 即可。

```html
<el-dialog title="关联分类" :visible.sync="cateRelationDialogVisible" width="30%" v-on:close="$data.catelogPath=[]">
</el-dialog>
```

#### v-for
遍历数据渲染页面是非常常用的需求，Vue 中通过 `v-for` 指令来实现。

1、遍历数组

语法：`v-for="item in items"`：
1. `items`：要遍历的数组，需要在 Vue 的 `data` 中定义好；
1. `item`：迭代得到的当前正在遍历的元素；

```html
<body>
   <div id="app">
      <ul>
         <li v-for="user in users">
            姓名：{{user.name}} 性别：{{user.sex}} 年龄：{{user.age}}<br>
         </li>
      </ul>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         users: [
            {name: "柳岩", gender: "女", age: 21},
            {name: "张三", gender: "男", age: 18},
            {name: "范冰冰", gender: "女", age: 24},
            {name: "刘亦菲", gender: "女", age: 18},
            {name: "古力娜扎", gender: "女", age: 25},
         ]
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406135642.png)

2、数据角标

在遍历的过程中，如果我们需要知道数组角标，可以指定第二个参数： `v-for="(item,index) in items"`：   

1. `items`：要迭代的数组
1. `item`：迭代得到的数组元素别名
1. `index`：迭代到的当前元素索引，从 0 开始。

```html
<body>
   <div id="app">
      <ul>
         <li v-for="(user, index) in users">
            姓名：{{user.name}} 性别：{{user.sex}} 年龄：{{user.age}}<br>
         </li>
      </ul>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         users: [
            {name: "柳岩", gender: "女", age: 21},
            {name: "张三", gender: "男", age: 18},
            {name: "范冰冰", gender: "女", age: 24},
            {name: "刘亦菲", gender: "女", age: 18},
            {name: "古力娜扎", gender: "女", age: 25},
         ]
      }
   });
</script>
</html>
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210406135946.png)

3、遍历对象

v-for 除了可以迭代数组，也可以迭代对象。语法基本类似。

`v-for="value in object"`。

`v-for="(value,key) in object"`。

`v-for="(value,key,index) in object"`。

1. 1 个参数时，得到的是对象的属性值
1. 2 个参数时，第一个是属性值，第二个是属性名
1. 3 个参数时，第三个是索引，从 0 开始

4、key

用来标识每一个元素的唯一特征，这样 Vue 可以使用“就地复用”策略有效的提高渲染的效率。

```html
<ul><li v-for="item in items" :key=”item.id”></li></ul>

<ul><li v-for="(item,index) in items" :key=”index”></li></ul>
```

最佳实践：
1. 如果 `items` 是普通数组，可以使用 `index` 作为每个元素的唯一标识；
1. 如果 `items` 是对象数组，可以使用 `item.id` 作为每个元素的唯一标识；

#### v-if、v-show
`v-if` 顾名思义，条件判断。当得到结果为 `true` 时，所在的元素才会被渲染。

`v-show` 当得到结果为 `true` 时，所在的元素才会被显示。

```html
<body>
   <div id="app">
      <!-- false 时候，此标签不存在 -->
      <h1 v-if="canShow">看到我啦</h1>
      <!-- false 时候，此标签 display:none -->
      <h1 v-show="canShow">看到我啦</h1>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         canShow: false
      }
   });
</script>
</html>
```

2、与 `v-for` 结合

当 `v-if` 和 `v-for` 出现在一起时，`v-for` 优先级更高。也就是说，会先遍历，再判断条件。

修改 `v-for` 中的案例，添加` v-if`：

```html
<ul>
   <!-- 只显示女性 -->
   <li v-for="(user,index) in users" v-if="user.gender == '女'">
      姓名：{{user.name}} 性别：{{user.sex}} 年龄：{{user.age}}<br>
   </li>
</ul>
```

#### v-else、v-else-if
`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。

```html
<body>
   <div id="app">
      <h1 v-if="random >= 0.75">0.75</h1>
      <h1 v-else-if="random >= 0.50">0.50</h1>
      <h1 v-else-if="random >= 0.25">0.25</h1>
      <h1 v-else="random >= 0.75">0.00</h1>
   </div>
</body>
 
<script>
   let m = new Vue({
      el: "#app",
      data: {
         random: 0.67
      }
   });
</script>
</html>
```