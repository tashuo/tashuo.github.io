---
title: Go by Example - Worker Pools
date: 2017-11-24 12:21:09
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

## <center>Go by Example: Worker Pools</center>
这节探索如何使用`goroutine`和`channel`实现工作池的功能。

```golang
    package main

    import "fmt"
    import "time"

    // 这个是工作进程，用来执行并发的任务
    // 每个进程都会从管道jobs接受工作，并且将相应发送至管道results
    // 每个工作我们都会睡眠一秒用以模拟一项耗时的任务
    func worker(id int, jobs <-chan int, results chan<- int) {
        for j := range jobs {
            fmt.Println("worker", id, "started job", j)
            time.Sleep(time.Second)
            fmt.Println("worker", id, "finished job", j)
            results <- j * 2
        }
    }

    func main() {
        // 初始化两个管道用以给工作进程分配任务及接收结果
        jobs := make(chan int, 100)
        results := make(chan int, 100)
    
        // 启动三个工作进程
        // 此时它们是阻塞的，因为管道jobs是空的
        for w:= 1; w <= 3; w++ {
            go worker(w, jobs, results)
        }
    
        // 初始化5个任务
        // 关闭管道jobs，标识没有多余的任务了
        for j:= 1; j <= 5; j++ {
            jobs <- j
        }
        close(jobs)
    
        // 收集所有任务的结果
        for a := 1; a <= 5; a++ {
            <-results
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run worker-pool.go
    worker 3 started job 1
    worker 2 started job 3
    worker 1 started job 2
    worker 3 finished job 1
    worker 3 started job 4
    worker 2 finished job 3
    worker 1 finished job 2
    worker 2 started job 5
    worker 2 finished job 5
    worker 3 finished job 4
```

通过输出可以看出5个任务被多个工作进程执行。程序只消耗了2秒多而不是5秒，是因为所有的任务被3个工作进程并发处理。

原文链接：[Go by Example: Worker Pools](https://gobyexample.com/worker-pools)





