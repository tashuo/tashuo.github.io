---
title: Go by Example - Constants
date: 2017-07-11 09:49:52
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

## <center>Go by Example: Constants</center>

golang支持字符型、字符串型、布尔型和数值型的常量。

```golang
    package main

    import "fmt"
    import "math"

    // const关键字生命常量
    const s string = "constant"

    func main() {
        fmt.Println(s)

        // const常量可以在任何变量可以声明的地方声明，并且支持科学计数法等算术运算
        const n = 500000000
        const d = 3e20 / n
        fmt.Println(d)

        // 数值型常量默认没有特定的类型，可以显式声明类型，也默认会根据上下文自动确定自身类型
        // 例如下面的语句`math.Sin`函数需要float64类型的参数，n就默认确定为float64类型
        fmt.Println(int64(d))
        fmt.Println(math.Sin(n))
    }
```

```bash
    $: go run constant.go
    constant
    6e+11
    600000000000
    -0.28470407323754404
```


原文链接：[Go by Example: Constants](https://gobyexample.com/constants)









