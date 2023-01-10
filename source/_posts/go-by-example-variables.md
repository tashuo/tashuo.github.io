---
title: Go by Example - Variables
date: 2017-07-10 15:45:23
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

## <center>Go by Example: Variables</center>

golang中的变量被显式声明，编译时会做类型检查。

```golang
    package main

    import fmt

    func main() {
        // 使用var关键字声明一个变量并赋值，变量名在前，类型在后
        var a string = "initial"
        fmt.Println(a)

        // 可以一次性初始化多个变量并赋值
        var b,c int = 1, 2
        fmt.Println(b, c)

        // golang会自动推断出未显式声明类型的变量
        var d = true
        fmt.Println(d)

        // 初始化未赋值的变量，golang默认会赋值其相应类型的零值
        var e int
        fmt.Println(e)

        // 也可以使用':='操作符
        f := "short"
        fmt.Println(f)
    }
```

```bash
    $ go run variables.go
    initial
    1 2
    true
    0
    short
```


原文链接：[Go by Example: Variables](https://gobyexample.com/variables)




