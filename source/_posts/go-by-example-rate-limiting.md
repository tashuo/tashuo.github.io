---
title: Go by Example - Rate Limiting
date: 2017-11-24 15:28:43
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

## <center>Go by Example: Rate Limiting</center>
限速是控制资源利用率和维持服务质量的一种重要机制。golang通过`goroutines`、`channels`和`tickers`优雅的支持限速。

```golang
    package main

    import "fmt"
    import "time"
    
    func main() {
        // 首先来看下基本的限速功能
        // 假设我们想限制对请求的处理
        // 我们将从同一个管道中提供这样的请求
        requests := make(chan int,  5)
        for i := 1; i <= 5; i++ {
            requests <- i
        }
        close(requests)
    
        // 该管道每200毫秒接收一个值
        // 这个是我们限速的核心
        limiter := time.Tick(time.Millisecond * 200)
    
        // 通过处理每个请求时管道接收值的阻塞，我们限制每200毫秒只处理一个请求
        for req := range requests {
            <-limiter
            fmt.Println("request", req, time.Now())
        }
    
        // 某些时候可能希望在保持整体限速规则下允许少量的请求不受限制
        // 可以通过含有缓冲区的管道来实现
        // burstyLimiter管道可以使同时处理的请求达到3个
        burstyLimiter := make(chan time.Time, 3)
        for i := 0; i < 3; i++ {
            burstyLimiter <- time.Now()
        }
    
        // 每200毫秒发送给burstyLimiter管道一个值
        go func() {
            for t := range time.Tick(time.Millisecond * 200) {
                burstyLimiter <- t
            }
        }()
    
        // 模拟5个新的请求，前三个请求由于burstyLimiter管道缓冲区的处理可以突破200毫秒的限制
        burstyRequests := make(chan int, 5)
        for i := 1; i <= 5; i++ {
            burstyRequests <- i
        }
        close(burstyRequests)
    
        for req := range burstyRequests {
            <-burstyLimiter
            fmt.Println("request", req, time.Now())
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run rate-limiting.go
    request 1 2017-11-24 15:45:04.641607273 +0800 CST
    request 2 2017-11-24 15:45:04.838581623 +0800 CST
    request 3 2017-11-24 15:45:05.040839234 +0800 CST
    request 4 2017-11-24 15:45:05.241695212 +0800 CST
    request 5 2017-11-24 15:45:05.441367122 +0800 CST
    request 1 2017-11-24 15:45:05.441428032 +0800 CST
    request 2 2017-11-24 15:45:05.441435722 +0800 CST
    request 3 2017-11-24 15:45:05.441440602 +0800 CST
    request 4 2017-11-24 15:45:05.641538132 +0800 CST
    request 5 2017-11-24 15:45:05.84206538 +0800 CST
```

前5个请求跟预期一样是每200秒处理一个，后面5个请求中的前3个并没有被200毫秒的规则限制到。

原文链接：[Go by Example: Rate Limiting](https://gobyexample.com/rate-limiting)






