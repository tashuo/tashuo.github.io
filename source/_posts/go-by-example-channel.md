---
title: Go by Example - Channels
date: 2017-08-03 15:18:11
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

## <center>Go by Example: Channels</center>

channel是golang中连接不同goroutine之间的管道。你可以从一个goroutine往channel中发送数据而从另一个goroutine中获取到这些数据。

```golang
    package main

    import "fmt"

    func main() {
        // 创建一个channel
        messages := make(chan string)

        // 使用'<-'操作符来往管道发送数据
        // 在一个新的goroutine执行
        go func() {
            messages <- "ping"
        }

        // 使用'<-'操作符接收管道中的数据
        // 这里会接收到从上面那个新开的goroutine中发往管道的"ping"
        // 默认情况下对于管道的操作-发送和接收都是阻塞的，直到发送方和接收方准备就绪。
        // 这个特性使得我们在程序结尾可以不用做任何同步操作而去等待接收管道中的数据"ping"。
        msg := <- messages
        fmt.Println(msg)
    }
```


```bash
    $ go run channels.go 
    ping
```

原文链接：[Go by Example: Channels](https://gobyexample.com/channels)



















