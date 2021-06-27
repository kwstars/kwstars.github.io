---
title: "dlv"
date: 2020-04-01T15:13:52+08:00
draft: false
tags: ["dlv"]
categories: ["Go"]
---



### 远程调试并使用配置文件

```bash
# 使用Go 1.10或更高版本编译应用程序：
go build -gcflags \"all=-N -l\" github.com/app/demo

# 然后使用以下命令通过Delve运行它：
dlv --listen=:9345 --headless=true --api-version=2 --accept-multiclient exec ./demo -- -config=../config/config.xml
```



---

[delve](https://github.com/go-delve/delve)