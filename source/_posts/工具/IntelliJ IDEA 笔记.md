---
title: IntelliJ IDEA 笔记

categories:
- 工具

date: 2017-10-25
---

IntelliJ在业界被公认为最好的Java开发平台之一，在智能代码助手、代码自动提示、重构、J2EE支持、Ant、JUnit、CVS整合、代码审查、 创新的GUI设计等方面表现突出,并支持基于Android平台的程序开发。


## Inspections 检查设置

- Typo 拼写检查
 

## 常用快捷键

- `Alt + Enter` 导入包，自动修正
- `Ctrl + Alt + L` 格式化代码
- `Ctrl + Alt + O` 优化导入的类和包
- `Ctrl + N` 查找类
- `Ctrl + F` 查找文本
- `Ctrl + R` 替换文本
- `Ctrl + Shift + N` 查找文件
- `Ctrl + Alt + B` 可以跳转到抽象方法的实现
- `Ctrl + Alt + left/right` 返回至上次浏览的位置
- `Alt + left/right` 切换代码视图
- `Ctrl + B` 快速打开光标处的类或方法
- `Alt + F1 > 1` 滚到到源码所在的文件夹

## SVN 快捷键
- `Alt + ~` VCS 操作菜单
- `Ctrl + K` 提交更改
- `Ctrl + T` 更新项目
- `Ctrl + Alt + Shift + D` 显示变化
- `Ctrl + Alt + Z` 撤销
- `Alt + Shift + L` 显示SVN提交记录

## 常用设置

- 关闭参数提示
    `Settings > Editor > Appearance` unselect `Show parameter name hints`
- 设置粗体
    `Settings > Editor > Color Scheme > General > Text Default Text` 设置Bold。

## 关闭Inspections

- `Access can be private`
    `Java -> Declaration redundancy` unselect `Declaration access can be weaker`
- `Class never used`
    `Settings > Inspections > Declaration redundancy` unselect `Unused Declaration`
- `Field injection is not recommended`
    意思是不推荐属性注入，详见`Alt + Enter`
    `Settings > Inspections > Field injection warning` unselect
- 去除XML SQL背景色
    `Prefernces > Editor > Inspections > SQL > No data sources configure` unselect
    `Prefernces > Editor > Inspections > SQL > SQL dialect detection` unselect
- 去掉“注入语言”的背景色
    `Prefernces > Editor > Colors & Fonts > General > Code > Injected language fragment > Background` 取消勾选
    
## 常见问题

- 出现 `0%classes,0% lines covered`
    `Ctrl + Alt + F6`