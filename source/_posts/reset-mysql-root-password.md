---
title: 重置MySQL root密码
date: 2016-12-19 17:20:13
categories:
- 笔记
- mysql
tags: mysql
---

OSX(brew)环境重置MySQL的root用户密码   

#### 停止MySQL服务
`$ brew services stop mysql`

#### 启动MySQL安全模式
`$ mysqld_safe --skip-grant-tables`  

'--skip-grant-tables' 参数会忽略权限验证及免密码登录

#### 登入MySQL
`$ mysql`

#### 修改root用户密码
`mysql> use mysql;`  
`mysql> update user set authentication_string=PASSWORD('your-new-password') where user='root';`

#### 重启MySQL服务
`$ pkill mysqld`    
`$ brew services start mysql`
