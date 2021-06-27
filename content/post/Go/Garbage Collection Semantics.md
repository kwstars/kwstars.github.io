---
title: "Garbage Collection Semantics"
date: 2021-06-07T18:15:52+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---



![image-20210607181656649](https://i.imgur.com/py3DDfe.png)

从1.12版本开始，go使用了一个非生成的、并发的、三色的、标记和清扫的收集器

![image-20210607181843760](https://i.imgur.com/3svO0jw.png)



### Mark Setup

![image-20210607181901522](https://i.imgur.com/PesOUe7.png)

当一个垃圾收集开始时，必须执行的第一个活动是打开写屏障。

为了打开写屏障，每一个正在运行的应用程序的程序都必须停止。



![image-20210607182151657](https://i.imgur.com/GJuc8cN.png)

在垃圾收集开始之前，有四个应用程序的程序在运行。



![image-20210607182308723](https://i.imgur.com/NVxfaFO.png)

在1.14中修复，但现在哦我们正在等待



![image-20210607182513609](https://i.imgur.com/zjuFY5h.png)



![image-20210607182610814](https://i.imgur.com/uQkt84E.png)



### Marking

![image-20210607182659609](https://i.imgur.com/hHilN4F.png)

检查堆栈以找到指向堆的根指针。
从这些根指针遍历堆图。
标记堆上仍在使用中的值。

![image-20210607182849712](https://i.imgur.com/SRF6kOc.png)

采集器将25%的可用CPU容量占为己有。



![image-20210607183121824](https://i.imgur.com/i7z0Hox.png)

放慢分配速度，反过来加快收集速度。
采集器的目标是消除对标记辅助的需要。
如果一个收集需要大量的标记辅助，收集器可以提前开始下一个收集。



![image-20210607183512429](https://i.imgur.com/k9RPB4d.png)



### Mark Termination

![image-20210607183616632](https://i.imgur.com/o656mFf.png)

关掉写入障碍。
执行各种清理任务。
计算下一个收集目标。



![image-20210607183859716](https://i.imgur.com/TaSQXA3.png)



![image-20210607183924005](https://i.imgur.com/teJbxL5.png)



## Collector Behavior 

### Sweeping

![image-20210607184128459](https://i.imgur.com/2Un9cZa.png)

扫荡

释放堆内存 

当goroutines试图分配新的堆内存时发生。
扫频的延迟被添加到执行分配的成本中，而不是GC。



![image-20210607184348984](https://i.imgur.com/U55Qtoo.png)



![image-20210607184531768](https://i.imgur.com/kEjiCjX.png)

当新的分配出现时，扫除内存。 

![image-20210607184638061](https://i.imgur.com/cytgI34.png)



### GC Percentage

![image-20210607184752232](https://i.imgur.com/lYsatr9.png)

开始收集

默认情况下，GC百分比被设置为100。
代表在下一个集合开始之前可以分配多少新的堆内存的比例。
用GOGC环境变量设置。



![image-20210607185041217](https://i.imgur.com/zJZfJ7O.png)

免责声明 

这张牌中的堆内存图并不代表使用围棋时的真实情况。
围棋中的堆内存往往是零散的、混乱的，你不可能像图片中表现的那样，有一个干净的分离。
这些图表提供了一种可视化堆内存的方法，准确地反映了这种行为。

![image-20210607185339307](https://i.imgur.com/Fe3VAsL.png)



![image-20210607185407230](https://i.imgur.com/giVLpEv.png)

![image-20210607185449646](https://i.imgur.com/rTrV0SG.png)



![image-20210607185527390](https://i.imgur.com/BVrsFMk.png)



![image-20210607185649581](https://i.imgur.com/ya5c4KG.png)

![image-20210607185746501](https://i.imgur.com/BZwo88g.png)



![image-20210607185817092](https://i.imgur.com/YwpnL8R.png)



![image-20210607190020539](https://i.imgur.com/XBG9Vk5.png)



![image-20210607190055406](https://i.imgur.com/MHe3xfe.png)



![image-20210607190137253](https://i.imgur.com/iYbunYG.png)

![image-20210607190226680](https://i.imgur.com/jxqRGoC.png)

算法
有一个节奏算法，用于确定何时开始收集。
反馈环路，收集有关运行中的应用程序和压力的信息。
压力，决定采集器需要运行的速度。

![image-20210607190440711](https://i.imgur.com/LDBhb9b.png)



![image-20210607190556495](https://i.imgur.com/4mwo5W8.png)



![image-20210607190611469](https://i.imgur.com/LSEMwex.png)



![image-20210607190623548](https://i.imgur.com/pS0y3zT.png)



![image-20210607190638855](https://i.imgur.com/yWenwKE.png)



![image-20210607190655679](https://i.imgur.com/zLbKT8q.png)

![image-20210607190708127](https://i.imgur.com/Gq9WtoF.png)

![image-20210607190722222](https://i.imgur.com/uljq9iw.png)

![image-20210607190732976](https://i.imgur.com/7i7e1UO.png)

![image-20210607190746430](https://i.imgur.com/y7R8UV3.png)
