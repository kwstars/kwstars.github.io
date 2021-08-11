---
title: "Reflection and Interfaces for AllSeasons"
date: 2021-07-25T15:47:13+08:00
draft: true
tags: ["Go"]
categories: ["Go"]
---

## Type methods

```go
https://golang.org/src/os/file_plan9.go
func (f *File) Close() error {
    if err := f.checkValid("close"); err != nil {
        return err
    }
    return f.file.close()
}
```

The `Close()` function is a type method because there is that `(f *File)` parameter in front of its name and after the `func` keyword.

In object-oriented programming terminology this process can be described as sending a message to an object.



```GO
package main

import "fmt"

type twoInts struct {
	X int64
	Y int64
}

func regularFunction(a, b twoInts) twoInts {
	temp := twoInts{X: a.X + b.X, Y: a.Y + b.Y}
	return temp
}

func (a twoInts) method(b twoInts) twoInts {
	temp := twoInts{X: a.X + b.X, Y: a.Y + b.Y}
	return temp
}

func main() {
	i := twoInts{X: 1, Y: 2}
	j := twoInts{X: -5, Y: -2}
	fmt.Println(regularFunction(i, j))
	fmt.Println(i.method(j))
}
```



## Go interfaces

### About type assertions

Type assertions help you to do tow things

1. The first thing is checking whether an interface value keeps a particular type.

2. Type assertions help with is allowing you to use the concrete value stored in an interface or assign it to a new variable. 

```GO
package main

import "fmt"

func main() {
	var myInt interface{} = 123
	k, ok := myInt.(int)
	if ok {
		fmt.Println("Success:", k)
	}
	v, ok := myInt.(float64)
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("Failed without panicking!")
	}

	i := myInt.(int)
	fmt.Println("No checking:", i)
    //j := myInt.(bool)  //panic: interface {} is int, not bool.
	//fmt.Println(j)
}
```



## Writing your own interfaces

### Using a Go interface

```go
package main

import (
	"fmt"
	"math"
)

type Shape interface {
	Area() float64
	Perimeter() float64
}

type square struct {
	X float64
}

type circle struct {
	R float64
}

func (s square) Area() float64 {
	return s.X * s.X
}

func (s square) Perimeter() float64 {
	return 4 * s.X
}

func (s circle) Area() float64 {
	return s.R * s.R * math.Pi
}

func (s circle) Perimeter() float64 {
	return 2 * s.R * math.Pi
}

func Calculate(x Shape) {
	_, ok := x.(circle)
	if ok {
		fmt.Println("Is a circle!")
	}
	v, ok := x.(square)
	if ok {
		fmt.Println("Is a square:", v)
	}
	fmt.Println(x.Area())
	fmt.Println(x.Perimeter())
}

func main() {
	x := square{X: 10}
	fmt.Println("Perimeter:", x.Perimeter())
	Calculate(x)
	y := circle{R: 5}
	Calculate(y)
}
```



### Using switch with interface and data types

```go
package main

import "fmt"

type square struct {
	X float64
}

type circle struct {
	R float64
}

type rectangle struct {
	X float64
	Y float64
}

func tellInterface(x interface{}) {
	switch v := x.(type) {
	case square:
		fmt.Println("This is a square!")
	case circle:
		fmt.Printf("%v is a circle!\n", v)
	case rectangle:
		fmt.Println("This is a rectangle!")
	default:
		fmt.Printf("Unknown type %T!\n", v)
	}
}

func main() {
	x := circle{R: 10}
	tellInterface(x) //{10} is a circle!
	y := rectangle{X: 4, Y: 1}
	tellInterface(y) //This is a rectangle!
	z := square{X: 4}
	tellInterface(z)  //This is a square!
	tellInterface(10) //Unknown type int!
}
```



## Refection

### A simple reflection example

why is reflection necessary and when should you use it?

```go
package main

import (
	"fmt"
	"os"
	"reflect"
)

type a struct {
	X int
	Y float64
	Z string
}

type b struct {
	F int
	G int
	H string
	I float64
}

func main() {
	x := 100
	xRefl := reflect.ValueOf(&x).Elem()
	xType := xRefl.Type()
	fmt.Printf("The type of x is %s.\n", xType) //The type of x is int.

	A := a{100, 200.12, "Struct a"}
	B := b{1, 2, "Struct b", -1.2}
	var r reflect.Value
	arguments := os.Args
	if len(arguments) == 1 {
		r = reflect.ValueOf(&A).Elem()
	} else {
		r = reflect.ValueOf(&B).Elem()
	}

	iType := r.Type()
	fmt.Printf("i Type: %s\n", iType)                             //i Type: main.a
	fmt.Printf("The %d fields of %s are:\n", r.NumField(), iType) //The 3 fields of main.a are:
	for i := 0; i < r.NumField(); i++ {
		fmt.Printf("Field name: %s ", iType.Field(i).Name)
		fmt.Printf("with type: %s ", r.Field(i).Type())
		fmt.Printf("and value %v\n", r.Field(i).Interface())
	}
	/*	Field name: X with type: int and value 100
		Field name: Y with type: float64 and value 200.12
		Field name: Z with type: string and value Struct a*/
}
```



### A more advanced reflection example

```go
package main

import (
	"fmt"
	"os"
	"reflect"
)

type t1 int
type t2 int

type a struct {
	X    int
	Y    float64
	Text string
}

func (a1 a) compareStruct(a2 a) bool {
	r1 := reflect.ValueOf(&a1).Elem()
	r2 := reflect.ValueOf(&a2).Elem()
	for i := 0; i < r1.NumField(); i++ {
		if r1.Field(i).Interface() != r2.Field(i).Interface() {
			return false
		}
	}
	return true
}

func printMethods(i interface{}) {
	r := reflect.ValueOf(i)
	t := r.Type()
	fmt.Printf("Type to examine: %s\n", t)
	for j := 0; j < r.NumMethod(); j++ {
		m := r.Method(j).Type()
		fmt.Println(t.Method(j).Name, "-->", m)
	}
}

func main() {
	x1 := t1(100)
	x2 := t2(100)
	fmt.Printf("The type of x1 is %s\n", reflect.TypeOf(x1)) //The type of x1 is main.t1
	fmt.Printf("The type of x2 is %s\n", reflect.TypeOf(x2)) //The type of x2 is main.t2

	var p struct{}
	r := reflect.New(reflect.ValueOf(&p).Type()).Elem()
	fmt.Printf("The type of r is %s\n", reflect.TypeOf(r)) //The type of r is reflect.Value

	a1 := a{1, 2.1, "A1"}
	a2 := a{1, -2, "A2"}

	if a1.compareStruct(a1) {
		fmt.Println("Equal!") //Equal!
	}

	if !a1.compareStruct(a2) {
		fmt.Println("Not Equal!") //Not Equal!
	}

	var f *os.File
	printMethods(f)
	/*
		Type to examine: *os.File
			Chdir --> func() error
			Chmod --> func(fs.FileMode) error
			Chown --> func(int, int) error
			Close --> func() error
			Fd --> func() uintptr
			Name --> func() string
			Read --> func([]uint8) (int, error)
			ReadAt --> func([]uint8, int64) (int, error)
			ReadDir --> func(int) ([]fs.DirEntry, error)
			ReadFrom --> func(io.Reader) (int64, error)
			Readdir --> func(int) ([]fs.FileInfo, error)
			Readdirnames --> func(int) ([]string, error)
			Seek --> func(int64, int) (int64, error)
			SetDeadline --> func(time.Time) error
			SetReadDeadline --> func(time.Time) error
			SetWriteDeadline --> func(time.Time) error
			Stat --> func() (fs.FileInfo, error)
			Sync --> func() error
			SyscallConn --> func() (syscall.RawConn, error)
			Truncate --> func(int64) error
			Write --> func([]uint8) (int, error)
			WriteAt --> func([]uint8, int64) (int, error)
			WriteString --> func(string) (int, error)
	*/
}
```



### The three disadvantages of reflection

1. The first reason is that extensive use of reflection will make your programs hard to read and maintain. A potential solution to this problem is good documentation, but developers are famous for not having the time to write the required documentation.
2. The second reason is that the Go code that uses reflection will make your programs slower. Additionally, such dynamic code will make it difficult for tools to refactor or analyze your code.
3. The last reason is that reflection errors cannot be caught at build time and are reported at runtime as panics, which means that reflection errors can potentially crash your programs.



### The reflectwalk library

```go
package main

import (
	"fmt"
	"github.com/mitchellh/reflectwalk"
	"reflect"
)

type Values struct{ Extra map[string]string }

type WalkMap struct {
	MapVal reflect.Value
	Keys   map[string]bool
	Values map[string]bool
}

func (t *WalkMap) Map(m reflect.Value) error {
	t.MapVal = m
	return nil
}

func (t *WalkMap) MapElem(m, k, v reflect.Value) error {
	if t.Keys == nil {
		t.Keys = make(map[string]bool)
		t.Values = make(map[string]bool)
	}
	t.Keys[k.Interface().(string)] = true
	t.Values[v.Interface().(string)] = true
	return nil
}

func main() {
	w := new(WalkMap)
	type S struct{ Map map[string]string }
	data := &S{Map: map[string]string{"V1": "v1v", "V2": "v2v", "V3": "v3v", "V4": "v4v"}}
	err := reflectwalk.Walk(data, w)
	if err != nil {
		fmt.Println(err)
		return
	}
	r := w.MapVal
	fmt.Println("MapVal:", r) //MapVal: map[V1:v1v V2:v2v V3:v3v V4:v4v]

	rType := r.Type()
	fmt.Printf("Type of r: %s\n", rType) //Type of r: map[string]string

	for _, key := range r.MapKeys() {
		fmt.Println("key:", key, "value:", r.MapIndex(key))
	}
	/*
		key: V2 value: v2v
		key: V3 value: v3v
		key: V4 value: v4v
		key: V1 value: v1v
	*/
}
```



## Object-oriented programming in Go

The first technique uses methods in order to associate a function with a type, which means that in some ways, the function and the type construct an object. 

In the second technique, you embed a type into a new structure type in order to create a kind of hierarchy.

```go
package main

import "fmt"

type a struct {
	XX int
	YY int
}

type b struct {
	AA string
	XX int
}

type c struct {
	A a
	B b
}

func (A a) A() { fmt.Println("Function A() for A") }
func (B b) A() { fmt.Println("Function A() for B") }

func main() {
	var i c
	i.A.A() //Function A() for A
	i.B.A() //Function A() for B
}
```



```go
package main

import "fmt"

type first struct{}

func (a first) F() { a.shared() }

func (a first) shared() { fmt.Println("This is shared() from first!") }

type second struct {
	first
}

func (a second) shared() { fmt.Println("This is shared() from second!") }

func main() {
	first{}.F()       //This is shared() from first!
	second{}.shared() //This is shared() from second!
	i := second{}
	j := i.first
	j.F()                //This is shared() from first!
	(i.first).F()        //This is shared() from first!
	(second{}.first).F() //This is shared() from first!
}
```

