---
title: "LittleKnowledge"
date: 2020-11-03T01:45:17+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---



### [CGO_ENABLED动态链接](https://twitter.com/bot_golang/status/1275443004484935681)
![enter image description here](https://pbs.twimg.com/media/EbNIJiiU0AATDBP?format=jpg&name=large)

### [测试跳过](https://twitter.com/bot_golang/status/1279131910434742272)
![enter image description here](https://pbs.twimg.com/media/EcBiO8MU8AIU4uI?format=jpg&name=small)

### [90000000打印为90,000,000](https://twitter.com/bot_golang/status/1283827095299227648)
![enter image description here](https://pbs.twimg.com/media/EdERVjDVcAAuqyT?format=png&name=large)

### [子context超时时间会覆上层的context超时](https://twitter.com/bot_golang/status/1271500532373307393)
![enter image description here](https://pbs.twimg.com/media/EaVFqMRUYAEJShZ?format=jpg&name=large)


### [--work flag打印位置](https://twitter.com/bot_golang/status/1270018337192329216)
![enter image description here](https://pbs.twimg.com/media/EaAAFckUYAMfiTg?format=jpg&name=small)

### [设置log日志格式](https://twitter.com/bot_golang/status/1268219462148186113)
![enter image description here](https://pbs.twimg.com/media/EZmd2icU0AATqsj?format=jpg&name=small)

### [proto不删除字段, 而是保留](https://twitter.com/bot_golang/status/1264177910690136064)
![enter image description here](https://pbs.twimg.com/media/EYtDAGrVAAAzxDm?format=png&name=900x900)

### [gRPC 自定义错误码](https://twitter.com/bot_golang/status/1262805169172955137)
![enter image description here](https://pbs.twimg.com/media/EYZiVS2VAAA4VpZ?format=jpg&name=large)

### [尝试返回close函数, 这样不容易忘记](https://twitter.com/joncalhoun/status/1259134879146663936)
![enter image description here](https://pbs.twimg.com/media/EXlYC_7XkAIKxdf?format=jpg&name=large)



### [Printf前填充](https://twitter.com/bot_golang/status/1329497392253526021)

可以在格式说明符中使用“ 0”标志来填充前导零的数字。

```go
package main

import "fmt"

func main() {
	fmt.Printf("%05d\n", 12)
	fmt.Printf("%05d\n", 123)
	fmt.Printf("%05d\n", 1234)
	fmt.Printf("%05d\n", 12345)
}
```



### [结构体转换](https://twitter.com/bot_golang/status/1334529748752424960)

如果两个结构都具有相同的基础类型，则可以将它们转换为另一个结构。即使它们具有不同的struct标签，此方法也有效。

```go
package main

import "fmt"

type Person struct {
	Name string `json:"fullName"`
}

type Employee struct {
	Name string `json:"firstName"`
}

func main() {
	p := Person{
		Name: "Go",
	}

	var e Employee
	e = Employee(p)
	fmt.Printf("%+v", e)
}

Output
----------
{Name:Go}
```



### PrintStack

“调试”包的“打印堆栈”功能可用于打印调用goroutine的堆栈跟踪。这在调试过程中跟踪函数调用的路径非常有用。

```go
package main

import (
	"runtime/debug"
)

func hello2() {
	debug.PrintStack()
}

func hello1() {
	hello2()
}

func main() {
	hello1()
}

Output
----------
goroutine 1 [running]:
runtime/debug.Stack(0xc000034778, 0xc000066f78, 0x404845)
	/usr/local/go/src/runtime/debug/stack.go:24 +0x9f
runtime/debug.PrintStack()
	/usr/local/go/src/runtime/debug/stack.go:16 +0x25
main.hello2(...)
	/home/kira/workspace/demo/http01/demo1.go:8
main.hello1(...)
	/home/kira/workspace/demo/http01/demo1.go:12
main.main()
	/home/kira/workspace/demo/http01/demo1.go:16 +0x25
```



### [NumGoroutine](https://twitter.com/bot_golang/status/1280923623868395521)

“运行时”程序包的“ NumGoroutine() ”函数返回当前正在运行的goroutine的数量。 此功能可用于调试goroutine泄漏。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func hello() {
	time.Sleep(15 * time.Second)
}

func main() {
	go hello()
	fmt.Println("No. of goroutines:", runtime.NumGoroutine())
}

Output
----------
No. of goroutines: 2
```



### [DisallowUnknownFields](https://twitter.com/bot_golang/status/1318620173503397889)

当输入JSON包含目标结构的字段中不存在的键时，可以使用json解码器的DisallowUnknownFields（）方法返回错误。

```go
package main

import (
	"encoding/json"
	"fmt"
	"strings"
)

type person struct {
	Name string
}

func main() {
	const sample = `{"Name":"Gopher", "Age":50}`

	dec := json.NewDecoder(strings.NewReader(sample))
	dec.DisallowUnknownFields()

	var p person
	err := dec.Decode(&p)
	if err != nil {
		fmt.Println("JSON decode error:", err)
		return
	}
	fmt.Println("Name:", p.Name)
}
----------
JSON decode error: json: unknown field "Age"
```



Go更新

![图像](https://pbs.twimg.com/media/EhgFfxGXsAMNc2H?format=png&name=900x900)



