---
title: 分支管理最佳实践与团队项目版本管理方案
date: 2017-04-26 14:38
categories: code
tags: Git branch
---

## Git 分支管理最佳实践

### 单主干

- 所有成员都在 master 分支上进行开发
- 使用 tag 或发布分支进行发布
- 在 master 分支中修复 bug，再 cheery-pick解释 到发布分支
- 适用于小项目
- 因为在同一分支上进行开发，协作人员之间必须有良好的交流
- 省去另开分支的时间，不用频繁切换分支

{% asset_img git-branch-tbd.png %}

### Github Flow

- 只包含两类分支：master 和修改分支，master 作为部署分支
- 无论是 feature，bug 还是 hotfix 都从 master  另开分支
- 从分支到 master 的合并需要提交 pull  request
- 在 pull request 上需要进行代码审查和测试，通过后再合并
- Release 在 master 上通过 tag 进行标记
- 对自动化测试、持续集成等相关基础设施要求较高，新分支的测试和部署人工操作较为繁杂

{% asset_img git-branch-github-flow.png %}

### Git Flow
- 同样以 master 作为部署分支，其上所有 commit 都应围绕版本而生
- 共有五类分支 master，develop，feature，release 和 hotfix
- Release 分支测试的问题直接在 release 分支上修改，测试完毕后再分别合并到 master 和 develop，相当于 develop 的需求冻结，可以让新功能开发和版本测试同步进行
- 合并时多数情况不使用 fast-forward 模式
- Hotfix 分支用于修复线上问题，向 master 和 develop 合并
- Feature，release，hotfix 分支合并完成后应被删除
- 操作较为繁琐，有对应的命令行工具
- 较为适合于长期大型项目

| 分支类型    | 命名规范      | 创建自     | 合并至              | 说明                 |
| ------- | --------- | ------- | ---------------- | ------------------ |
| master  | master    | -       | -                | 部署版本分支             |
| develop | develop   | -       | -                | 代码集成分支             |
| feature | feature/* | develop | develop          | 新功能                |
| release | release/* | develop | develop 和 master | 一次新版本的发布           |
| hotfix  | hotfix/*  | master  | develop 和 master | 生产环境中发现的紧急 bug 的修复 |

{% asset_img git-branch-git-flow.png %}

## 团队项目版本管理方案

> **核心问题**
>
> - 如何进行分支管理，怎样合并
> - 是否要进行代码审阅、评论
> - 是否以及如何进行测试和部署
> - 围绕人为核心还是功能为核心

### 单人小项目

#### 关注点

- 单人完成项目，无需他人审查
- 小功能快速迭代，无需额外的 feature 分支
- 主要目标为保存代码进度及回退
- 需要注意对接交递项目时的问题

#### 方案

1. 单主干
2. Master + develop 混合
  - Develop 分支进行开发、修复
  - Master 分支进行版本发布、部署

### 多人协作项目

#### 关注点

- 需要一定的代码审查
- 需要保证沟通的效率
- 分支与对应功能绑定
- 潜在的代码审核

#### 方案

1. Github Flow
2. Git Flow 简化
  - 单一 develop 分支
  - 可选的 hotfix 分支

### 组内共享项目

#### 关注点

- 以分享知识和共同进步为核心
- 代码以人为本
- 需要 merge request，共同评论，学习他人优秀代码以及提出改进意见

#### 方案

- Master + develop 主开发分支
- 从 develop 分支分出 [username]/dev 个人开发分支
- 个人开发分支上可使用任意的模式
- 从 [username]/dev 禁止合并到 develop 
- 需要合并及需要从 develop 分支拉去时从 /dev 分支分出 [username]/merge/[mm-dd] 分支并提交 merge request，冻结个人代码版本

## 参考链接

[A successful Git branching](http://nvie.com/posts/a-successful-git-branching-model/)

[Git 分支管理最佳实践](https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/)

[实用 GIT 工作流](http://yedingding.com/2013/09/11/practical-git-flow-for-startups.html)

[Git 在小团队中的管理流程](http://www.cnblogs.com/tangyikejun/p/4217561.html)

[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

