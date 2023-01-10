---
title: Go by Example - Switch
date: 2017-07-11 15:19:23
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

## <center>Go by Example: Switch</center>

golang也有常用的switch语句。

```golang
    package main

    import "fmt"
    import "time"

    func main() {
        i := 2
        fmt.Print("Write", i, " as ")
        // 典型的switch结构
        switch i {
            case 1:
                fmt.Println("one")
            case 2:
                fmt.Println("two")
            case 3:
                fmt.Println("three")
        }

        // case后可以跟多个条件，逗号分隔。也有常用的default。
        switch time.Now().Weekday() {
            case time.Saturday, time.Sunday:
                fmt.Println("It's the weekend")
            default:
                fmt.Println("It's a weekday")
        }

        // switch后不跟条件判断语句可以模仿if/else的功能
        t := time.Now()
        switch {
            case t.Hour() < 12:
                fmt.Println("It's before noon")
            default:
                fmt.Println("It's after noon")
        }

        // 变量类型判断的switch语句，其中的type是switch独有的用法。
        // 主要用来判断一个interface值的类型。
        whatAmI := func(i interface{}) {
            switch t := i.(type) {
                case bool:
                    fmt.Println("I'm a bool")
                case int:
                    fmt.Println("I'm an int")
                default:
                    fmt.Println("Don't know type %T\n", t)
            }
        }
        whatAmI(true)
        whatAmI(1)
        whatAmI("hey")
    }
```

```bash
    $: go run switch.go
    Write 2 as two
    It's a weekday
    It's after noon
    I'm a bool
    I'm an int
    Don't know type string
```

原文链接：[Go by Example: Switch](https://gobyexample.com/switch)








