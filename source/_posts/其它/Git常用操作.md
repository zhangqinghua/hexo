---
title: Git常用操作

categories:
- 其它

date: 2020-02-29
---

## 分支操作
#### 查看本地分支
```shell
zhangqinghua$ git branch
* 3.3.1
  master

```

#### 查看本地和远程分支
```
zhangqinghua$ git branch -a
* 3.3.1
  master
  remotes/origin/3.0
  remotes/origin/3.1

```

#### 从当前分支创建出一个新分支
```shell
zhangqinghua$ git checkout -b 3.3.1
Switched to a new branch '3.3.1'

```

#### 切换本地分支
```shell
zhangqinghua$ git checkout 3.0
M       .DS_Store
Switched to branch '3.0'
```

#### 切换本地分支，如果不存在，则从当前分支创建
```
zhangqinghua$ git checkout -b 3.0
M       .DS_Store
Switched to branch '3.0'
```

#### 切换远程分支
```shell
zhangqinghua$ git checkout -b 3.1.2 origin/3.1.2
```

#### 删除本地分支
```shell
zhangqinghua$ git branch -d 3.0
Deleted branch 3.0 (was a8d45e7).
```


#### 删除本地分支（未合并）
```shell
zhangqinghua$ git branch -d 3.1
error: The branch '3.1' is not fully merged.
If you are sure you want to delete it, run 'git branch -D 3.1'.
```

#### 强制删除分支
```shell
zhangqinghua$ git branch -D 3.0
Deleted branch 3.0 (was a8d45e7).
```

#### 删除远程分支
```shell
zhangqinghua$ git push origin :3.1
To code.aliyun.com:icebartech-java-core/icebartech-core.git
 - [deleted]         3.1
```

## 回滚操作