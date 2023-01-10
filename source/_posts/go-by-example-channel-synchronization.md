---
title: Go by Example - Channel Synchronization
date: 2017-08-03 16:39:56
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

## <center>Go by Example: Channel Synchronization</center>

我们可以使用channel来同步不同goroutine的执行。

```golang
    package main

    import "fmt"
    import "time"

    // 定义一个函数，接收一个channel作为参数，该参数用来通知另一个goroutine该函数已经执行完毕
    func worker(done chan bool) {
        fmt.Println("working...")
        time.Sleep(time.Second)
        fmt.Println("done")

        // 将成功信号发往channel
        done <- true
    }

    func main() {
        done := make(chan bool)
        
        // 启动新的goroutine执行worker函数，并出入channel
        go worker(done)

        // 接收channel中的值
        // 如果移除这一句可能收不到任何输出，因为程序可能在worker函数还未执行已经结束了
        <-done
    }
```

```bash
    $ go run channel-synchronization.go      
    working...
    done
```


原文链接：[Go by Example: Channel Synchronization](https://gobyexample.com/channel-synchronization)












