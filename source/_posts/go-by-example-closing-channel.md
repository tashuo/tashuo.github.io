---
title: Go by Example - Closing Channels
date: 2017-08-25 09:38:27
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

## <center>Go by Example: Closing Channels</center>

关闭一个管道表明已经没有值会通过它来发送，这个对于向管道的接收者表达已完成的信号是有用的。

```golang
    package main

    func main() {
        // 在这个示例中我们使用一个管道jobs来分发工作，这些工作从main goroutine发送到另一个工作goroutine中执行
        // 如果没有剩余工作要做就关闭管道jobs
        jobs := make(chan int, 5)
        done := make(chan bool)

        // 这个就是上文提到的工作goroutine
        // 它通过‘j, more := <-jobs’重复取值，如果管道jobs被关闭并且所有值都已经接收，more的值将会是false
        // 我们通过这种方式在已经做完所有工作时通知完成的状态
        go func() {
            for {
                j, more := <-jobs
                if more {
                    println("recieved job", j)
                } else {
                    println("recieved all jobs")
                    done <- true
                    return
                }
            }
        }()

        // 通过管道jobs发送三个任务，然后关闭管道
        for j := 1; j < 3; j++ {
            jobs <- j
            println("sent job", j)
        }

        close(jobs)
        println("sent all jobs")

        // 通过管道同步来等待工作goroutine
        <-done
    }
```

```bash
    tashuo:golang ta_shuo$ go run closing_channel.go
    sent job 1
    sent job 2
    sent all jobs
    recieved job 1
    recieved job 2
    recieved all jobs
```

原文链接：[Go by Example: Closing Channels](https://gobyexample.com/closing-channels)

