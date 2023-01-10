---
title: Unix Domain Socket vs TCP SOCKET
date: 2017-06-20 15:50:29
categories:
- 笔记
tags:
- unix domain socket
- tcp/ip
- php-fpm
---

之前配置lnmp环境时有发现php-fpm既可以监听`127.0.0.1:9000`(ip+端口形式), 也可以监听`/path/to/unix/socket`(Unix Domain Socket形式), 用起来感觉并没有什么差异, 网上查下了资料，大略总结下。

## Unix Domain Socket
> Unix domain socket 或者 IPC socket是一种终端，可以使 **同一台操作系统** 上的两个或多个进程进行数据通信。与管道相比，Unix domain sockets 既可以使用字节流，又可以使用数据队列，而管道通信则只能使用字节流。Unix domain sockets的接口和Internet socket很像，但它不使用网络底层协议来通信。Unix domain socket 的功能是POSIX操作系统里的一种组件。

> Unix domain sockets **使用系统文件的地址来作为自己的身份**。它可以被系统进程引用。所以两个进程可以同时打开一个Unix domain sockets来进行通信。不过这种通信方式是发生在系统内核里而不会在网络里传播。
 
### 同一台操作系统
* 直接弊端: 限定UDS的使用只能在同一台服务器，**即限定Nginx和php必须部署在同一台机**。

### 系统文件标识身份
* 方便利用linux的文件访问权限机制，来控制进程的访问权限。如Nginx运行用户为www, 则该socket文件必须对www用户有读写的权限，而其他没有读写权限的进程无法与php-fpm进程通信。
* *less copying of data, fewer context switches*。相比基于TCP/IP协议栈的形式，UDS减少了如数据安全校验、路由、流量控制、封包解包等操作的性能消耗，在性能表现方面略高，不过这方面的性能优势只是理论上的，实际线上环境的差异应该不大。
<!-- more -->
### 其他
* 网上看到一个很脑洞的UDS使用
``` 
    upstream backend 
    {
        server unix:/var/run/php5-fpm.sock1  weight=100 max_fails=5 fail_timeout=5;
        server unix:/var/run/php5-fpm.sock2  weight=100 max_fails=5 fail_timeout=5;
    }
```

在同一台服务器php-fpm配置两个或更多UDS文件配合Nginx做负载均衡, 性能会略高于普通的单个的upstream backends。


## 所以如何选择？
* php只有一台服务器且与Nginx在同一台机，建议使用Unix Domain Socket方式，性能会略高于TCP/IP，不过基本看不出来差异
* 其他情况就只能监听ip+端口号, 没得选 :)

<br/><br/>
参考资料：
> [unix domain sockets vs. internet sockets](https://lists.freebsd.org/pipermail/freebsd-performance/2005-February/001143.html)