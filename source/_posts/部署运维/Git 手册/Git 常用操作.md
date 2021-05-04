---
title: Git 常用操作

categories:
- 部署运维
- Git 手册

date: 2021-05-04
---
## 更新操作
#### 强制更新，放弃本地修改
该命令直接放弃所有修改代码，并更新到版本库最新版本代码。

```bash
$ git fetch --all
Fetching origin

$ git reset --hard origin/master
HEAD is now at 648386f add

$ git pull
Already up to date.
```

> git reset --hard origin/分支代码 一般都是主干。若有多期工程，需自己更改。