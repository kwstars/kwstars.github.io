---
title: "删除Git中已提交的敏感数据"
date: 2021-07-20T08:31:00+08:00
draft: true
tags: ["Git"]
categories: ["Git"]
---

## 方案选择

GitHub官网删除敏感数据主要是通过`git filter-branch` 和 `BFG`来删除。



GitLab官网减少仓库大小，建议使用`git filter-branch` 。

Rewriting a repository can remove unwanted history to make the repository smaller. We **recommend [`git filter-repo`](https://github.com/newren/git-filter-repo/blob/main/README.md)** over [`git filter-branch`](https://git-scm.com/docs/git-filter-branch) and [BFG](https://rtyley.github.io/bfg-repo-cleaner/).



## git filter-repo







---

[Removing sensitive data from a repository](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)

[Reduce repository size](https://docs.gitlab.com/ee/user/project/repository/reducing_the_repo_size_using_git.html)

