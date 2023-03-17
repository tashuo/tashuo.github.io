---
title: Go by Example - Tickers
date: 2017-09-06 10:50:18
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

## <center>Go by Example: Tickers</center>

`Timer`是当你想要在未来某个时间只执行一次某个事件时使用，而`ticker`可以用来以一定时间间隔重复执行一个事件。

```golang
    package main

    import "time"
    import "fmt"

    func main() {
        // Ticker跟Timer是类似的机制:使用管道来通信
        // 下面示例中我们使用内置的for-range语句对于每500毫秒接收到一个的值进行迭代处理 
        ticker := time.NewTicker(time.Millisecond * 500)
        go func() {
            for t := range ticker.C {
                fmt.Println("Tick at", t)
            }
        }()

        // 跟Timer一样，Ticker也可以被停止
        // 当Ticker取消时，其相应的管道不会再接收到值
        // 该示例中我们在1600毫秒后停止
        time.Sleep(time.Millisecond * 1600)
        ticker.Stop()
        fmt.Println("Ticker stoped")
    }
```

```bash
    tashuo:golang ta_shuo$ go run ticker.go
    Tick at 2017-09-06 11:47:35.512980179 +0800 CST
    Tick at 2017-09-06 11:47:36.012186259 +0800 CST
    Tick at 2017-09-06 11:47:36.515210336 +0800 CST
    Ticker stoped
```

在执行1600毫秒被停止时Ticker 500毫秒的间隔应该已经重复执行了三次

原文链接：[Go by Example: Tickers](https://gobyexample.com/tickers)






