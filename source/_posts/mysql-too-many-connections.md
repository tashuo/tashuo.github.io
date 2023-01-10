---
title: MySQL连接数过多的处理
date: 2016-12-19 09:59:05
categories:
- 笔记
tags:
- mysql
---
场景：MySQL出现“too many connections”错误  

MySQL默认会额外保留一条供root用户建立的连接，以确保root用户可随时登录查看并检测服务器状态，当出现上述场景时，我们就可通过`show [full] processlist`命令查看当前所有进程状态，通过kill掉某些慢sql及分析产生原因来优化sql代码. 操作如下.

### 查看mysql所有执行状态
`mysql> show full processlist`  

如果MySQL登录用户有[PROCESS权限](http://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_process)，列表会输出所有进程信息，否则只有当前登录用户所属的进程，若不加full关键字，Info字段信息会只截取前100个字符显示.  

列表字段说明：
#### id
MySQL连接的id标示
#### User
建立此次连接的MySQL user
#### Host
此次连接建立的客户端，如果是系统进程此处则为空。MySQL通过此次TCP/IP连接中的`host_name:client_port`信息获取host name
#### db
操作的数据库，可能为null
#### Command
当前sql语句的类型，一般为休眠（sleep），查询（query），连接（connect）
#### Time
当前进程在当前状态停留的时间，单位秒
#### State
当前进程的[执行状态](http://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html)，大部分的状态切换很快，**如果发现某个进程在某个状态阻塞很久说明sql执行可能有问题**
#### Info
进程详情信息，如select查询进程此处会是select语句
<!-- more -->
### 杀掉某个进程
`mysql> kill {id}`  

通过上面的state/Command/Time字段可大致判断出有问题的sql语句，可通过kill命令杀掉该进程，MySQL命令只能一个个kill进程，批量kill需结合循环进行，示例如下:
``` php
<?php
    # kill掉阻塞到当前状态已超过200秒的进程
    $result = mysqli_query("SHOW FULL PROCESSLIST");
    while ($row = mysqli_fetch_array($result)) {
      $process_id = $row["Id"];
      if ($row["Time"] > 200 ) {
        $sql = "KILL $process_id";
        mysqli_query($sql);
      }
    }
?>
```
