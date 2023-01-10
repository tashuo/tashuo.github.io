---
title: Go by Example - Variadic Functions
date: 2017-07-18 16:34:44
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

## <center>Go by Example: Variadic Functions</center>

可以使用任意数量的参数来调用可变参数函数，如`fmt.Println`就是一个常用的可变参数函数。

```golang
    package main

    import "fmt"

    // 定义一个函数，可以接受任意数量的int值作为参数。
    func sum(nums ...int) {
        fmt.Print(nums, " ")
        total := 0
        for _, num := range nums {
            total += num
        }
        fmt.Println(total)
    }

    func main() {
        // 使用独立的参数调用。
        sum(1, 2)
        sum(1, 2, 3)
        
        // 使用相应类型的slice调用可变参数函数。func(slice...)。
        nums := []int{1, 2, 3, 4}
        sum(nums...)
    }
```

```bash
    $ go run variadic-functions.go 
    [1 2] 3
    [1 2 3] 6
    [1 2 3 4] 10
```


原文链接：[Go by Example: Variadic Functions](https://gobyexample.com/variadic-functions)






