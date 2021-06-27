---
title: "HTTP3(基于QUIC的HTTP)如何工作？UDP的优点，版本之间的差异(v1.0/v1.1/v2/v3)"
date: 2020-12-13T19:53:26+08:00
draft: false
tags: ["HTTP"]
categories: ["HTTP"]
---

本文翻译自[《How does HTTP/3 (HTTP over QUIC) work?〜Advantages of UDP, differences between versions(v1.0/v1.1/v2/v3)〜》](https://milestone-of-se.nesuke.com/en/l7protocol/http/http3-over-quic/)。全文如下：



HTTP协议为互联网的传播做出了贡献。而现在，其新版本HTTP/3（http版本3）的规范正在进行中。第20版草案已于2019/4/23发布。

在这篇文章中，我们将根据HTTP的历史来看一下HTTP/3的特点。



## HTTP规范结构

在HTTP RFC规范中，有两个主要元素。

1. Syntax
2. Semantics



**"语法 "告诉你如何通过网络向对方有效地传递信息。**

他规定了 Cache 和 Keep-Alive 的规则，以及表示语义学中定义的每个数据的位置的数据包格式。HTTP / 1.1的RFC规范是[RFC 7230](https://tools.ietf.org/html/rfc7230)。



**语义学显示了你告诉对方的信息。**

它规定了请求的方法，如GET、POST、CONNECT，以及响应代码，如200（OK），头字段。有[RFC7231](https://tools.ietf.org/html/rfc7231)作为HTTP/1.1的规范。如果你想查看方法列表和响应代码列表，我认为这个RFC的目录很容易阅读和理解。



## HTTP协议栈的过渡和比较

下面显示了协议栈和功能从HTTP / 1.0到HTTP/3的变化情况。

![image-20210518135037347](https://i.imgur.com/T5pS1NX.png)

# 

## 从HTTP/1.0到HTTP/1.1的改变

HTTP/1.0是在1996年的RFC1945中规定的，当时互联网已经普及，并在试验和错误中反复改进。HTTP/1.1（RFC2068）在第二年的1997年发表，此后又进行了两次修订。最终的版本就是前面提到的RFC7230和RFC7231。



从HTTP/1.0到1.1的巨大变化之一是对 "Keep-Alive "的正式支持。在HTTP/1.0中，每个TCP连接只能发出一个请求。现在，HTTP/1.1允许在一个TCP连接上 "按顺序 "提出多个请求。



![image-20210518135514339](https://i.imgur.com/iOd1Gy9.png)



其他改进包括增加了方法和响应代码，以及用 "Host "头字段支持虚拟域。



## 从HTTP/1.1到HTTP/2的改变

在HTTP /1.1中，有可能在一个TCP连接中 "按顺序 "发送一些请求，但这被认为是一个问题，如果一个请求被堵塞，后面的请求将被相应地延迟。



如上所述，"当单个（慢）对象阻止其他/后续的对象前进时 "的事件一般被称为`HoL（Head of Line）Blocking`。例如: 高速公路上发生了一起车祸, 可能会使整条道路拥堵。



由于这个问题，符合HTTP/1.1的浏览器经常建立多个TCP连接，即使它可以使用Keep-Alive。



为了解决这个问题，HTTP/2现在可以在一个TCP连接中 "并行 "地发出多个请求。这种机制被称为 `"Straem"`。



浏览器将一个流分配给几个请求中的每一个，并将它们一起发送。网络服务器识别这些流，并向每个流发送响应。



![image-20210518140538655](https://i.imgur.com/AZ1FpiJ.png)



HTTP/2语义在HTTP/1.1中没有变化。换句话说，方法、响应代码和头字段的含义都没有改变。在HTTP/2中，stream等syntax部分得到了极大的改善。



关于HTTP/2的其他改进包括在`HTTP层面实施流量控制和优先级控制`，并实施了`PUSH功能`，它允许服务器发送认为必要的文件（如链接的.css或图像文件），而无需客户端的请求。



### HoL Blocking problem, again

然而，即使是这样的HTTP/2，也被发现陷入了队头阻塞。



因为HTTP/2首先是在TCP上处理的，当一个分段的TCP有效载荷在客户端组装时，最终会按照TCP序列号的顺序重新排列，所以`如果缺少一个数据包，就无法处理后续的数据包`。



这就是TCP/IP通信的库的问题。因此，即使流发出多个请求，即使一个数据包丢失后的所有响应都是完整的，但在丢失的数据包被重新发送之前，这些响应也不能被显示。



![image-20210518141611904](https://i.imgur.com/hq1VHN3.png)



换句话说，即使HTTP级别的队头阻塞得到了解决，但`TCP级别的队头阻塞却没有得到解决`。



所以，许多浏览器都实现了建立多个TCP连接来访问一个网络服务器，就像HTTP/1.1中一样。



另一方面，在同一时间，谷歌在测试中发现，"TCP 3way握手或TLS协商都要绕行多次，但这实际上对网络显示速度影响很大。`减少往返时间（RTT）和往返次数，比增加带宽更有效`！"。



谷歌开发了SPDY协议和QUIC协议，反映了这个想法。(我偏离了这个故事的主流，但TLS 1.2到1.3的变化也在遵循这个趋势。)



有了这些背景，就产生`a stream of TCP removal`，并决定`HTTP/3通过UDP（正是QUIC）工作`。



## 从HTTP/2到HTTP/3的改变

HTTP/3已经被决定在一个叫做QUIC的新协议上运行。QUIC通过UDP工作。HTTP/3的语义与HTTP/1.1或HTTP/2没有变化。



因此，HTTP/3的主要变化是语法部分，主要内容是 `"HTTP/2从TCP+TLS过渡到UDP+QUIC以及相应的适应"`。



![image-20210518142057237](https://i.imgur.com/fkldHhd.png)



流和流量控制不再由HTTP/3提供，而是转移到QUIC中。



因此，TCP+TLS拥有的大部分功能可以由UDP+QUIC提供。

- TCP connection -> QUIC Connection-ID
- TCP Sequence Numbers(ack/retransmission) -> QUIC Packet Numbers
- TCP Flow-Control
  (Window) -> QUIC Flow-Control (Window)



由于UDP的原因，预计会出现以下情况。



### 通过取消TCP 3way 握手来提高速度

如上所述，在当今时代，减少通信往返次数比扩大带宽更有效地提高速度。通过取消 TCP 3way 握手，可以减少往返次数。



下面是 "TCP + TLS v1.3 "和 "UDP + QUIC + TLS v1.3 "的比较。



![image-20210518142437682](https://i.imgur.com/psIxELC.png)



上图显示了1-RTT，它使用一次往返来建立QUIC会话，但是当会话恢复时，也有一种叫做0-RTT的方法，它与恢复消息一起发送应用数据（HTTP/3等）。然而，这在某种程度上不太安全，所以由应用程序的组成人员决定是否使用。



### TCP 队头阻断的解决方案

数据是按`QUIC stream unit`（而不是QUIC连接单元）从QUIC传递到应用程序（网络服务器）的。

![image-20210518143035405](https://i.imgur.com/awX6vLg.png)



### 客户端IP在漫游变更

从SIM卡切换到无线（或反之），经常被用作智能电话的使用场景。这时，智能电话的IP会发生变化。在TCP的情况下，你必须重新建立TCP连接。由于TCP连接是由源/目的IP/端口号的组合来识别的）。



然而，`在QUIC使用UDP的情况下，客户端和服务器之间的连接是由[**QUIC Connection ID**]识别的`，因此即使IP改变，连接也会继续。



![image-20210518143256597](https://i.imgur.com/zHCiMER.png)



### 其他特性

HTTP/3也有Prioritization-Control功能。这是HTTP/2增加的功能，被HTTP/3继承了）。

客户端（浏览器等）告诉服务器 `"在并行请求中你应该响应什么优先级"`。

服务器可以用这个请求来回应。然而，这取决于服务器端，他可以在最坏的情况下忽略它。



还有一个流量控制功能。这与TCP流量控制几乎相同，它是通过及时向对方传递可接受的缓冲区数量（滑动窗口）来简化通信的方法。

Flow-Control  在 QUIC connection units 和 QUIC stream units 上使用。



