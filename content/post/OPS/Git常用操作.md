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



### 推送本地的test分支到master

```bash
git push origin test:master
```



### 暂存

```bash
git stash

git stash list 

git stath pop
```



### 删除某个历史记录

```bash
# 找到删除commit的上一条COMMIT_ID
git rebase -i COMMIT_ID

# 将要删除的pick改为drop
```



### 单独删除某个文件的所有历史记录

```bash
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch src/main/resources/config/application-test.yml' --prune-empty --tag-name-filter cat -- --all

git push origin --force --all

git push origin --force --tags
```



### submodule

[How To Add and Update Git Submodules](https://devconnected.com/how-to-add-and-update-git-submodules/)

```bash
# 添加submodule
git submodule add <remote_url> <destination_folder>
git commit -m "Added the submodule to the project."
git push

# push submodule
git submodule update --init --recursive

# 更新 submodule
git submodule update --remote --merge
```

