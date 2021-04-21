---
title: IDEA 使用指南

categories:
- 开发工具

date: 2021-01-10
---

作为一个从事 Java 开发的程序员，每天离不开编辑器的帮助。还记得刚开始学习 Java 编程的时候，使用 Eclipse 作为日常开发工具。后来工作以后，需要使用 Intellij IDEA，刚开始其实并不想怎么用。毕竟 Eclipse 已经足够强大，可以满足日常开发的需求，何必再花时间再去学习其他工具那。刚开始改变是困难的。但是没办法，公司强制使用，不得不去了解去使用。后来用了一段时间才发现 IDEA 是的真的强大。

真香啊~

下面就来介绍一下本人觉得 IDEA 一些强大的功能。

## 安装软件
#### 配置Tomcat
参考：https://blog.csdn.net/zhoukun1314/article/details/88910242

#### Install plugin from disk
1. Idea Plugin [官网](https://plugins.jetbrains.com/)。
1. 键入你需要下载的插件名称、下载 jar，无需解压。
1. 打开settings，选择 `Install plugin from disk`, 选中压缩包。完成！

#### Install javap
https://blog.csdn.net/weixin_30409927/article/details/102951048

#### 微服务开启 services
Views -> Tool Windows -> Services.

## 配置优化
#### 取消 IDEA 默认打开工程
path : Appearance & Behavior > System Settings
value: Reopen last project on startupaa

#### 关闭参数提示
path : Editor > General > Apperance
value: Show parameter name hint

path : Editor > General > code completion (2019.3)
value: Show parameter name hint

path : Inlay Hints > Java > Parameter hints (2019.3)
value: Show parameter hints for

#### 关闭代码自动提示
https://blog.csdn.net/wldds/article/details/98517106

#### 切换快捷键
path : keymap
value:

#### 自动下载源码
path : Build, Execution, Deployment > Build Tools > Maven > Importing
value: Automatically download

#### 默认不折叠一行
path : Editor > General > Code Folding
value: One-line methods

#### 设置不索引 node_moudles 目录
因为项目中前端是用 vue 写的，用 Idea 打开项目的时候，Updating Indexes 到 node_moudles 目录的时候  会很慢很慢很慢。。。。

鼠标右键 选择 Mark directory as 然后继续选择 Excluded。

#### 代码超出长度限制时不自动换行
path : Editor > Code Sytle > Wrap on typing
value: 200 

## 格式化
#### 等号对齐、参数对齐
path : Ediotr > Java
value: Warpping and Braces > Align when multiline

#### 单行注释与代码对齐
path : Editor > Code Style > Java
value: Code Generation > Comment Code > enable add a space at comment start

### 注释不格式化
path : Code Style > Java > Java Doc
value: disable Enable JavaDoc Formatting

## 常见问题
#### 快捷键失灵
现象：黑苹果上，有时间切换app回来发现Idea的快捷键/键盘失灵。
原因：未知
解决：

#### 提示找不到符号，但是类存在
现象：类存在，Idea编辑器没有提示报错，但是编译时提示找不到符号
原因：未知
解决：https://blog.csdn.net/weixin_43789011/article/details/86620573

#### Cannot resolve symbol
IDEA 无法识别同一个 package 里的其他类，将其显示为红色，但是 compile 没有问题。鼠标放上去后显示 “Cannot resolve symbol XXX”，重启 IDEA ，重新 sync gradle，Clean build 都没有用。

多半是因为 IDEA 之前发生了错误，某些 setting 出了问题。

解决方法：点击菜单中的 “File” -> “Invalidate Caches / Restart”，然后点击对话框中的 “Invalidate and Restart”，清空 cache 并且重启。语法就会正确的高亮了。

#### TextMate Bundles for JavaScript
try again -> remove .js

#### Illuminated Cloud is invalid
plugins -> disable Illuminated Cloud is invalid

#### 导入包报错：Cannot resolve com.xxx
我这里的问题是，本地Maven的settings.xml配置了远程仓库，项目里也设置了另外一个远程仓库。下载依赖包找到本地的远程仓库去了。

把本地的settting文件删掉即可。

#### 中英文字体无法等宽
参考：https://www.v2ex.com/t/187956