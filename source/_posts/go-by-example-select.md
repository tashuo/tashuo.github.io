---
title: Go by Example - Select
date: 2017-08-24 10:03:07
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

## <center>Go by Example: Select</center>

`select`允许程序等待多个管道操作，使用`select`来组合`goroutine`和`channel`是golang的一项重要的功能。

```golang
    package main

    import "time"
    import "fmt"
    
    func main() {
        // 定义两个管道用来测试`select`
        c1 := make(chan string)
        c2 := make(chan string)

        // 每个管道变量在独立的goroutine中隔若干时间后接收到一个值，用来模拟阻塞的RPC调用
        go func() {
            time.Sleep(time.Second * 1)
            c1 <- "one"
        }()
        go func() {
            time.Sleep(time.Second * 2)
            c2 <- "two"
        }()
        
        // 使用`select`同时等待这些管道变量，并且当管道有值时就将其打印出来
        for i := 0; i < 2; i++ {
            select {
                case msg1 <- c1:
                    fmt.Println("received", msg1)

                case msg2 <- c2:
                    fmt.Println("received", msg2)
            }
        }
    }

```

```bash
    $ time go run select.go 
    received one
    received two

    //  由于两个睡眠操作是并行执行的，所以执行时间并没有超过3秒
    real 0m2.245s
```


原文链接：[Go by Example: Select](https://gobyexample.com/select)






