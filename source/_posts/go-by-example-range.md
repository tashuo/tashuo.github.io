---
title: Go by Example - Range
date: 2017-07-12 16:00:10
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

## <center>Go by Example: Range</center>

range用来遍历各种不同数据结构中的元素。

```golang
    package main

    import "fmt"

    func main() {
        // 遍历slice，同array。
        nums := []int{2, 3, 4}
        sum := 0
        for _, num := range nums {
            sum += num
        }
        fmt.Println("sum:", sum)

        // slice和array在遍历时都会返回索引和值。
        for i, num := range nums {
            if num == 3 {
                fmt.Println("index:", i)
            }
        }

        // 遍历map。
        kvs := map[string]string{"a": "apple", "b": "banner"}
        for k, v := range kvs {
            fmt.Printf("%s -> %s\n", k, v)
        }

        // 也可以只接收一个返回值，range会遍历map所有的key并返回。
        for k := range kvs {
            fmt.Println("key:", k)
        }

        // 对字符串进行遍历是通过unicode字节编码进行。
        // 例子中的英文字符都是一个字节的长度，而中文字符‘哈’的utf8编码是三个字节长度，对应的二进制表示‘21704’，所以‘l’的索引值就是5。
        for i, c := range "go哈lang" {
            fmt.Println(i, c)
        }
    }
```

```bash
    $ go run range.go
    sum: 9
    index: 1
    a -> apple
    b -> banana
    key: a
    key: b
    0 103
    1 111
    2 21704
    5 108
    6 97
    7 110
    8 103
```

原文链接：[Go by Example: Range](https://gobyexample.com/range)







