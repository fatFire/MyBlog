---
title: "Go的零值与空值问题"
date: 2022-02-09T11:14:54+08:00
draft: false
categories: ["编程", "go"]
tags: ["go"]
---

## 问题背景

在 go 中，对于变量声明，即使没有显示的赋初始值，go 也默认给变量赋值，具体的值取决于不同类型对应的**零值**。**本质是 go 中的变量在创建时即分配内存，并且在内存中分配了对应类型的零值。**

## 零值

### 基本类型

```go
var a int
var b bool
var s string

fmt.Printf("%v", a) // 0
fmt.Printf("%v", b) // false
fmt.Printf("%v", s) // ""
```

### 复合类型

```go
var a [4]int
var b []int
var m map[string]int
var p *int
var f func()
var person Person
var c chan int

type Person struct {
	Name string
	age  int
}

fmt.Printf("%v\n", a) // [0 0 0 0]
fmt.Printf("%v\n", b) // [] 
fmt.Printf("%v\n", m) // map[]
fmt.Printf("%v\n", p) // <nil>
fmt.Printf("%v\n", f) // <nil>
fmt.Printf("%v\n", person) <""0>
fmt.Printf("%v\n", c) // <nil>
```

### 总结

| Type           | Zero Value |
| -------------- | ---------- |
| Integer        | 0          |
| Floating point | 0.0        |
| Boolean        | false      |
| String         | “”         |
| Array: \[4]int | \[0 0 0 0] |
| struct         | {field: }  |
| Pointer        | nil        |
| Interface      | nil        |
| Slice          | nil        |
| Map            | nil        |
| Channel        | nil        |
| Function       | nil        |

## 如何进行空值判断

### 基本类型

```go
var a *int
var b *bool
var s *string

if a == nil {
    fmt.Println("空值判断", "a == 0")
}
if b == nil {
    fmt.Println("空值判断", "b == nil")
}
if s == nil {
    fmt.Println("空值判断", "s == nil")
}
```

### 复合类型

```go
var a *[4]int
var b []int
var m map[string]int
var p *int
var f func()
var person *Person
var c chan int

type Person struct {
	Name string
	age  int
}

if a == nil {
    fmt.Println("空值判断", "b == nil")
}
if b == nil {
    fmt.Println("空值判断", "b == nil")
}
if m == nil {
    fmt.Println("空值判断", "m == nil")
}
if p == nil {
    fmt.Println("空值判断", "p == nil")
}
if f == nil {
    fmt.Println("空值判断", "f == nil")
}
if person == nil {
    fmt.Println("空值判断", "person == nil")
}
if c == nil {
    fmt.Println("空值判断", "c == nil")
}
```

总结：

-   对于值类型，判断空值只能使用值类型的指针变量。因为你无法根据零值判空，因为零值既有可能是程序员赋值，也有可能是初始化赋值。

    -   对于数组，`var a [4]int`。默认创建并初始化，可以直接赋值 `a[0] = 1`。

    -   对于 结构体。它是值类型。默认创建并初始化，内部`field`初始化成对应类型的空值。

-   对于引用类型，判断空值利用 `类型变量 == nil`

    -   对于 slice 和 map，创建时没有初始化，赋值之前必须初始化。

        ```go
        var s []int
        var m map[string]int

        s[0] = 1   // panic
        m["0"] = 1 // panic

        s = make([]int, 4)
        m = make(map[string]int)

        s[0] = 1   
        m["0"] = 1
        ```

## 区别零值和空值

在变量初始化时，go 自动赋予变量零值(0, false, “”)。

我们来看一个例子体会零值和空值的区别。

按需更新，这里指的是接口设计时，规定传来的字段一定是要更新的字段，不需要更新的字段就不要出现。举个例子：

```json
// PUT /score
{
	"id": 1,
	"name": "张三",
	"score": 100,
	"create_time": "2021-12-12"
}
```

通过以上json，我们创建一条数据，这时该如何设计更新接口？

```json
// POST /score
{	
	"id": 1,
	"name": "不对，我叫李四",
	"score": 0,
	"create_time": ""
}
```

如上，假如规定零值字段不更新，只有非零值的字段参与更新。那么，如果用户真的想把`score`字段更新为0怎么办，实际上，零值和空值就产生了歧义。

```json
// POST /score
{
	"id": 1,
	"name": "我叫李四"
}
```

所以为了避免歧义，我们规定不更新的字段禁止出现在json中。同样的，在json转为go中的结构体时，我们也只能通过指针来接收：

```go
type UpdateScore struct{
	Id int 
	Name *string
	Score *int
	CreateTime *string
}
```
