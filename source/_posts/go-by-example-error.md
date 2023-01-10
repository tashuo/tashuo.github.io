---
title: Go by Example - Errors
date: 2017-08-03 10:55:27
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

## <center>Go by Example: Errors</center>

golang中通过显式、单独的返回值来标识错误是一种常用的语言习惯。这与java和ruby等语言中使用的异常或C语言中有时使用重载单个返回值／错误是不同的。golang的这种处理方式很容易获取方法中返回的错误并且与其他没有返回错误的情况有相同的语句结构。

```golang
    package main

    import "errors"
    import "fmt"

    // 按照惯例，错误一般作为最后一个返回值，并且是内置的error类型
    // errors.New创建一个以给定字符串为值的error
    func f1(arg int) (int, error) {
        if arg == 42 {
            return -1, errors.New("Can't work with 42")
        }

        // nil标识没有错误返回
        return arg + 3, nil
    }

    // 也可以通过定义一个实现了Error方法的类型作为error类型
    // 该Error方法会打印出struct的两个变量
    type argError struct {
        arg int
        prob string
    }
    func (e *argError) Error() string {
        return fmt.Spintf("%d - %s", e.arg, e.prob)
    }

    // 使用&argError构造一个struct，作为error类型值返回
    func f2(arg int) (int, error) {
        if arg == 42 {
            return -1, &argError{arg, "can't work with it"}
        }

        return arg + 3, nil
    }

    func main() {
        // 下面的两个循环测试了两个会返回error值的函数
        for _, i := range []int{7, 42} {
            if r, e := f1(i); e != nil {
                fmt.Println("f1 failed: ", e)
            } else {
                fmt.Println("f1 worked:", r)
            }
        }
        for _, i := range []int{7, 42} {
            if r, e := f2(i); e != nil {
                fmt.Println("f2 failed:", e)
            } else {
                fmt.Println("f2 worked:", r)
            }
        }

        // 如果要使用返回的自定义error数据，你需用断言的方式判断该返回值是自定义error的一个实例
        _, e := f2(42)
        if ae, ok := e.(*argError); ok {
            fmt.Println(ae.arg)
            fmt.Println(ae.prob)
        }
    }
```

```bash
    $ go run errors.go
    f1 worked: 10
    f1 failed: can\'t work with 42
    f2 worked: 10
    f2 failed: 42 - can\'t work with it
    42
    can\'t work with it
```


延伸阅读：[Error handling and Go](https://blog.golang.org/error-handling-and-go)

原文链接：[Go by Example: Errors](https://gobyexample.com/errors)




