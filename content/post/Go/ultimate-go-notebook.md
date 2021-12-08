---
title: "Ultimate Go Notebook"
date: 2021-12-08T10:46:26+08:00
draft: true
tags: [""]
categories: [""]
---

## Interfaces

接口赋予程序以结构，并鼓励通过组合进行设计。它们使组件之间的分工更加明确，并强制执行。接口的标准化可以设定清晰和一致的期望。解耦意味着减少组件和它们使用的类型之间的依赖关系。这导致了正确性、质量和可维护性。

接口是毫无价值(`Interfaces Are Valueless`)的，它的申明是基于行为而不是状态。



在以下情况下使用接口：

- API 的用户需要提供实现细节。
- API 有多个需要在内部维护的实现。
- 可以更改的 API 部分已经确定并需要解耦。

不要使用接口：

- 为了使用一个接口而使用。
- 概括一个算法。
- 当用户可以声明自己的接口时。
- 如果不清楚接口如何使代码更好。



### Interfaces Are Valueless



### Implementing Interfaces



## Polymorphism

多态意味着一段代码根据它所操作的具体数据改变其行为。这是 BASIC 的发明者 Tom Kurtz 所说的。这是我们将使用的定义。



 ![image-20211208110844674](/Users/kira/Library/Application Support/typora-user-images/image-20211208110844674.png)

接口值的组成

- 第一个字指向称为 iTable 的特殊数据结构。	

- 第二个字指向正在存储的值。在这种情况下，它是文件值的副本，因为值语义在起作用。



iTable 有两个用途： 

- 它描述了存储的值的类型。就我而言，它是一个文件值。
- 它提供指向为存储的值类型设置的方法的具体实现的函数指针。



## Method Set Rules

```go
type notifier interface {
	notify()
}

type user struct {
	name  string
	email string
}

func (u *user) notify() {
	fmt.Printf("Sending User Email To %s<%s>\n", u.name, u.email)
}

func sendNotification(n notifier) {
	n.notify()
}

func main() {
	u := user{"Bill", "bill@email.com"}
	sendNotification(u)  // cannot use u (type user) as type notifier in argument to sendNotification: user does not implement notifier (notify method has pointer receiver)
}
```



These are the rules defined in the specification:

- For any value of type T, only those methods implemented with a value receiver for that type belong to the method set of that value.
- For any address of type T, all methods implemented for that type belong to the method set of that value.

换句话说，当使用地址（指针）时，所有实现的方法都被附加并可供调用。当处理一个值时，只有那些用值接收器实现的方法被附加并且可以被调用。



在上一课有关方法的课程中，我能够针对具体数据块调用方法，而不管接收者声明的数据语义如何。这是因为编译器可以进行调整以进行调用。在这种情况下，一个值被存储在一个接口中并且方法必须存在。无法进行任何调整。

现在的问题变成了：为什么用指针接收器实现的方法不能附加到类型 T 的值上？这里的完整性问题是什么？

一个原因是因为我不能保证 T 类型的每个值都是可寻址的。如果值没有地址，则无法共享。

```go
type duration int

func (d *duration) notify() {
	fmt.Println("Sending Notification in", *d)
}

func main() {
	duration(42).notify()  // cannot call pointer method on duration(42)
	                       // cannot take the address of duration(42)
}
```

在这个例子中，42的值是一个int类型的常数。即使该值被转换为一个持续时间类型的值，它也没有被存储在一个变量内。这意味着该值从未在堆栈或堆中出现。没有一个地址。常量只在编译时存在。



## Slice of Interface

```go
type printer interface {
	print()
}

type canon struct {
	name string
}

func (c canon) print() {
	fmt.Printf("Printer Name: %s\n", c.name)
}

type epson struct {
	name string
}

func (e *epson) print() {
	fmt.Printf("Printer Name: %s\n", e.name)
}

func main() {
	c := canon{"PIXMA TR4520"}
	e := epson{"WorkForce Pro WF-3720"}

	printers := []printer{
		c,
		&e,
	}
	c.name = "PROGRAF PRO-1000"
	e.name = "Home XP-4100"

	for _, p := range printers {
		p.print()
	}
}

// output
Printer Name: PIXMA TR4520
Printer Name: Home XP-4100
```



# Embedding
