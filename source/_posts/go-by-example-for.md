---
title: Go by Example - For
date: 2017-07-11 10:19:10
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

## <center>Go by Example: For</center>

`for`是golang唯一的循环结构。

```golang
    package main

    import "fmt"

    func main() {
        // 最常见的形式，只有一个条件。类似于PHP中的while循环
        i := 1
        for i <= 3 {
            fmt.Println(i)
            i = i + 1
        }

        // 经典的for循环
        for j := 7; j <= 9; j++ {
            fmt.Println(j)
        }

        // 不带任何条件的for循环，类似于PHP中的while(true)循环。除非从函数内部break或return，否则会一直循环下去。
        for {
            fmt.Println("loop")
            break
        }

        // continue关键字，同PHP。停止此次循环的执行，直接进入下一次循环。
        for n:= 0; n <= 5; n++ {
            if n%2 == 0 {
                continue
            }
            fmt.Println(n)
        }
    }
```

```bash
    $ go run for.go
    1
    2
    3
    7
    8
    9
    loop
    1
    3
    5
```


原文链接：[Go by Example: For](https://gobyexample.com/for)



