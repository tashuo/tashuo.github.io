---
title: Go by Example - Closures
date: 2017-07-18 16:45:26
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

## <center>Go by Example: Closures</center>

golang支持匿名函数，可以用来构建闭包。当你需要定一个不需要命名的内连函数时匿名函数很有用。

```golang
    package main
    
    import "fmt"

    // 定义一个函数intSeq返回内部定义的一个匿名函数。
    // 返回的函数通过返回变量i来结束并构建闭包。
    func intSeq() func() int {
        i := 0
        return func() int {
            i += 1
            return i
        }
    }

    func main() {
        // 调用函数intSeq，并将其返回给变量nextInt。
        // 该函数值使用自己拥有的变量i，i在我们调用nextInt时都会更新。
        nextInt := intSeq()

        // 调用nextInt函数
        fmt.Println(nextInt())
        fmt.Println(nextInt())
        fmt.Println(nextInt())
        
        // 变量i的值独立于每个intSeq函数调用。
        nextInts := intSeq()
        fmt.Println(nextInts())
    }
```

```bash
    $ go run closures.go
    1
    2
    3
    1
```


原文链接：[Go by Example: Closures](https://gobyexample.com/closures)











