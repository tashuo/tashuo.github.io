---
title: MySQL常用表结构调整语句
date: 2016-12-19 17:51:31
categories:
- 笔记
- mysql
tags: mysql
---

*   修改表的引擎  
`mysql> ALTER TABLE $table_name ENGINE = InnodB;`

*   修改表名    
`mysql> RENAME TABLE $old_table_name TO $new_table_name`

*   增加主键/索引  
`mysql> ALTER TABLE $tableName ADD (PRIMARY) KEY ($column)`

*   待续...
