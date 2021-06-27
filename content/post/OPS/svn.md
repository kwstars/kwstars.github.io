---
title: "SVN"
date: 2018-11-24T18:26:07+08:00
draft: false
tags: ["SVN"]
categories: ["OPS"]
---



### 回退到某个版本

```bash
# 从修订版150（当前）回到修订版140
svn update
svn log -l 5
svn merge -r 150:140 .
svn commit -m "Rolled back to r140"
```

