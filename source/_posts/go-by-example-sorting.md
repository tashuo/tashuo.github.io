---
title: Go by Example - Sorting
date: 2019-04-13 18:20:18
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

## <center>Go by Example: Sorting</center>
golang的`sort`包可以对内置类型和用户定义的类型进行排序操作，接下来我们先看下内置类型。

<!-- more -->

``` golang
    package main
    
    import "fmt"
    import "sort"
    
    func main() {
        // Sort包适用于特定的内置类型
        // 这里是字符串排序的示例，需要注意排序改变的是变量的值，该方法并不会返回一个新的值
        strs := []string{"c", "a", "b"}
        sort.Strings(strs)
        fmt.Println("Strings:", strs)
    
        // 这里是整数排序的示例
        ints := []int{7, 2, 4}
        sort.Ints(ints)
        fmt.Println("Ints:", ints)
    
        // 还可以用Sort包判断一个切片是否有序
        s := sort.IntsAreSorted(ints)
        fmt.Println("Sorted:", s)
    }
```

``` bash
    $ go run sorting.go
    Strings: [a b c]
    Ints: [2 4 7]
    Sorted: true
```

原文链接：[Go by Example: Slices](https://gobyexample.com/sorting)


