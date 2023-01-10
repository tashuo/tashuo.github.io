---
title: Go by Example - Values
date: 2017-07-10 15:33:22
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

## <center>Go by Example: Values</center>

golang内置有多种基本数据类型，包含字符串、整型、浮点型、布尔型等等。上面是一些基本类型的示例。

```golang
    package main

    import "fmt"

    func main() {
        // 字符串连接
        fmt.Println("go" + "lang")
    
        // 整数加法
        fmt.Println("1+1 =", 1+1)

        // 浮点数除法
        fmt.Println("7.0/3.0 =", 7.0/3.0)
    
        // 逻辑运算
        fmt.Println(true && false)
        fmt.Println(true || false)
        fmt.Println(!true)
    }
```

```bash
    $ go run values.go
    golang
    1+1 = 2
    7.0/3.0 = 2.3333333333333335
    false
    true
    false
```


原文链接：[Go by Example: Values](https://gobyexample.com/values)






