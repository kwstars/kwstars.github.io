---
title: "Go JSON"
date: 2019-12-10T11:44:48+08:00
draft: false
tags: ["JSON"]
categories: ["Go"]
---

### 结构体 field 大小写对序列化的影响
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type ColorGroup struct {s
	ID     int
	Name   string
	Colors []string
}

func main() {
	m := []*ColorGroup{{
		ID:     1,
		Name:   "a",
		Colors: nil,
	}, {
		ID:     2,
		Name:   "a",
		Colors: nil,
	}}

	b, err := json.Marshal(m)
	if err != nil {
		fmt.Println("error:", err)
	}
	os.Stdout.Write(b)
}
```

```
//output
[{"ID":1,"Name":"a","Colors":null},{"ID":2,"Name":"a","Colors":null}]
```



将ColorGroup中的field改成小写

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type ColorGroup struct {
	id     int
	name   string
	colors []string
}

func main() {
	m := []*ColorGroup{{
		id:     1,
		name:   "a",
		colors: nil,
	}, {
		id:     2,
		name:   "a",
		colors: nil,
	}}

	b, err := json.Marshal(m)
	if err != nil {
		fmt.Println("error:", err)
	}
	os.Stdout.Write(b)
}
```

```
//output
[{},{}]
```



### json 标准库对 nil slice 和 空 slice 的处理

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type TestDemo struct {
	Id     int
	Name   string
	HobbyA []string
	HobbyB []string
}

func main() {
	m := TestDemo{
		HobbyB: nil,
	}

	b, err := json.Marshal(m)
	if err != nil {
		fmt.Println("error:", err)
	}
	os.Stdout.Write(b)
}
```

```
//go1.15 output  
{"Id":0,"Name":"","HobbyA":null,"HobbyB":null}
```



