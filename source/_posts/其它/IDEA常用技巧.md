---
title: IDEA常用技巧

categories:
- 其它

date: 2020-03-4
---

#### 测试
## 
#### 取消idea默认打开工程
path : Appearance & Behavior > System Settings
value: Reopen last project on startupaa

#### 关闭参数提示
path : Editor > General > Apperance 
value: Show parameter name hint
path : Editor > General > code completion (2019.3)
value: Show parameter name hint
path : Inlay Hints > Java > Parameter hints (2019.3)
value: Show parameter hints for

#### 切换快捷键
path : keymap
value: 

#### 自动下载源码
path : Build, Execution, Deployment > Build Tools > Maven > Importing 
value: Automatically download
	
#### 默认不折叠一行
path : Editor > General > Code Folding 
value: One-line methods
	
	
## 格式化
#### 等号对齐、参数对齐
path : Ediotr > Java
value: Warpping and Braces > Align when multiline
	
#### 单行注释与代码对齐
path : Editor > Code Style > Java 
value: Code Generation > Comment Code > enable add a space at comment start

#### Cannot resolve symbol
IDEA 无法识别同一个 package 里的其他类，将其显示为红色，但是 compile 没有问题。鼠标放上去后显示 “Cannot resolve symbol XXX”，重启 IDEA ，重新 sync gradle，Clean build 都没有用。

多半是因为 IDEA 之前发生了错误，某些 setting 出了问题。解决方法如下：

点击菜单中的 “File” -> “Invalidate Caches / Restart”，然后点击对话框中的 “Invalidate and Restart”，清空 cache 并且重启。语法就会正确的高亮了。


#### TextMate Bundles for JavaScript
try again -> remove .js

#### Illuminated Cloud is invalid
plugins -> disable Illuminated Cloud is invalid

#### 导入包报错：Cannot resolve com.xxx
我这里的问题是，本地Maven的settings.xml配置了远程仓库，项目里也设置了另外一个远程仓库。下载依赖包找到本地的远程仓库去了。

把本地的settting文件删掉即可。

## 配置Tomcat
参考：https://blog.csdn.net/zhoukun1314/article/details/88910242


#### TextMate Bundles for JavaScript
