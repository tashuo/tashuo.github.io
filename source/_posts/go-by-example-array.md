---
title: Go by Example - Array
date: 2017-07-11 16:10:26
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

## <center>Go by Example: Array</center>

golang中的数组是一个**特定长度**的元素有序集合。

```golang
    package main

    import "fmt"

    func main() {
        // 定义一个长度为5的元素类型为int的数组，元素的类型和数组的长度都是数组的属性。默认情况数组是零值，如int型的话就是数组长度个0。
        var a[5] int
        fmt.Println("emp:", a)

        // 根据索引设置／获取数组中某项的值
        a[4] = 100
        fmt.Println("set:", a)
        fmt.Println("get:", a[4])
        
        // 获取数组的长度
        fmt.Println("len:", len(a))
        
        // 定义并初始化一个数组
        b := [5]int{1, 2, 3, 4, 5}
        fmt.Println("dcl:", b)

        // 数组类型是一维的，但是可以通过复合类型来构造多维的结构。
        var twoD [2][3]int
        for i := 0; i < 2; i++ {
            for j := 0; j < 3; j++ {
                twoD[i][j] = i + j
            }
        }
        fmt.Println("2d:", twoD)
    }
```

```bash
    $ go run arrays.go
    emp: [0 0 0 0 0]
    set: [0 0 0 0 100]
    get: 100
    len: 5
    dcl: [1 2 3 4 5]
    2d:  [[0 1 2] [1 2 3]]
```


原文链接：[Go by Example: Arrays](https://gobyexample.com/arrays)
