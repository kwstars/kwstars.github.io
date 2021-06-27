---
title: "Git常用操作"
date: 2020-05-10T16:21:40+08:00
draft: false
tags: ["Git"]
categories: ["Git"]
---



### 子模块

```bash
# 添加git的子模块
git submodule add https://github.com/koirand/pulp.git themes/pulp

# 更新子模块
git submodule update --remote --rebase
```



### 删除分支

```bash
# 查看分支
git branch -a

# 删除远程分支
git push origin --delete BRANCH_NAME

# 删除本地分支
git branch -d BRANCH_NAME
```



### git diff

```bash
# 工作区与暂存区
git diff FILE_NAME

# 暂存区与本地仓库
git diff --cached FINE_NAME

# 本地仓库与远程仓库
git diff BRANCH origin/main
```



### 保持最新的fork
[Keeping a fork up to date](https://gist.github.com/CristinaSolana/1885435)

[fork forced sync](https://gist.github.com/glennblock/1974465)

[Syncing a Fork of a GitHub Repository with Upstream](https://ardalis.com/syncing-a-fork-of-a-github-repository-with-upstream/)