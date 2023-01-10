---
title: Go by Example - Goroutines
date: 2017-08-03 14:50:07
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

## <center>Go by Example: Goroutines</center>

golang中的goroutine是一个比线程还要轻量的调度单位。

```golang
    package main

    import "fmt"

    func f(form string) {
        for i := 0; i < 3; i++ {
            fmt.Println(from, ":", i)
        }
    }

    func main() {
        // 常用的函数调用方式，同步执行
        f("direct")

        // 使用go关键字来启动goroutine执行函数，跟正在执行的函数调用并行执行
        go f("goroutine")

        // 也可以启动goroutine执行匿名函数
        go func(msg string) {
            fmt.Println(msg)
        }("going")

        // 上面两个go启动的函数调用在独立的goroutine中异步执行
        // 下面Scanln函数在接收到外界一个输入时结束程序
        var input string
        fmt.Scanln(&input)
        fmt.Println("done")
    }
```

```bash
    // 从输出的结果，匿名函数的输出插入在第一个goroutine的输出中，可看出两个函数调用是在独立的goroutine中并行执行
    $ go run goroutines.go
    direct : 0
    direct : 1
    direct : 2
    goroutine : 0
    going
    goroutine : 1
    goroutine : 2
    <enter>
    done
```


原文链接：[Go by Example: Goroutines](https://gobyexample.com/goroutines)







