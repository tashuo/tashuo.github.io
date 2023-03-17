---
title: Go by Example - Multiple Return Values
date: 2017-07-18 16:21:35
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

## <center>Go by Example: Multiple Return Values</center>

golang原生支持多个返回值，这在golang的编程习惯中很常见，如函数同时返回返回值和error。

```golang
    package main

    import "fmt"

    // (int, int)标识函数会返回两个int值
    func vals() (int, int) {
        return 3, 7
    }

    func main() {
        // 使用两个变量接收两个返回值
        a, b := vals()
        fmt.Println(a)
        fmt.Println(b)

        // 使用‘_’可以忽略掉该返回值
        _, c := vals()
        fmt.Println(c)
    }
```

```bash
    $ go run multiple-return-values.go
    3
    7
    7
```


原文链接：[Go by Example: Multiple Return Values](https://gobyexample.com/multiple-return-values)




