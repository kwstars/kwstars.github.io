---
title: "Contexts and structs"
date: 2021-07-11T21:14:41+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---



此文为 [Contexts and structs](https://blog.golang.org/context-and-structs) 译文



## 介绍

在许多Go API中，尤其是现代的API，函数和方法的第一个参数往往是`context.Context`。Context 提供了一种跨API边界和进程之间传输**超时**、**取消**和**传递元数据信息**。当一个库与远程服务器（如数据库、API等）直接或转接地进行交互时，它经常被使用。



[context的文档](https://golang.org/pkg/context/)指出。

> 上下文不应该存储在一个结构类型内，而应该传递给需要它的每个函数。



本文对这一建议进行了阐述，并举例说明了为什么传递Context而不是将其存储在其他类型中很重要。它还强调了一种罕见的情况，即在结构类型中存储Context可能是合理的，以及如何安全地这样做。



## 作为参数传递的上下文

为了理解不在结构中存储上下文的建议，让我们首先考虑下`Context`作为参数的形式。

```go
type Worker struct { /* … */ }

type Work struct { /* … */ }

func New() *Worker {
  return &Worker{}
}

func (w *Worker) Fetch(ctx context.Context) (*Work, error) {
  _ = ctx // A per-call ctx is used for cancellation, deadlines, and metadata.
}

func (w *Worker) Process(ctx context.Context, work *Work) error {
  _ = ctx // A per-call ctx is used for cancellation, deadlines, and metadata.
}
```



在这里，`(*Worker).Fetch`和`(*Worker).Process`方法都直接接受一个context。通过这种传递即参数的设计，用户可以设置每个调用的截止日期、取消和元数据。而且，传递给每个方法的`context.Context`将被如何使用是很清楚的：不存在传递给一个方法的`context.Context`会被其他方法使用。这是因为上下文的范围是需要多小的操作就有多小的范围，这大大增加了这个包中context的效用和清晰度。



## 在struct中存储context会导致混乱

让我们再来看看上面的Worker例子，采用不受欢迎的context-in-struct方法。它的问题在于，当你将上下文存储在一个结构中时，你会对调用者掩盖其一生，或者更糟糕的是以不可预测的方式将两个作用域混合在一起。

```go
type Worker struct {
  ctx context.Context
}

func New(ctx context.Context) *Worker {
  return &Worker{ctx: ctx}
}

func (w *Worker) Fetch() (*Work, error) {
  _ = w.ctx // A shared w.ctx is used for cancellation, deadlines, and metadata.
}

func (w *Worker) Process(work *Work) error {
  _ = w.ctx // A shared w.ctx is used for cancellation, deadlines, and metadata.
}
```



`(*Worker).Fetch`和`(*Worker).Process`方法都使用一个存储在Worker中的context。这防止了Fetch和Process的调用者（它们本身可能有不同的context）在每个调用的基础上指定一个截止日期、请求取消和附加元数据。例如：用户无法仅仅为(*Worker).Fetch提供一个截止日期，或者仅仅取消(*Worker).Process的调用。调用者的生命周期与共享的上下文交织在一起，而context的范围是创建Worker的生命周期。



与作为参数传递的方法相比，API 也让用户更加困惑。用户可能会问自己：

- 由 New 构造 context.Context，构造函数正在执行时是否需要取消或截止日期的工作？
- 传递给 New 的 context.Context 是否适用于 (*Worker).Fetch 和 (*Worker).Process 中的工作？两者都没？有一个，但没有另一个？



API需要大量的文档来明确地告诉用户context.Context的具体用途。用户可能还需要阅读代码，而不是依靠API所传达的结构。



而且，最后，设计一个生产级的服务器可能是相当危险的，因为它的请求并不是每个都有一个context，因此不能充分地履行取消。如果没有设置每个请求的最后期限的能力，你的[进程可能会积压并耗尽它的资源](https://sre.google/sre-book/handling-overload/)（如内存）！所以，在设计时要考虑到这一点。



## 规则的例外：保留向后兼容性

当Go 1.7--[引入context.Context](https://golang.org/doc/go1.7)--发布时，大量的API必须以向后兼容的方式增加对context的支持。例如，[net/http的客户端方法](https://golang.org/pkg/net/http/)，如Get和Do，是context的最佳候选者。用这些方法发送的每个外部请求都会从 context.Context 所带来的最后期限、取消和元数据支持中受益。



有两种方法可以以向后兼容的方式添加对 context.Context 的支持：在一个结构中包含一个 context，正如我们稍后看到的，以及复制函数，复制的函数接受 context.Context 并将 Context 作为其函数名称的后缀。复制的方法应该比结构中的上下文更受欢迎，在《[Keeping Your Modules Compatible](https://blog.golang.org/module-compatibility)》中会进一步讨论。然而，在某些情况下这是不切实际的：例如，如果你的API暴露了大量的函数，那么将它们全部重复可能是不可行的。



net/http 包选择了 context-in-struct 的方法，它提供了一个有用的案例研究。让我们看一下 net/http 的 Do。在引入 context.Context 之前，Do 的定义如下。

```go
func (c *Client) Do(req *Request) (*Response, error)
```

在 Go 1.7 之后，Do 可能看起来像下面这样，如果不是因为它会破坏向后兼容性：

```go
func (c *Client) Do(ctx context.Context, req *Request) (*Response, error)
```

但是，保持向后的兼容性和遵守Go 1的兼容性承诺对标准库来说是至关重要的。因此，维护者选择在http.Request结构上添加context.Context，以允许支持context.Context而不破坏后向兼容性。

```go
type Request struct {
  ctx context.Context

  // ...
}

func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error) {
  // Simplified for brevity of this article.
  return &Request{
    ctx: ctx,
    // ...
  }
}

func (c *Client) Do(req *Request) (*Response, error)
```



当改造你的API以支持上下文时，将context.Context添加到一个结构中可能是有意义的，如上所述。然而，记得首先考虑重复你的函数，这样可以在不牺牲实用性和理解力的情况下，以向后兼容的方式改造context.Context。比如说。

```go
func (c *Client) Call() error {
  return c.CallContext(context.Background())
}

func (c *Client) CallContext(ctx context.Context) error {
  // ...
}
```



## 结论

上下文使得重要的跨库和跨API的信息很容易在调用栈中传播。但是，为了保持可理解性、易于调试和有效，它的使用必须一致和明确。



当作为方法的第一个参数传递而不是存储在结构类型中时，用户可以充分利用其扩展性，以便通过调用堆栈建立一个强大的取消、截止日期和元数据信息树。而且，最重要的是，当它被作为参数传入时，它的范围被清楚地理解，从而导致堆栈上下的清晰理解和可调试性。



在设计带有上下文的API时，请记住以下建议：将context.Context作为一个参数传入；不要将其存储在结构中。
