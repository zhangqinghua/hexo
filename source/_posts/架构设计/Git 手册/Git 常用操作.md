---
title: Git 常用操作

categories:
- 架构设计
- Git 手册

date: 2021-05-04
---
## 查看分之
#### 查看本地分支
```bash
zhangqinghua$ git branch   
  master
* test3
```

#### 查看远程分支
```bash
zhangqinghua$ git branch -r
  origin/HEAD -> origin/master
  origin/master
  origin/test2
  origin/test3
```

#### 查看本地和远程分支
```bash
zhangqinghua$ git branch -a
  master
* test3
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/test2
  remotes/origin/test3
```

## 切换分支
#### 创建分支
```bash
# 从当前分支创建
zhangqinghua$ git branch 3.0
```

#### 创建分支并切换
```bash
# 从当前分支创建
zhangqinghua$ git checkout -b 3.1
M       .DS_Store
Switched to a new branch '3.1'
```

#### 切换本地分支
```bash
zhangqinghua$ git checkout 3.0
M       .DS_Store
Switched to branch '3.0'
```

#### 切换远程分支
```bash
zhangqinghua$ git checkout -b 3.1.2 origin/3.1.2
```

## 提交操作


## 更新操作
#### 强制更新，放弃本地修改
该命令直接放弃所有修改代码，并更新到版本库最新版本代码。

```bash
zhangqinghua$ git fetch --all && git reset --hard origin/master && git pull
Fetching origin
HEAD is now at de0aaac Initial commit
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> master
```

> git reset --hard origin/分支代码 一般都是主干。若有多期工程，需自己更改。

#### 本地删除远程已经不存在的分支
待解决。。。。

## 删除操作
#### 删除本地分支
```bash
# 删除分支
zhangqinghua$ git branch -d 3.0
Deleted branch 3.0 (was a8d45e7).

# 删除分支（未合并）
zhangqinghua$ git branch -d 3.1
error: The branch '3.1' is not fully merged.
If you are sure you want to delete it, run 'git branch -D 3.1'.

# 强制删除分支
zhangqinghua$ git branch -D 3.0
Deleted branch 3.0 (was a8d45e7).
```
#### 删除远程分支
```bash
zhangqinghua$ git push origin :main
To github.com:zhangqinghua-tutorials/gitflow.git
 - [deleted]         main
```

## 常见问题
#### 删除远程分支错误：refusing to delete the current branch
场景：删除 Github 远程分支失败，提示：refusing to delete the current branch: refs/heads/main

```bash
zhangqinghua$ git push origin :main  
To github.com:zhangqinghua-tutorials/gitflow.git
 ! [remote rejected] main (refusing to delete the current branch: refs/heads/main).
error: failed to push some refs to 'git@github.com:zhangqinghua-tutorials/gitflow.git'
```

原因：删除命令被远程仓库拒绝，因为远程分支 dev_test 是当前分支。

解决：Setting -> Branchs -> Default branch: change to master.

参考：[Git- [!remote rejected]:refusing to delete the current branch](https://blog.csdn.net/qq_32452623/article/details/76684751)