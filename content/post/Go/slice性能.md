---
title: "Slice Tricks"
date: 2021-01-12T18:26:42+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

## Slice Tricks

[Slice Tricks的图](https://ueokande.github.io/go-slice-tricks/)

### Append

```go
a = append(a, b...)
```



### Copy

```go
b = make([]T, len(a))
copy(b, a)

// 这两种方法往往比上面的方法慢一些，但如果复制后有更多的元素要附加到b上，效率会更高。
b = append([]T(nil), a...)
b = append(a[:0:0], a...)
```



### Cut

```go
a = append(a[:i], a[j:]...)
```



### Delete

```go
a = append(a[:i], a[i+1:]...)
// or
a = a[:i+copy(a[i:], a[i+1:])]
```



**删除时不保留顺序**

```go
a[i] = a[len(a)-1] 
a = a[:len(a)-1]
```



**注意** 如果元素的类型是指针或带有指针字段的结构体，需要进行垃圾回收，那么上述Cut和Delete的实现就有一个潜在的内存泄漏问题：一些带有值的元素仍然被分片a引用，因此无法回收。下面的代码可以解决这个问题。

```go
// Cut
copy(a[i:], a[j:])
for k, n := len(a)-j+i, len(a); k < n; k++ {
	a[k] = nil // or the zero value of T
}
a = a[:len(a)-j+i]

// Delete
copy(a[i:], a[i+1:])
a[len(a)-1] = nil // or the zero value of T
a = a[:len(a)-1]

// Delete without preserving order
a[i] = a[len(a)-1]
a[len(a)-1] = nil
a = a[:len(a)-1]
```



### Expand

```go
a = append(a, make([]T, j)...)
```

### Extend

```go
a = append(a, make([]T, j)...)
```



### Filter (in place)

```go
n := 0
for _, x := range a {
	if keep(x) {
		a[n] = x
		n++
	}
}
a = a[:n]
```



### Insert

```go
a = append(a[:i], append([]T{x}, a[i:]...)...)
```



注意：第二个追加创建了一个新的具有自己底层存储的分片，并将a[i:]中的元素复制到该分片中，然后这些元素被复制回分片a（由第一个追加）。通过使用另一种方法可以避免创建新的分片（从而避免内存垃圾）和第二次复制。

```go
// Insert
s = append(s, 0 /* use the zero value of the element type */)
copy(s[i+1:], s[i:])
s[i] = x
```



### InsertVector

```go
a = append(a[:i], append(b, a[i:]...)...)
```



注意：为了得到最好的效率，最好在不使用append的情况下进行插入，特别是当插入元素的数量是已知的。

```go
// Assume element type is int.
func Insert(s []int, k int, vs ...int) []int {
	if n := len(s) + len(vs); n <= cap(s) {
		s2 := s[:n]
		copy(s2[k+len(vs):], s[k:])
		copy(s2[k:], vs)
		return s2
	}
	s2 := make([]int, len(s) + len(vs))
	copy(s2, s[:k])
	copy(s2[k:], vs)
	copy(s2[k+len(vs):], s[k:])
	return s2
}

a = Insert(a, i, b...)
```



### Push

```go
a = append(a, x)
```



### Pop

```Pop
x, a = a[len(a)-1], a[:len(a)-1]
```



### Push Front/Unshift

```go
a = append([]T{x}, a...)
```



### Pop Front/Shift

```go
x, a = a[0], a[1:]
```



## Additional Tricks

### Filtering without allocating

这一招利用了一个分片与原片共享相同的备份阵列和容量，所以过滤后的分片会重复使用存储。当然，原始内容会被修改。

```go
b := a[:0]
for _, x := range a {
	if f(x) {
		b = append(b, x)
	}
}
```

对于必须进行垃圾收集的元素，可以在之后加入以下代码。

```go
for i := len(b); i < len(a); i++ {
	a[i] = nil // or the zero value of T
}
```



### Reversing

用相同的元素替换一个切片的内容，但顺序相反。

```go
for i := len(a)/2-1; i >= 0; i-- {
	opp := len(a)-1-i
	a[i], a[opp] = a[opp], a[i]
}
```



同样的事情，只是有two indices。

```go
for left, right := 0, len(a)-1; left < right; left, right = left+1, right-1 {
	a[left], a[right] = a[right], a[left]
}
```



### Shuffling

Fisher-Yates算法。

> Since go1.10, this is available at [math/rand.Shuffle](https://godoc.org/math/rand#Shuffle)

```go
for i := len(a) - 1; i > 0; i-- {
    j := rand.Intn(i + 1)
    a[i], a[j] = a[j], a[i]
}
```



### Batching with minimal allocation

如果你想对大的切片进行批处理，这很有用。

```go
actions := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
batchSize := 3
batches := make([][]int, 0, (len(actions) + batchSize - 1) / batchSize)

for batchSize < len(actions) {
    actions, batches = actions[batchSize:], append(batches, actions[0:batchSize:batchSize])
}
batches = append(batches, actions)
```



结果如下：

```go
[[0 1 2] [3 4 5] [6 7 8] [9]]
```



### In-place deduplicate (comparable)

```go
import "sort"

in := []int{3,2,1,4,3,2,1,4,1} // any item can be sorted
sort.Ints(in)
j := 0
for i := 1; i < len(in); i++ {
	if in[j] == in[i] {
		continue
	}
	j++
	// preserve the original data
	// in[i], in[j] = in[j], in[i]
	// only set what is required
	in[j] = in[i]
}
result := in[:j+1]
fmt.Println(result) // [1 2 3 4]
```



### Move to front, or append if not present, in place

```go
// moveToFront moves needle to the front of haystack, in place if possible.
func moveToFront(needle string, haystack []string) []string {
	if len(haystack) == 0 || haystack[0] == needle {
		return haystack
	}
	var prev string
	for i, elem := range haystack {
		switch {
		case i == 0:
			haystack[0] = needle
			prev = elem
		case elem == needle:
			haystack[i] = prev
			return haystack
		default:
			haystack[i] = prev
			prev = elem
		}
	}
	return append(haystack, prev)
}

haystack := []string{"a", "b", "c", "d", "e"} // [a b c d e]
haystack = moveToFront("c", haystack)         // [c a b d e]
haystack = moveToFront("f", haystack)         // [f c a b d e]
```



### Sliding Window

```go
func slidingWindow(size int, input []int) [][]int {
	// returns the input slice as the first element
	if len(input) <= size {
		return [][]int{input}
	}

	// allocate slice at the precise size we need
	r := make([][]int, 0, len(input)-size+1)

	for i, j := 0, size; j <= len(input); i, j = i+1, j+1 {
		r = append(r, input[i:j])
	}

	return r
}
```



## Slice先分配优于append

slice先分配空间和后分配空间的性能差别

```go
package main

const num = 10

func T1() {
	t := make([]int, 0, num)
	for i := 0; i < num; i++ {
		t = append(t, i)
	}
}

func T2() {
	t := make([]int, 0)
	for i := 0; i < num; i++ {
		t = append(t, i)
	}
}
```



```go
package main

import "testing"

func BenchmarkTestT1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		T1()
	}
}

func BenchmarkTestT2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		T2()
	}
}
```



```bash
# go test -bench=.

# num为 10 的时候
goos: linux
goarch: amd64
pkg: demo/test
BenchmarkTestT1-4       151955784                7.81 ns/op
BenchmarkTestT2-4        6553101               172 ns/op
PASS
ok      demo/test       3.304s

# num为 100 的时候
goos: linux
goarch: amd64
pkg: demo/test
BenchmarkTestT1-4       18935092                61.2 ns/op
BenchmarkTestT2-4        2196962               548 ns/op
PASS
ok      demo/test       2.986s
```



