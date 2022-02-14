---
title: "指针与结构体的一个细节"
date: 2022-02-11T11:14:54+08:00
draft: false
categories: ["编程", "go"]
tags: ["go"]
---

## **结构体实现接口**

一个实例只要实现了一个接口中的全部方法，那么就实现了这个接口。之后，我们就可以利用多态的概念，调用实例实现的接口方法。

接口体实现变量有两种方式：值接收者和指针接收者

我们有一个 `Mover` 接口和一个 `dog` 结构体

```
type Mover interface {
    move()
}
​
type dog struct {}
```

### **值接收者实现接口**

```
func (d dog) move() {
    fmt.Println("狗会动")
}
```

此时实现接口的是`dog`类型：

```
func main() {
    var x Mover
    var wangcai = dog{} // 旺财是dog类型
    x = wangcai         // x可以接收dog类型
    var fugui = &dog{}  // 富贵是*dog类型
    x = fugui           // x可以接收*dog类型
    x.move()
}
```

从上面的代码中我们可以发现，使用值接收者实现接口之后，不管是dog结构体还是结构体指针\*dog类型的变量都可以赋值给该接口变量。因为Go语言中有对指针类型变量求值的语法糖，dog指针`fugui`内部会自动求值`*fugui`。

### **指针接收者实现接口**

同样的代码我们再来测试一下使用指针接收者有什么区别：

```
func (d *dog) move() {
    fmt.Println("狗会动")
}
func main() {
    var x Mover
    var wangcai = dog{} // 旺财是dog类型
    x = wangcai         // error: x不可以接收dog类型
    var fugui = &dog{}  // 富贵是*dog类型
    x = fugui           // x可以接收*dog类型
}
```

此时实现`Mover`接口的是`*dog`类型，所以不能给`x`传入`dog`类型的wangcai，此时x只能存储`*dog`类型的值。
