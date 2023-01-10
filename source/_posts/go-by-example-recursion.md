---
title: Go by Example - Recursion
date: 2017-07-18 17:31:28
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

## <center>Go by Example: Recursion</center>

golang也支持递归函数。

```golang
    package main

    import "fmt"

    // 定义一个函数fact递归调用自己直到参数变为0。
    func fact(n int) int {
        if n == 0 {
            return 1
        }

        return n * fact(n-1)
    }

    func main() {
        fmt.Println(fact(7))
    }
```

原文链接：[Go by Example: Recursion](https://gobyexample.com/recursion)






