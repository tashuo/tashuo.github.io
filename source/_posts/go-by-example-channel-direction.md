---
title: Go by Example - Channel Directions
date: 2017-08-23 17:04:20
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

## <center>Go by Example: Channel Directions</center>

当使用管道作为函数参数时，可以显式声明该管道是作为发送者还是接收者，这种特性增加了程序的类型安全。

```golang
        package main

        import "fmt"

        // 定义ping函数只接受用来发送值的管道参数（即管道是接收者）
        // 如果试图从该管道参数中接收值，编译将会报错
        func ping(pings chan<- string, msg string) {
            pings <- msg
        }

        // pong函数接受一个用来接收值的管道pings(即管道是发送者)和一个用来发送值的管道pongs(即管道是接收者)
        func pong(pings <-chan string, pongs chan<- string) {
            msg := <-pings
            pongs <- msg
        }

        func main() {
            pings := make(chan string, 1)
            pongs := make(chan string, 1)

            // 将消息发送到管道pings 
            ping(pings, "passed message")
            
            // 将管道pings中的值发送至管道pongs中
            pong(pings, pongs)
            fmt.Println(<-pongs)
        }
```

```bash
        $ go run channel-directions.go
        passed message
```


原文链接：[Go by Example: Channel Directions](https://gobyexample.com/channel-directions)

