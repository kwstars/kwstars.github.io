---
title: "Escape Analysis"
date: 2021-12-27T13:53:03+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---



## 为什么要分配在栈上

```go
package main

import "testing"

type SmallObject struct {
	X int32 // 4B
}

type HugeObject struct {
	X [1000]int32 // 4KB
}

type SuperLargeObject struct {
	X [10 * 1000 * 1000]byte // 10MB
}

var global interface{}

func BenchmarkMallocSmallObjectHeap(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		global = &SmallObject{}
	}
}

func BenchmarkMallocSmallObjectStack(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		local := SmallObject{}
		_ = local
	}
}

func BenchmarkMallocHugeObjectHeap(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		global = &HugeObject{}
	}
}

func BenchmarkMallocHugeObjectStack(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		local := HugeObject{}
		_ = local
	}
}

func BenchmarkMallocSuperLargeObjectHeap(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		global = &SuperLargeObject{}
	}
}

func BenchmarkMallocSuperLargeObjectStack(b *testing.B)  {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		local := SuperLargeObject{}
		_ = local
	}
}
```



```bash
$ go test -cpu 1,2,4,8,16 -bench=. malloc_small_object_test.go 
goos: darwin
goarch: arm64
BenchmarkMallocSmallObjectHeap                  109207452               10.94 ns/op            4 B/op          1 allocs/op
BenchmarkMallocSmallObjectHeap-2                167094769                7.575 ns/op           4 B/op          1 allocs/op
BenchmarkMallocSmallObjectHeap-4                163276366                7.245 ns/op           4 B/op          1 allocs/op
BenchmarkMallocSmallObjectHeap-8                166555658                7.150 ns/op           4 B/op          1 allocs/op
BenchmarkMallocSmallObjectHeap-16               166184260                7.186 ns/op           4 B/op          1 allocs/op
BenchmarkMallocSmallObjectStack                 1000000000               0.3143 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSmallObjectStack-2               1000000000               0.3156 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSmallObjectStack-4               1000000000               0.3150 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSmallObjectStack-8               1000000000               0.3140 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSmallObjectStack-16              1000000000               0.3132 ns/op          0 B/op          0 allocs/op
BenchmarkMallocHugeObjectHeap                    4251728               280.2 ns/op          4096 B/op          1 allocs/op
BenchmarkMallocHugeObjectHeap-2                  3723063               315.2 ns/op          4096 B/op          1 allocs/op
BenchmarkMallocHugeObjectHeap-4                  3727275               330.5 ns/op          4096 B/op          1 allocs/op
BenchmarkMallocHugeObjectHeap-8                  3709653               334.5 ns/op          4096 B/op          1 allocs/op
BenchmarkMallocHugeObjectHeap-16                 3560139               331.5 ns/op          4096 B/op          1 allocs/op
BenchmarkMallocHugeObjectStack                  1000000000               0.3139 ns/op          0 B/op          0 allocs/op
BenchmarkMallocHugeObjectStack-2                1000000000               0.3273 ns/op          0 B/op          0 allocs/op
BenchmarkMallocHugeObjectStack-4                1000000000               0.3129 ns/op          0 B/op          0 allocs/op
BenchmarkMallocHugeObjectStack-8                1000000000               0.3216 ns/op          0 B/op          0 allocs/op
BenchmarkMallocHugeObjectStack-16               1000000000               0.3272 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSuperLargeObjectHeap                 4999            222281 ns/op        10002432 B/op          1 allocs/op
BenchmarkMallocSuperLargeObjectHeap-2               5833            200381 ns/op        10002432 B/op          1 allocs/op
BenchmarkMallocSuperLargeObjectHeap-4               5227            201307 ns/op        10002433 B/op          1 allocs/op
BenchmarkMallocSuperLargeObjectHeap-8               5778            208181 ns/op        10002436 B/op          1 allocs/op
BenchmarkMallocSuperLargeObjectHeap-16              5443            206968 ns/op        10002435 B/op          1 allocs/op
BenchmarkMallocSuperLargeObjectStack            1000000000               0.3131 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSuperLargeObjectStack-2          1000000000               0.3161 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSuperLargeObjectStack-4          1000000000               0.3269 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSuperLargeObjectStack-8          1000000000               0.3190 ns/op          0 B/op          0 allocs/op
BenchmarkMallocSuperLargeObjectStack-16         1000000000               0.3139 ns/op          0 B/op          0 allocs/op
PASS
ok      command-line-arguments  28.745s
```



## 如何查看是否逃逸

通过编译器命令，就可以看到详细的逃逸分析过程。而指令集 `-gcflags` 用于将标识参数传递给 Go 编译器，涉及如下：

-  `-m` 会打印出逃逸分析的优化策略，实际上最多总共可以用 4 个 `-m`，但是信息量较大，一般用 1 个就可以了

- `-l` 会禁用函数内联，在这里禁用掉 inline 能更好的观察逃逸情况，减少干扰

```bash
go run -gcflags="-m -l" main.go
go build -gcflags="-m -l" main.go 
go tool compile -m -l   main.go
```



## 场景分析

### basic concept

```go
package main

var g *int

func main() {
	v := 0 // ecsape to heap
	g = &v
}
```



### closure function

```go
package main

func closure1() {
	var x *int
	func(x1 *int) {
		func(x2 *int) {
			func(x3 *int) {
				y := 1
				x3 = &y
			}(x2)
		}(x1)
	}(x)
	_ = x
}

func closure2() {
	var x *int
	func() {
		func() {
			func() {
				y := 1 // moved to heap: y
				x = &y
			}()
		}()
	}()
	_ = x
}
```



### escape size

```go
package main

type hugeExplicitT struct {
	a [3 * 1000 * 1000]int32 // 12MB 不能超过10MB
}

func main() {
	dcl1 := hugeExplicitT{} // dcl1 escapes to heap: too larg for stack
	dcl2 := make([]int32, 0, 17*1000) // dcl2 escapes to heap: too large for stack  不能超过64K
	_ = dcl1
	_ = dcl2
}
```



### input parameters

```go
package main

func f1(x1 *int) **int { // moved to heap: x1
	return &x1
}

func f2(x2 *int) *int { // leaking param: x2 to result ~r1 level=0
	return x2
}

func f3(x3 *int) int {
	return *x3
}

func main() {
	v1 := 1 // moved to heap: v1
	f1(&v1)

	v2 := 1
	f2(&v2)

	v3 := 1
	f3(&v3)
}
```



### map

```go
package main

func map1() {
	m1 := make(map[int]int)
	k1 := 0
	v1 := 0
	m1[k1] = v1
}

func map2() {
	m2 := make(map[*int]*int)
	k2 := 0 // moved to heap: k2
	v2 := 0 // moved to heap: v2
	m2[&k2] = &v2
}

func map3() {
	m3 := make(map[interface{}]interface{})
	k3 := 0 // moved to heap: k3
	v3 := 0 // moved to heap: v3
	m3[&k3] = &v3
}
```



### return value

```go
package main

func f1() **int {
	t := 0   // moved to heap: t
	x1 := &t // moved to heap: x1
	return &x1
}

func f2() *int {
	t := 0 // moved to heap: t
	x2 := &t
	return x2
}

func f3() int {
	t := 0
	x3 := t
	return x3
}

func f4() map[string]int {
	kv := make(map[string]int) // make(map[string]int) escapes to heap
	return kv
}

func f5() []int {
	s := []int{} // []int{} escapes to heap
	return s
}
```



### slice

```go
package main

func main() {
	const constSize = 10
	var varSize = 10
	a := "aaaaaaa"
	_ = []int32{}
	_ = make([]int32, varSize) // make([]int32, varSize) escapes to heap
	_ = make([]int32, constSize)
	_ = make([]int32, len(a))           // make([]int32, len(a)) escapes to heap
	_ = make([]int32, varSize, varSize) // make([]int32, varSize, varSize) escapes to heap
	_ = make([]int32, varSize, constSize)
	_ = make([]int32, constSize, varSize) // make([]int32, constSize, varSize) escapes to heap
	_ = make([]int32, constSize, constSize)
}
```







---

[Deep Dive into The Escape Analysis in Go | GopherCon TW 2020](https://www.youtube.com/watch?v=CkSv4dp2KMY)
