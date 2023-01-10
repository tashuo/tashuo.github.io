---
title: Go by Example - If/Else
date: 2017-07-11 10:34:20
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

## <center>Go by Example: If/Else</center>

goalng中的if条件语句可以省略括号，但是大括号不能少。而且golang没有常用的三元运算符`?:`, 所以必须用if来实现。

```golang
    package main

    import "fmt"

    func main() {
        // 经典的if／else
        if 7%2 == 0 {
            fmt.Println("7 is even")
        } else {
            fmt.Println("7 is odd")
        }

        // if
        if 8%4 == 0 {
            fmt.Println("8 is divisible by 4")
        }

        // 条件判断语句前可以执行一条语句（简直酷毙了），并且定义的变量可以在所有条件分支引用
        if num := 9; num < 0 {
            fmt.Println(num, "is negative")
        } else if num < 10 {
            fmt.Println(num, "has 1 digit")
        } else {
            fmt.Println(num, "has multiple digits")
        }
    }
```

```bash
    $ go run if-else.go 
    7 is odd
    8 is divisible by 4
    9 has 1 digit
```

原文链接：[Go by Example: If/Else](https://gobyexample.com/if-else)




