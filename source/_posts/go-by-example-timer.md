---
title: Go by Example - Timers
date: 2017-09-06 09:46:27
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

## <center>Go by Example: Timers</center>

我们经常希望golang代码在未来某个时间点执行或者以一定的时间间隔重复执行，golang内置的`timer`和`ticker`功能可以轻松实现这两种工作。

```golang
    package main

    import "time"

    func main() {
        // Timer类似于未来一个独立的事件，可以告诉它你要等待多久，它会提供一个管道用来在时间点到时发出通知
        // 该例中会等待2秒
        timer1 := time.NewTimer(time.Second * 2)

        // <- timer1.C会一直阻塞直到时间过期管道发送出一个值
        <- timer1.C
        println("Timer1 expired")

        // 如果只是想等待一段时间，可以使用`time.Sleep`
        // 但使用timer你可以在时间到期前取消，这个也是Sleep做不到的
        timer2 := time.NewTimer(time.Second)
        go func() {
            <-timer2.C
            println("Timer2 expired")
        }()

        // 使用Stop来取消timer
        stop2 := timer2.Stop()
        if stop2 {
            println("Timer2 stopped")
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run timer.go
    Timer1 expired
    Timer2 stopped
```

第一个timer会在2秒后过期，但是第二个timer在过期前被Stop，所以没有过期

原文链接：[Go by Example: Timers](https://gobyexample.com/timers)



