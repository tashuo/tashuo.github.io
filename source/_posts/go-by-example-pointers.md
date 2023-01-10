---
title: Go by Example - Pointers
date: 2017-07-18 17:41:09
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

## <center>Go by Example: Pointers</center>

golang支持指针，允许在程序中传递值和记录的引用。

```golang
    package main

    import "fmt"

    // 接收一个int类型参数。
    // 对ival的赋值不会影响外部变量的值。
    func zeroval(ival int) {
        ival = 0
    }

    // 接收一个int指针类型的参数。
    // 对*iptr的赋值操作会直接修改到iptr指针所指向内存的地址。
    func zeroptr(iptr *int) {
        *iptr = 0
    }

    func main() {
        i := 1
        fmt.Println("initial:", i)

        // 传值。
        zeroval(i)
        fmt.Println("zeroval:", i)
        
        // 传引用。
        zeroptr(&i)
        fmt.Println("zeroptr:", i)
        
        // 打印指针指向的内存地址。
        fmt.Println("pointer:", &i)
    }
```

```bash
    $ go run pointers.go
    initial: 1
    zeroval: 1
    zeroptr: 0
    pointer: 0x42131100
```

原文链接：[Go by Example: Pointers](https://gobyexample.com/pointers)





