---
title: Go by Example - Range over Channels
date: 2017-08-25 10:44:04
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

## <center>Go by Example: Range over Channels</center>

在前面的示例中看到`for`和`range`语句如何给基本数据类型提供迭代操作，我们也可以用它们来迭代channel中接收到的值。

```golang
    package main

    import "fmt"

    func main() {
        // 定义有两个字符串缓存区的channel queue
        // 我们将要迭代queue中的两个值
        queue := make(chan string, 2)
        queue <- "one"
        queue <- "two"

        close(queue)

        // range会迭代queue中接收到的每个值
        // 由于在上面已经close掉管道queue，所以在接收完两个值之和迭代就结束了
        for elem := range queue {
            fmt.Println(elem)
        }
    }
```

```bash
    tashuo:golang ta_shuo$ go run r_channel.go
    one
    two
```

上面的示例中也看出可以关闭掉一个还有待接收值的非空管道。

原文链接：[Go by Example: Range over Channels](https://gobyexample.com/range-over-channels)


