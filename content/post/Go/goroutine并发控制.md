---
title: "Goroutine并发控制"
date: 2021-01-15T20:57:51+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

此文为 [Optimization for Number of goroutines Using Feedback Control](https://speakerdeck.com/monochromegane/optimization-for-number-of-goroutines-using-feedback-control) 译文



## Concurrency and complexity





## Concurrency and Go



```go
func main() {
	var wg sync.WaitGroup
	sem := make(chan struct{}, 3)

	for i := 0; i < 10; i++ {
		wg.Add(1)
		sem <- struct{}{}
		go func() {
			defer wg.DonOptimization for Number of goroutines Using Feedback Controle()
			defer func() { <-sem }()
			task()
		}()
	}
	wg.Wait()
	close(sem)
}
```



```go

```





## Concurrency and application

 ![image-20210116115945965](https://i.imgur.com/mS2E84g.png)

- 为了实现速度和稳定性，并发数的设计是非常重要的。
  - 跑太多的任务容易饥饿。
- 并发数的设计是很难的。
  - 最佳的数量取决于应用程序、环境和负载条件。
  - 程序的调整环境和执行环境是不同的。
- 希望动态确定并发数，并通过检测程序运行的瓶颈来快速控制并发数。



### Goal

- 动态确定最佳并发数，并快速控制。

### Basic idea

1. 增加goroutine的数量，以达到性能目标。
2. 如果达到性能目标，则停止增加goroutine的数量。
3. 试图减少goroutines的数量。

 ![image-20210116120803742](https://i.imgur.com/A8ZJ9JM.png)



### Issues to solve for the realization

1. 性能指标的选择
2. 如何快速、持续地控制

### Performance metrics

- 任务使用的独立资源类型
- 例如 CPU使用率，吞吐量
  - go的调度器通过不断切换任务，将阻塞的任务变成尽可能多的CPU约束



- 指标上限是在运行时计算的
  - 设置目标值
  - 逐步调整到上限值

 ![image-20210116122049742](https://i.imgur.com/SCSEopO.png)

