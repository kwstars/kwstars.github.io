---
title: "Go Build Flags"
date: 2021-01-13T15:13:52+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---



## 在构建时设置一个环境变量

一个非常方便的构建标志，允许你在构建时从命令行设置一个变量。看下面的例子。

```go
package main

import (
    "fmt"
)

var who string

func main() {
    fmt.Printf("Hello, %s\n", who)
}
```



使用默认值我们得到以下结果。

```bash
$ go run main.go
Hello, 


```



通过添加`-X`链接器标志，我们可以在编译时改变`who`。这只是一个愚蠢的例子，但这对于在编译时更新二进制文件的版本信息很有用。

```bash
$ go run -ldflags "-X main.who=Kira" main.go         
Hello, Kira
```



## Bonus Content

`symbol table`可以使用`go tool nm`命令查看。只需将可执行文件、对象文件或存档的名称传递给它。让你自己更轻松，只需在输出中寻找你的变量即可。

```bash
$ go tool nm main | grep who
  555bf0 D main.who
```



## Shrinking Your Go Binaries

go的版本为`1.15.6`，有原来的`2M`压缩成了`1.4M`。

我们可以用以下标志省略`symbol table`、调试信息和DWARF表。可以用以下标志省略： `-s -w`

```bash
$ go version
go version go1.15.6 linux/amd64

$ go build -ldflags "-s -w" -o main-s-w 
$ ls -alh
总用量 3.4M
drwxrwxr-x 2 kira kira  116 Jan 13 15:24 .
drwxrwxrwx 7 kira kira  114 Jan 12 12:20 ..
-rwxrwxr-x 1 kira kira 2.0M Jan 13 15:21 main
-rw-rw-r-- 1 kira kira  103 Jan 13 15:17 main.go
-rwxrwxr-x 1 kira kira 1.4M Jan 13 15:24 main-s-w 

```

最好的可执行文件打包器之一是[UPX](https://upx.github.io/)。它使用起来非常简单。我用`-9`运行它，以速度为代价获得最大的压缩。

```bash
$ go build 
$ go build -ldflags "-s -w" -o main-s-w 
$ upx -9 -o main-s-w-upx main-s-w       
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
   1400832 ->    558868   39.90%   linux/amd64   main-s-w-upx                  

Packed 1 file.

$ ls -alh
总用量 3.9M
drwxrwxr-x 2 kira kira   89 Jan 13 15:35 .
drwxrwxrwx 7 kira kira  114 Jan 12 12:20 ..
-rw-rw-r-- 1 kira kira  140 Jan 13 15:34 main.go
-rwxrwxr-x 1 kira kira 2.0M Jan 13 15:35 main
-rwxrwxr-x 1 kira kira 1.4M Jan 13 15:35 main-s-w
-rwxrwxr-x 1 kira kira 546K Jan 13 15:35 main-s-w-upx

```



## Windows Executable Without Console Window

我觉得当我运行一个exe时，打开一个控制台窗口很不美观，而且我不需要看到任何输出。我在任何一个golang网站上都找不到这个文档。我不记得是在哪里找到的了，但知道这一点很好。

只要在你的链接器标志中添加`"-H=windowsgui"`。控制台窗口消失了。欢迎你。

```bash
C:\Users\Luke\go\src\hello>go build -ldflags "-X main.who=Universe -H=windowsgui -s -w"
```



---

[Useful Go Build Flags](https://lukeeckley.com/post/useful-go-build-flags/)

[Command link](https://golang.org/cmd/link/)

[git版本信息注入go程序](https://juejin.cn/post/6844903598342537230)

