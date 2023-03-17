---
title: Go by Example - Methods
date: 2017-08-02 10:43:42
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

## <center>Go by Example: Methods</center>

golang支持在结构体上定义方法。

```golang
    package main

    import "fmt"

    type rect struct {
        width, height int
    }

    // 在rect的引用类型上定义一个area方法
    func (r *rect) area() int {
        return 2*r.width * 2*r.height
    }

    // 也可以将方法定义在结构体上
    funv (r rect) perim() int {
        return 2*r.width + 2*r.height
    }

    func main() {
        r := rect{width: 10, height: 5}

        // 直接使用结构体来调用两个方法
        fmt.Println("area: ", r.area())
        fmt.Println("perim: ", r.perim())

        rp := &r
        
        // golang会在方法调用时在结构体和引用之间自动转换，你可能希望使用指针来避免方法调用时的值传递，并且可以在方法内修改传入的结构体的值
        fmt.Println("area: ", rp.area())
        fmt.Println("perim: ", rp.perim())
    }
```

```bash
    $ go run methods.go 
    area:  50
    perim: 30
    area:  50
    perim: 30
```


原文链接：[Go by Example: Methods](https://gobyexample.com/methods)








