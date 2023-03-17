---
title: Go by Example - Mutexes
date: 2017-12-13 10:37:07
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

## <center>Go by Example: Mutexes</center>
在前面的示例中我们看到如何通过原子操作来管理简单的计数器。对于更复杂的状态，我们可以使用互斥锁在多个goroutine中安全地访问数据。

```golang
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "sync/atomic"
    "time"
)

func main() {
    // 此处状态是一个map
    var state = make(map[int]int)
    // 初始化锁
    var mutex = &sync.Mutex{}
    // 定义两个变量用于追踪读和写的总次数
    var readOps uint64 = 0
    var writeOps uint64 = 0

    // 启动100个goroutines读取状态值，每个goroutine耗时1毫秒
    for r := 0; r < 100; r++ {
        go func() {
            total := 0
            for {
                // 随机读取状态的某一个值
                // 1、加锁
                // 2、读取
                // 3、解锁
                key := rand.Intn(5)
                mutex.Lock()
                total += state[key]
                mutex.Unlock()
                // 次数自增1
                atomic.AddUint64(&readOps, 1)
    
                // 两次读取之间睡眠1毫秒
                time.Sleep(time.Millisecond)
            }
        }()
    }

    // 同上，启动10个goroutines模拟写操作
    // 随机给状态赋值，写操作次数自增1，睡眠1毫秒
    for w := 0; w < 10; w++ {
        go func() {
            for {
                key := rand.Intn(5)
                val := rand.Intn(100)
                mutex.Lock()
                state[key] = val
                mutex.Unlock()
                atomic.AddUint64(&writeOps, 1)
                time.Sleep(time.Millisecond)
            }
        }()
    }

    // 睡眠一秒等待读写操作
    // 打印出读写次数
    time.Sleep(time.Second)
    readOpsFinal := atomic.LoadUint64(&readOps)
    fmt.Println("readOps:", readOpsFinal)
    writeOpsFinal := atomic.LoadUint64(&writeOps)
    fmt.Println("writeOps:", writeOpsFinal)

    // 使用锁取出最后的状态值
    mutex.Lock()
    fmt.Println("state:", state)
    mutex.Unlock()
}
```
<!-- more -->
```bash
tashuo:golang ta_shuo$ go run mutex.go
readOps: 78300
writeOps: 7830
state: map[1:86 0:61 4:17 2:26 3:26]
```

执行程序，根据输出得知读写次数分别为78300和7830，最后的状态为map[1:86 0:61 4:17 2:26 3:26]


原文链接：[Go by Example: Mutexes](https://gobyexample.com/mutexes)


