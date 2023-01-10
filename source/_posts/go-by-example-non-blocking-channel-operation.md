---
title: Go by Example - Non-Blocking Channel Operations
date: 2017-08-24 16:34:03
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

## <center>Go by Example: Non-Blocking Channel Operations</center>

普通管道的发送与接收都是阻塞的，然而我们可以使用select的default条件来实现非阻塞的发送，接收和甚至是非阻塞的多路复用的`selects`。

```golang
    package main

    import "fmt"

    func main() {
        messages := make(chan string)
        signals := make(chan bool)

        // 这里是一个非阻塞的接收者，如果管道msg就绪的话select会使用管道值执行这条分支
        // 否则的话select会执行default分支
        select {
            case msg := <-messages:
                fmt.Println("recieved message", msg)
            default:
                fmt.Println("no message received")
        }

        // 类似的非阻塞的发送者
        msg := "hi"
        select {
            case messages <- msg:
                fmt.Println("sent message", msg)
            default:
                fmt.Println("no message sent")
        }

        // 可以在default分支前使用多个条件分支用于实现一个非阻塞的多路复用select
        // 下面的例子尝试同时非阻塞的接收messages和signals两个管道的值
        select {
            case msg := <-messages:
                fmt.Println("recieved messages", msg)
            case sig := <-signals:
                fmt.Println("recieved signals", sig)
            default:
                fmt.Println("no activity")
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run non-blocking-channel.go
    no message received
    no message sent
    no activity
```

原文链接：[Go by Example: Non-Blocking Channel Operations](https://gobyexample.com/non-blocking-channel-operations)

