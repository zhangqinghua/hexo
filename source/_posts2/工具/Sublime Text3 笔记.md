---
title: Sublime Text3 笔记
categories:
- 工具

date: 2017-09-28
---

Sublime Text是一款流行的文本编辑器软件，有点类似于TextMate，跨平台，可运行在Linux，Windows和Mac OS X。也是许多程序员喜欢使用的一款文本编辑器软件。

## 安装插件
- 安装package control组件
	1. `Ctrl + '`调出console
	2. 粘贴以下代码到底部命令行并回车
		```python
		import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')
		```

- 用Package Control安装插件的方法
	1. 按下`Ctrl+Shift+P`调出命令面板
	2. 输入install 调出 Install Package 选项并回车，然后在列表中选中要安装的插件

## 常用插件
- HTML+CSS+JAVASCRIPT+JSON快速格式化
	1. 在Sublime Text中，按下`Ctrl+Shift+P`调出命令面板
	2. 输入install 调出 Install Package 选项并回车
	3. 输入pretty，并在列表中选择HTML-CSS-JS Prettify后回车即可安装
	4. `Ctrl+Shift+H`快捷使用
	5. 该插件需要nodejs的支持