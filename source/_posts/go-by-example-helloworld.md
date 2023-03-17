---
title: Go by Example - Hello world
date: 2017-07-10 15:18:42
categories:
- 笔记
tags:
---
翻译自 [https://gobyexample.com/](https://gobyexample.com/)

> Go by Example
> 
> Go is an open source programming language designed for building simple, fast, and reliable software.
> 
> Go by Example is a hands-on introduction to Go using annotated example programs. Check out the first example or browse the full list below.

## <center>Go by Example: Hello world</center>

```golang
    package main

    import "fmt"

    func main() {
        fmt.Println("hello world")
    }
```

```bash
    // 直接运行go文件
    $ go run hello-world.go
    hello world

    // 编译go文件，生成可执行的二进制文件，生成的文件名默认同源文件名
    $ go build hello-world.go
    $ ls
    hello-world hello-world.go

    // 执行编译好的可执行文件
    $ ./hello-world
    hello world
```

即go文件可直接通过`go run golang_file`运行，可以通过编译为二进制文件方式运行。

原文链接：[Go by Example: Hello World](https://gobyexample.com/hello-world)







