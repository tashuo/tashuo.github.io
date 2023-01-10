---
title: Go by Example - Slices
date: 2017-07-12 10:20:40
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

## <center>Go by Example: Slices</center>

slice(切片)是golang中一个很重要的数据类型，它给序列提供了比数组更加强大的功能接口。

同数组不同，切片只由包含的元素确定，跟元素个数无关。使用内置的make函数可以初始化一个空的切片。
<!-- more -->
```golang
    package main

    import "fmt"

    func main() {
        // 定义一个长度为3的字符串切片
        s := make([]string, 3)
        fmt.Println("emp", s)

        // 同数组一样，可以通过索引赋值、取值
        s[0] = "a"
        s[1] = "b"
        s[2] = "c"
        fmt.Println("set:", s)
        fmt.Println("get:", s[2])

        // 获取切片的长度，同数组
        fmt.Println("len:", len(s))

        // 除了上面的基本操作，切片还支持另外一些内置操作，这使得切片比数组要更加强大。
        // 其中一个就是append，可以在切片结尾追加一个或多个元素，并且返回一个新的切片。
        s = append(s, "d")
        s = append(s, "e", "f")
        fmt.Println("apd:", s)

        // 另一个内置函数copy，可以将一个切片的值copy到另一个切片。
        c := make([]string, len(s))
        copy(c, s)
        fmt.Println("cpy:", c)

        // 切片还支持一种特有的语法`slice[low:high]`，可以获取索引值从low到high的切片值。
        // low和high都可以省略，low省略的话默认是0，high省略的话默认是切片最后一个索引值。
        l := s[2:5]
        fmt.Println("sl1:", l)
        l = s[:5]
        fmt.Println("sl2:", l)
        l = s[2:]
        fmt.Println("sl3:", l)

        // 也可以在定义切片的时候初始化赋值。
        t := []string{"g", "h", "i"}
        fmt.Println("dcl:", t)

        // 切片也可以用来构建多维的数据结构。而跟多维数组不同，子切片的长度可以不一样。
        twoD := make([][]int, 3)
        for i := 0; i < 3; i++ {
            innerLen := i + 1
            twoD[i] = make([]int, innerLen)
            for j := 0; j < innerLen; j++ {
                twoD[i][j] = i + j
            }
        }
        fmt.Println("2d:", twoD)
    }
```

```bash
    $ go run slices.go
    emp: [  ]
    set: [a b c]
    get: c
    len: 3
    apd: [a b c d e f]
    cpy: [a b c d e f]
    sl1: [c d e]
    sl2: [a b c d e]
    sl3: [c d e f]
    dcl: [g h i]
    2d:  [[0] [1 2] [2 3 4]]
```

更多关于slice的细节推荐Google团队的一篇文章：[Go Slices: usage and internals

](https://blog.golang.org/go-slices-usage-and-internals)

原文链接：[Go by Example: Slices](https://gobyexample.com/slices)


