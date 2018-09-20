---
title: Hexo 笔记
categories: 
- 工具
date: 2017-10-29
---

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 安装

1. Git
1. Node.js
1. hexo
  `$ npm install -g hexo-cli`

## Front-matter
Front-matter 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数 | 描述 | 默认值 |
| :--: | :--: | :--: |
| layout | 布局 ||
| title | 标题 ||
| date | 建立日期 | 文件建立日期 |
| updated | 更新日期 | 文件更新日期 |
| comments | 开启文章的评论功能 | true |
| tags | 标签（不适用于分页） ||
| categories | 分类（不适用于分页） ||
| permalink | 覆盖文章网址 | . |


### 分类和标签

只有文章支持分类和标签，您可以在 Front-matter 中设置。在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 Foo, Bar 不等于 Bar, Foo；而标签没有顺序和层次。

```
categories:
- Diary
tags:
- PS3
- Games
```

#### 分类方法的分歧

如果您有过使用WordPress的经验，就很容易误解Hexo的分类方式。WordPress支持对一篇文章设置多个分类，而且这些分类可以是同级的，也可以是父子分类。但是Hexo不支持指定多个同级分类。下面的指定方法：

```
categories:
- Diary
- Life
```

会使分类`Life`成为`Diary`的子分类，而不是并列分类。因此，有必要为您的文章选择尽可能准确的分类。


### JSON Front-matter

除了 YAML 外，你也可以使用 JSON 来编写 Front-matter，只要将 `---` 代换成 `;;;` 即可。

```
"title": "Hello World",
"date": "2013/7/13 20:46:25"
;;;
```

## ReadMore自行截取

文件`/themes/[主题名]/layout/_partial/article.ejs`其中有一段为：
```ejs
<div class="article-entry" itemprop="articleBody">
  <% if (post.excerpt && index) { %>
    <%- post.excerpt %>
    <% if (theme.excerpt_link) { %>
      <p class="article-more-link">
        <a href="<%- config.root %><%- post.path %>#more"><%= theme.excerpt_link %></a>
      </p>
    <% } %>
  <% } else { %>
  	  <%- post.content %>
  <% } %>
</div>
```

改为：
```ejs
<div class="article-entry" itemprop="articleBody">
  <% if (post.excerpt && index) { %>
    <%- post.excerpt %>
    <% if (theme.excerpt_link) { %>
      <p class="article-more-link">
        <a href="<%- config.root %><%- post.path %>#more"><%= theme.excerpt_link %></a>
      </p>
    <% } %>
  <% } else { %>
  	<% var br = post.content.indexOf('\n') %>
  	<% if(br < 0 || !index) { %>
  	  <%- post.content %>
  	<% } else { %>
  	  <%- post.content.substring(0, br) %>
  	  <% if (theme.excerpt_link) { %>
        <p class="article-more-link">
          <a href="<%- config.root %><%- post.path %>#more"><%= theme.excerpt_link %></a>
        </p>
      <% } %>
  	<% } %>
  <% } %>
</div>
```

## 使用本地图片

`config.yml`文件中的`post_asset_folder`选项设为 `true`。

```yml
post_asset_folder: true
```

## 其它
#### 首页展示指定长度
Hexo 博文在首页展示时，“Read More” 或 “阅读更多” 按钮出现的位置是由作者在写文章的时候设定的。只需在文章正文里合适的位置加上 `<!-- more -->` 此标记之前的正文内容就会成为该文章的简述，显示在首页里。

#### 多级分类
`\cafe\layout\_widget\category.ejs` 将 `list_categories` 设置为10

```ejs
<%- list_categories({show_count: theme.show_count,depth: 10}) %>
```

#### 使用数学公式
使用Cafe主题，然后在主题_config.yml添加以下配置：
```yml
# MathJax Support
# 设置enable属性为true即可开启mathjax
# per_page如果为true表示所有文章都设置支持mathjax，如果为false，则在需要支持mathjax的文章加上mathjax: true
mathjax:
  enable: true
  per_page: false
  cdn: https://cdn.bootcss.com/mathjax/2.7.2/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```

#### cafe行内代码背景
cafe主题行内代码背景较为丑陋: 

![](04.png)

可以通过修改`cafe\source\css\_partial\highlight.styl`文件来改变行内代码块的样式：

![](05.png)

![](06.png)


#### cafe字体大小
`cafe/source/css/_variables.styl`的`font-size`属性，默认为16px，改其它值页面会有缩放效果。

## 常见问题

- `ERROR Deployer not found: github`
  `npm install hexo-deployer-git --save`