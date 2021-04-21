---
title: Vue

categories:
- 其它

date: 2020-09-4
---

## npm run dev 异常
1. TypeError: Cannot read property 'parseComponent' of undefined
    vue、@vue/cli-service、vue-template-compiler 等版本不一致，没有细究。

## 运行期间
1. Uncaught TypeError: Cannot set property 'render' of undefined
    组件里写了script标签，没写 `export default {}`。

1.  Vue 文件 render 函数直接操作html元素报错
    如 `vnodes.push(<span slot='title'>{(title)}</span>)`，[解决方法-方案2](https://www.cnblogs.com/xhliang/p/13150769.html)