---
title: Go by Example - Structs
date: 2017-07-28 15:39:49
categories:
- golang
- 笔记
tags:
---

翻译自 [https://gobyexample.com/](https://gobyexample.com/)

> Go by Example
> 
> Go is an open source programming language designed for building simple, fast, and reliable software.
> 
> Go by Example is a hands-on introduction to Go using annotated example programs. Check out the first example or browse the full list below.

## <center>Go by Example: Structs</center>

golang中定义结构体为一系列元素的集合，这对于将数据分组记录很有用。

```golang
    package main

    import "fmt"

    // 定义一个结构体，含有name和age两个字段
    type person struct {
        name string
        age int
    }

    func main() {
        // 实例化一个person结构体
        fmt.Println(person{"Bob", 20})

        // 也可以指定字段名赋值进行实例化
        fmt.Println(person{name: "Alice", age: 30})
        
        // 实例化为赋值的元素默认为零值
        fmt.Println(person{name: "Fred"})

        // 打印一个指向结构体的指针
        fmt.Println(&person{name: "Ann", age: 40})

        // 使用.操作符来操作结构体的字段
        s := person{name: "Sean", age: 50}
        fmt.Println(s.name)
        
        // 也可以使用.操作符操作结构体指针的字段，此时指针会自动取消引用转化为结构体本身
        sp := &s
        fmt.Println(sp.age)

        // 结构体的值是可变的
        sp.age = 51
        fmt.Println(sp.age)
    }
```

```bash
    $ go run structs.go
    {Bob 20}
    {Alice 30}
    {Fred 0}
    &{Ann 40}
    Sean
    50
    51
```

原文链接：[Go by Example: Structs](https://gobyexample.com/structs)






