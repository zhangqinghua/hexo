---
title: Git 分支操作

categories:
- 架构设计
- Git 手册

date: 2021-05-04
---
## 查看分支
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

## 创建分支
#### 创建本地分支
```bash
# 从当前分支创建
zhangqinghua$ git branch 3.0
```

#### 创建本地分支并切换
```bash
# 从当前分支创建
zhangqinghua$ git checkout -b 3.1
M       .DS_Store
Switched to a new branch '3.1'
```

#### 创建远程分支并提交
```bash
# 如果远程仓库不存在此分支，会创建一个
zhangqinghua$ git push origin 3.0
Total 0 (delta 0), reused 0 (delta 0)
To code.aliyun.com:icebartech-java-core/icebartech-core.git
 * [new branch]      3.0 -> 3.0
```

## 切换分支
#### 切换本地分支
```bash
zhangqinghua$ git checkout 3.0
M       .DS_Store
Switched to branch '3.0'
```

#### 切换远程分支
```bash
# 从远程拉取分支下来
zhangqinghua$ git checkout -b 3.1.2 origin/3.1.2
M       .DS_Store
Switched to branch '3.0'
```

## 合并分支
#### 在当前分支合并
```bash
zhangqinghua$ git merge 3.0
Already up to date.
```

## 推送分支
#### 推送到远程指定分支
```bash
# 如果远程仓库不存在此分支，会创建一个
zhangqinghua$ git push origin 3.0
Total 0 (delta 0), reused 0 (delta 0)
To code.aliyun.com:icebartech-java-core/icebartech-core.git
 * [new branch]      3.0 -> 3.0
```

## 更新分支
#### 更新远程新建的分支
#### 更新远程删除的分支
待解决。。。。

## 删除分支
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

## 关联分支
#### 关联本地和远程分支
```bash
git remote add origin git@github.com:lenve/test.git
```

## 常见问题
#### There is no tracking information for the current branch
场景：拉取代码的时提示：There is no tracking information for the current branch。
原因：这是因为本地分支没有和远程分支建立联系。使用 git branch -vv 可以查询本地分支和远程分支的关联关系。
解决：建立关联关系。

```bash
# 将本地的 v5 分支和远程的 v5 分支建立联系。
zhangqinghua$ git branch --set-upstream-to=origin/v5 v5
```

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