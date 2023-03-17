---
title: Go by Example - Timeouts
date: 2017-08-24 11:05:22
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

## <center>Go by Example: Timeouts</center>

超时对于需要联接外部资源或需要限定执行时间的程序非常重要，在golang中因为存在`channel`和`select`使得超时实现起来非常简单。

```golang
    package main

    import "time"
    import "fmt"

    func main() {
        // 通过管道c1模拟一个2s后才有返回值的外部请求
        c1 := make(chan string, 1)
        go func() {
            time.Sleep(time.Second * 2)
            c1 <- "result 1"
        }()

        // 使用select执行一条超时操作
        // res等待管道c1的值，<-time.After(time.Second * 1)等待一个一秒后发送的值。
        // 由于select会执行第一个就绪的条件，所以如果前者执行时间超过后者代码中规定的一秒便会进入后者超时的情况
        select {
            case res := <-c1:
                fmt.Println(res)
            case <-time.After(time.Second * 1):
                fmt.Println("timeout1")
        }

        // 由于res会在两秒后就绪，小于超时的三秒限制，所以不会超时
        c2 := make(chan string)
        go func() {
            time.Sleep(time.Second * 2)
            c2 <- "result 2"
        }()
        select {
            case res := <-c2:
                fmt.Println(res)
            case <-time.After(time.Second * 3):
                fmt.Println("timeout 2")
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run timeout.go
    timeout1
    result 2
```

使用select结构的超时模式需要借助管道进行通信，这通常是个好主意因为其他一些golang重要的功能也是基于管道和select。

原文链接：[Go by Example: Timeouts](https://gobyexample.com/timeouts)








