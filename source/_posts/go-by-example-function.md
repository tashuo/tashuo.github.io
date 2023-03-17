---
title: Go by Example - Functions
date: 2017-07-13 10:15:19
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

## <center>Go by Example: Functions</center>

函数是golang的核心。

```golang
    package main

    import "fmt"
    
    // 接收两个int类型的参数，并返回两者之和。
    // golang的函数需要显式的return。
    func plus(a int, b int) int {
        return a + b
    }

    // 多个同样类型的参数简便的定义。
    func plusPlus(a, b, c int) int {
        return a + b + c
    }

    func main() {
        res := plus(1, 2)
        fmt.Println("1+2=", res)

        res = plusPlus(1, 2, 3)
        fmt.Println("1+2+3=", res)
    }
```

```bash
    $ go run functions.go 
    1+2 = 3
    1+2+3 = 6
```

原文链接：[Go by Example: Functions](https://gobyexample.com/functions)





