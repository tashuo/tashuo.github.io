---
title: Go by Example - Maps
date: 2017-07-12 11:25:16
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

## <center>Go by Example: Maps</center>

map是golang内置关联数组类型，在其他语言中也被称为hash或字典。

```golang
    package main

    import "fmt"

    func main() {
        // 使用内置的make函数定义一个map，`make(map[key-type]val-type)`
        // 根据索引赋值
        m := make(map[string]int)
        m["k1"] = 7
        m["k2"] = 13
        fmt.Println("map:", m)

        // 根据索引取值
        v1 := m["k1"]
        fmt.Println("v1:", v1)

        // 获取map的长度
        fmt.Println("len:", len(m))

        // 内置delete函数删除map中的元素
        delete(m, "k2")
        fmt.Println("map:", m)

        // 获取map中某个索引对应的值。
        // 返回的第一个值是map中对应的值，如果存在的话，否则是map的元素类型的零值。
        // 返回的第二值可选，如果k2在m中，prs为true。否则，prs为false，并且返回的第一个值是map的元素类型的零值。
        // 该例中省略了第一个返回值。这种用法可以用来判断某个key是否在map中存在，如果直接使用第一返回值是无法判断返回的零值是因为key不存在还是该key对应的元素刚好是零值。
        _, prs := m["k2"]
        fmt.Println("prs:", prs)

        // map也可以如其他元素一样定义和初始化赋值一起。
        n := map[string]int{"foo": 1, "bar": 2}
        fmt.Println("map:", n)
    }
```

```bash
    $ go run maps.go 
    map: map[k1:7 k2:13]
    v1:  7
    len: 2
    map: map[k1:7]
    prs: false
    map: map[foo:1 bar:2]
```

原文链接：[Go by Example: Maps](https://gobyexample.com/maps)





