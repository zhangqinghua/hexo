---
title: Git 分支策略

categories:
- 架构设计
- Git 手册

date: 2021-06-09
---
Git 的特色之一就是可以灵活的建立分支，因为有分支的存在，才构成了多工作流的特色。事实的确如此，因为项目开发中，多人协作，分支很多，虽然各自在分支上互不干扰，但是我们总归需要把分支合并到一起，而且真实项目中涉及到很多问题，例如版本迭代，版本发布，Bug 修复等，为了更好的管理代码，需要制定一个工作流程，这就是通常意义上的Workflow，也就是我们常说的分支管理策略。

## 常见的分支策略
目前使用度最高的工作流前三名分别是以下三种（排名不分先后）：
1. Git Flow
1. GitHub Flow
1. GitLab Flow

其中 Git Flow 出现的最早，GitHub Flow 在 Git Flow 的基础上，做了一些优化，适用于持续版本的发布，而 GitLab Flow 出现的时间比较晚，所以综合前面两种工作流的优点，制定而成的一个工作流。

#### Git Flow
Git Flow 是 Vincent Driessen 2010 年发布出来的他自己的分支管理模型，属于强流程性，使用度非常高，比较适合开发技术能力中等的团队作战。

Git Flow 的分支结构很特别，按功能来说，可以分支为5种分支，从 5 种分支的生命时间上，又可以分别归类为长期分支和暂时分支，或者更贴切描述为，主要分支和协助分支。

**主要分支**

在采用 Git Flow 工作流的项目中，代码的中央仓库会一直存在以下两个长期分支：
1. Master Branch
1. Develop Branch

其中 origin/master 分支上的最新代码永远是版本发布状态。origin/develop 分支则是最新的开发进度。

当 develop 上的代码达到一个稳定的状态，可以发布版本的时候，develop 上这些修改会以某种特别方式被合并到 master 分支上，然后标记上对应的版本标签。

**协助分支**

除了主要分支，Git Flow 的开发模式还需要一系列的协助分支，来帮助更好的功能的并行开发，简化功能开发和问题修复。协助分支分为以下几类：
1. Hotfix Branch
1. Release Branch
1. Feature Branch

Feature 分支用来做分模块功能开发，建议命名为feature-xxx。模块完成之后，会合并到 develop 分支，然后删除。

Release 分支用来做版本发布的预发布分支，建议命名为 release-xxx。例如在软件 1.0.0 版本的功能全部开发完成，提交测试之后，从 develop 检出 release-1.0.0，测试中出现的小问题，在 release 分支进行修改提交，测试完毕准备发布的时候，代码会合并到 master 和 develop，master 分支合并后会打上对应版本标签 v1.0.0, 合并后删除自己。这样做的好处是，在测试的时候，不影响下一个版本功能并行开发。

Hotfix 分支是用来做线上的紧急 bug 修复的分支,建议命名为 hotfix-xxx。当线上某个版本出现了问题，将检出对应版本的代码，创建 Hotfix 分支，问题修复后，合并回 master 和 develop ，然后删除自己。这里注意，合并到 master 的时候，也要打上修复后的版本标签。

![](https://pic3.zhimg.com/80/v2-d9cac24cd7dced55b83d543a9cc173ca_720w.jpg)

图中画了 Git Flow 的五种分支，master，develop，feature ,release , hoxfixes，其中 master 和 develop 字体被加粗代表主要分支。master 分支每合并一个分支，无论是 hotfix 还是 release ,都会打一个版本标签。通过箭头可以清楚的看到分支的开始和结束走向，例如 feature 分支从 develop 开始，最终合并回 develop ，hoxfixes 从 master 检出创建，最后合并回 develop 和 master，master 也打上了标签。

#### Github Flow
GitHub Flow 是大型程序员交友社区 GitHub 制定并使用的工作流模型，由 scott chacon 在 2011 年 8 月 31 号正式发布。因为 Git Flow 对于大部分开发人员和团队来说，稍微有些复杂，而且没有 GUI 图形页面，只能命令行操作。所以为了更好的解决这些问题，GitHub Flow 应运而生了。

![](https://pic4.zhimg.com/80/v2-4830f2888ab8eed040cbb6556cc32557_720w.jpg)

对比上面那张 Git flow 分支模型图，GitHub flow 真的可以称得上简单明了，因为 GitHub Flow 推荐做法是只有一个主分支 master，团队成员们的分支代码通过 pull Request 来合并到 master 上。这种分支策略适合团队开发技术比较高的团队使用，否则就是 no zuo no die。

GitHub Flow 模型简单说明：
1. 只有一个长期分支 master，而且 master 分支上的代码，永远是可发布状态，一般 master 会设置 protected 分支保护，只有有权限的人才能推送代码到 master 分支。
1. 如果有新功能开发，可以从 master 分支上检出新分支。
1. 在本地分支提交代码，并且保证按时向远程仓库推送。
1. 当你需要反馈或者帮助，或者你想合并分支时，可以发起一个 pull request。
1. 当 review 或者讨论通过后，代码会合并到目标分支。
1. 一旦合并到 master 分支，应该立即发布。

其中最有特色的就是 pull request，后来 GitLab Flow也 受此启发有了 Merge Request。

#### GitLab Flow
GitLab Flow 是 GitLab 的 CEO Sytse Sijbrandij 在 2014 年 9 月 29 正式发布出来的。因为出现的比前面两种工作流稍微晚一些，所以它有个非常大的优势，集百家之长，补百家之短。

GitLab 既支持 Git Flow 的分支策略，也有 GitHub Flow 的 Pull Request（Merge Request） 和 issue tracking。

针对 GitHub 里面只有一个 Master 分支的情况，从需要发布的环境的角度出发，添加了 pre-production 和 prodution 分支都对应不同的环境，这个分支策略比较适用服务端，测试环境，预发环境，正式环境，一个环境建一个分支。

![](https://pic3.zhimg.com/80/v2-1f07b2eed7292e788127ff4fd460b0d6_720w.jpg)

这里要注意，代码合并的顺序，要按环境依次推送，确保代码被充分测试过,才会从上游分支合并到下游分支。除非是很紧急的情况，才允许跳过上游分支，直接合并到下游分支。这种规则称之为 "upstream first"，也就是 “上游优先”。

在 Git Flow ,版本记录是通过 master 上的 tag 来记录。发现问题，创建 hotfix 分支，完成之后合并到 master 和 develop。在 GitLab Flow ，建议的做法是每一个稳定版本，都要从master分支拉出一个分支，比如2-3-stable、2-4-stable等等。发现问题，就从对应版本分支创建修复分支，完成之后，先合并到 master，才能再合并到 release 分支，遵循 “上游优先” 原则。

![](https://pic1.zhimg.com/80/v2-76cb458e70552f68261bb41c7120a788_720w.jpg)

GitLab 中的 Merge Request（MR，合并请求）是作为编码协作及版本控制平台的 GitLab 的基础功能。就和它的命名一样：是一个将一个分支合并到另一个分支上的请求。

通过 GitLab 的 Merge requests，我们可以：
1. 对比两个分支的差异
1. 逐行地去 Review 和讨论改动内容
1. 将 MR 指派给任何已注册用户，并且可以任意多地改变受理人
1. 通过 UI 界面去解决冲突

#### 主干开发
以上三种是常见模式，补充一个目前比较高大上，但很少团队可以达到的模式，主干开发。

主线开发模型：如下图所示，主线开发模型是同一个产品的所有的开发人员共享一个 trunk，开发人员可以有自己的私有分支，但所有修改最后都会回到主干，只有在 Release 时才会创建 Release Branch，由 Release Engineer 进行维护，发布分支是主干某个时点的快照，必要时通过 cherry-pick 从主干挑拣代码到发布分支。基于主干的开发的优点是保证了所有用户看到的都是同一份代码的最新版本，避免了合并分支时的麻烦，缺点是不适于瀑布开发模型，同时对开发人员和发布分支的维护人员的技术要求比较高。

![](https://pic4.zhimg.com/80/v2-86a2c7151462cf37bb68f0e31c0b74c3_720w.jpg)

## Git Flow 流程
#### 01. 创建项目

#### 02. 创建开发分支

#### 03. feature 1：商品管理

#### 04. feature 2：性能优化

#### 05. 合并 feature 1 到 develop 分支

#### 06. 合并 feature 2 到 develop 分支

#### 07. 发布测试版本

#### 08. 修复测试版本 Bug

#### 09. 发布线上版本

#### 10. feature 3：评论管理

#### 11. 修复线上版本 Bug

#### 12. 补充说明
1. 远程仓库只保留 master 和 develop 分支。
1. featute 和 hotfix 分支用完即删

## Github Flow 流程
todo...

## Gitlab Flow 流程
todo...