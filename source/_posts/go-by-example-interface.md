---
title: Go by Example - Interfaces
date: 2017-08-02 11:05:36
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

## <center>Go by Example: Interfaces</center>

golang中接口被定义为一系列方法的集合。

```golang
    package main

    import "fmt"
    import "math"

    // 定义一个基本几何图形的接口
    type geometry interface {
        area() float64
        perim() float64
    }

    // 定义两个结构体 rect和circle 实现这个接口
    type rect struct {
        width, height float64
    }

    type circle struct {
        radius float64
    }

    // golang中实现一个接口类型，只需实现该接口所有的方法
    func (r rect) area() float64 {
        return r.width * r.height
    }

    func (r rect) perim() float64 {
        return 2*r.width + 2*r.height
    }

    func (c circle) area() float64 {
        return math.Pi * c.radius * c.radius
    }

    func (c circle) perim() float64 {
        return 2 * math.Pi * c.radius
    }

    // 如果一个变量实现了一个接口类型，则该变量可以调用接口所有的方法
    func measure(g geometry) {
        fmt.Println(g)
        fmt.Println(g.area())
        fmt.Println(g.perim())
    }

    func main() {
        r := rect{width: 3, height: 4}
        c := circle{radius: 5}

        measure(r)
        measure(c)
    }
```

```bash
    $ go run interfaces.go
    {3 4}
    12
    14
    {5}
    78.53981633974483
    31.41592653589793
```


延伸阅读: [how-to-use-interfaces-in-go](http://jordanorelli.tumblr.com/post/32665860244/how-to-use-interfaces-in-go)

原文链接：[Go by Example: Interfaces](https://gobyexample.com/interfaces)


