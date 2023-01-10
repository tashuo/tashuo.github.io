---
title: Go by Example - Channel Buffering
date: 2017-08-03 16:26:22
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

## <center>Go by Example: Channel Buffering</center>

默认情况下channel是没有缓冲区的，这意味着它们只会在有一个相应的接收者准备好接收管道数据时才会接受发送者往管道发送数据。有缓冲区的channel可以在没有相应接收者时接受发送者发送特定数量的数值。

```golang
    package main

    import "fmt"

    func main() {
        // 创建一个string类型的，缓冲区长度为2的channel
        messages := make(chan string, 2)

        // 由于创建的channel有缓冲区，所以可以直接往里面发送数据而不需要有接收者。
        messages <- "buffered"
        messages <- "channel"

        // 接收channel中的数据
        fmt.Println(<-messages)
        fmt.Println(<-messages)
    }
```

原文链接：[Go by Example: Channel Buffering](https://gobyexample.com/channel-buffering)


