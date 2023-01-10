---
title: Go by Example - Atomic Counters
date: 2017-11-28 20:54:10
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

## <center>Go by Example: Atomic Counters</center>
`golang`管理状态的主要机制是通过`channel`的通信，可以通过之前的`worker pools`的例子看到。还有一些其他的管理状态的方式，这次我们探究一下`sync/atomic`包中的可以被多个goroutine访问的`atomic counters`.

```golang
    package main

    import "fmt"
    import "time"
    import "sync/atomic"
    
    func main() {
        // 使用无符号正整数用来表示计数
        var ops uint64 = 0
    
        // 为了模拟并发更新，我们启动50个goroutine，每个goroutine对计数自增1，大概1毫秒的耗时
        for i := 0; i < 50; i++ {
            go func() {
                for {
                    // 使用AddUint64进行原子性的递增操作，将变量的地址作为参数传给该函数
                    atomic.AddUint64(&ops, 1)
                    
                    // 睡眠1毫秒，模拟耗时
                    time.Sleep(time.Millisecond)
                }
            }()
        }
    
        // 睡眠1秒进行若干累加操作
        time.Sleep(time.Second)
    
        // 由于计数器可能正在被其他goroutine进行累加操作，为了安全的获取计数值，我们通过LoadUint64函数获取计数器的一个拷贝，将计数器的内存地址给该函数
        opsFinal := atomic.LoadUint64(&ops)
        fmt.Println("ops: ", opsFinal)
    }
```

```bash
    tashuo:golang ta_shuo$ go run atomic-counter.go
    ops:  41280
```

运行该程序，通过返回值可得知计数器累加了大概40000多次

原文链接：[Go by Example: Atomic Counters](https://gobyexample.com/atomic-counters)

