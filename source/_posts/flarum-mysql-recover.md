---
title: 记一次mysql被黑的经历
date: 2023-03-17 23:11:11
categories:
- php
- mysql
tags:
- flarum
---
# 记一次mysql被黑的经历

## 背景

前段时间用`flarum`搭了个论坛，由于docker设置mysql端口对外开放且密码很简单，不出意外中招了:) 一大早有群友找上来说网站又打不开，我收住了即将第五十三次劝她换掉e家宽的冲动，打开网页一看事情没那么简单～

## 中招
![LGKvq.png](https://i.328888.xyz/2023/03/17/LGKvq.png)

错误日志：这他娘的库子都没了:)
![LGoEN.png](https://i.328888.xyz/2023/03/17/LGoEN.png)
<!-- more -->
mysql：谁还硬给我套上一条形迹可疑的库子
![LGS2o.png](https://i.328888.xyz/2023/03/17/LGS2o.png)

![LGsTV.png](https://i.328888.xyz/2023/03/17/LGsTV.png)

mysql：是他动的手，看你选择我还是选择btc吧

![LG9Wd.png](https://i.328888.xyz/2023/03/17/LG9Wd.png)

虽说不值钱吧，也Google了一下，发现大把的幸运儿都上了这趟车，基本都是弱密码、对外开放的mongo、mysql服务被自动化扫描工具拉上了车，对比其他网友我这0.21btc的价还算良心，要不给他汇过去？犹豫之余花呗账单发来了本期还款提醒，0.21rmb我也没能力给了！！淦他！！

## 止损

从冰箱拽出已经见底的快乐水猛嘬一口缓了缓神， 一看这新库子的创建时间和编号000017的binlog时间有猫腻

![LGJpb.png](https://i.328888.xyz/2023/03/17/LGJpb.png)

### 祭出mysqlbinlog追踪溯源

`mysqlbinlog --no-defaults --base64-output=decode-rows -v --database flarum_hnxc data/mysql/binlog.000017 | less`
![LGM4z.png](https://i.328888.xyz/2023/03/17/LGM4z.png)

```sql
#230316 22:22:55 server id 1  end_log_pos 817941 CRC32 0xb1803186 	Xid = 124243
COMMIT/*!*/;
# at 817941
#230317  2:22:11 server id 1  end_log_pos 818018 CRC32 0x6ccc23b0 	Anonymous_GTID	last_committed=801	sequence_number=802	rbr_only=no	original_committed_timestamp=1678990932030791	immediate_commit_timestamp=1678990932030791	transaction_length=202
# original_commit_timestamp=1678990932030791 (2023-03-17 02:22:12.030791 CST)
# immediate_commit_timestamp=1678990932030791 (2023-03-17 02:22:12.030791 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990932030791*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 818018
#230317  2:22:11 server id 1  end_log_pos 818143 CRC32 0x6a55306b 	Query	thread_id=9046	exec_time=1	error_code=0	Xid = 138918
SET TIMESTAMP=1678990931/*!*/;
SET @@session.foreign_key_checks=0/*!*/;
SET @@session.sql_mode=1168113696/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=45,@@session.collation_connection=45,@@session.collation_server=255/*!*/;
DROP DATABASE flarum_hnxc
/*!*/;
# at 818143
#230317  2:22:13 server id 1  end_log_pos 818220 CRC32 0x3bb6c0c2 	Anonymous_GTID	last_committed=802	sequence_number=803	rbr_only=no	original_committed_timestamp=1678990933086201	immediate_commit_timestamp=1678990933086201	transaction_length=244
# original_commit_timestamp=1678990933086201 (2023-03-17 02:22:13.086201 CST)
# immediate_commit_timestamp=1678990933086201 (2023-03-17 02:22:13.086201 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990933086201*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 818220
#230317  2:22:13 server id 1  end_log_pos 818387 CRC32 0x23b60335 	Query	thread_id=9048	exec_time=0	error_code=0	Xid = 138923
SET TIMESTAMP=1678990933/*!*/;
SET @@session.foreign_key_checks=1/*!*/;
/*!80016 SET @@session.default_table_encryption=0*//*!*/;
CREATE DATABASE IF NOT EXISTS README_TO_RECOVER_A
/*!*/;
# at 818387
#230317  2:22:13 server id 1  end_log_pos 818466 CRC32 0x38d5237c 	Anonymous_GTID	last_committed=803	sequence_number=804	rbr_only=no	original_committed_timestamp=1678990933650518	immediate_commit_timestamp=1678990933650518	transaction_length=261
# original_commit_timestamp=1678990933650518 (2023-03-17 02:22:13.650518 CST)
# immediate_commit_timestamp=1678990933650518 (2023-03-17 02:22:13.650518 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990933650518*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 818466
#230317  2:22:13 server id 1  end_log_pos 818648 CRC32 0xb40c7f04 	Query	thread_id=9048	exec_time=0	error_code=0	Xid = 138926
use `README_TO_RECOVER_A`/*!*/;
SET TIMESTAMP=1678990933/*!*/;
/*!80013 SET @@session.sql_require_primary_key=0*//*!*/;
CREATE TABLE IF NOT EXISTS RECOVER_YOUR_DATA (text VARCHAR(255))
/*!*/;
# at 818648
#230317  2:22:14 server id 1  end_log_pos 818727 CRC32 0x623fc5ec 	Anonymous_GTID	last_committed=804	sequence_number=805	rbr_only=yes	original_committed_timestamp=1678990934196612	immediate_commit_timestamp=1678990934196612	transaction_length=867
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
# original_commit_timestamp=1678990934196612 (2023-03-17 02:22:14.196612 CST)
# immediate_commit_timestamp=1678990934196612 (2023-03-17 02:22:14.196612 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990934196612*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 818727
#230317  2:22:13 server id 1  end_log_pos 818817 CRC32 0x8823f509 	Query	thread_id=9048	exec_time=0	error_code=0
SET TIMESTAMP=1678990933/*!*/;
BEGIN
/*!*/;
# at 818817
#230317  2:22:13 server id 1  end_log_pos 818899 CRC32 0x7d2db7cd 	Table_map: `README_TO_RECOVER_A`.`RECOVER_YOUR_DATA` mapped to number 214
# at 818899
#230317  2:22:13 server id 1  end_log_pos 819157 CRC32 0x9d8fc1a8 	Write_rows: table id 214 flags: STMT_END_F
### INSERT INTO `README_TO_RECOVER_A`.`RECOVER_YOUR_DATA`
### SET
###   @1='All your data is a backed up. You must pay 0.21 BTC to 18FJ6C6TBpv6va7vLKDri9ibGDWgnYx8Me After 48 hours expiration we will leak and expose all your data. After 48 hours the database dump will be deleted from our server!'
# at 819157
#230317  2:22:14 server id 1  end_log_pos 819239 CRC32 0xe67f3dbd 	Table_map: `README_TO_RECOVER_A`.`RECOVER_YOUR_DATA` mapped to number 214
# at 819239
#230317  2:22:14 server id 1  end_log_pos 819484 CRC32 0x732fb5ba 	Write_rows: table id 214 flags: STMT_END_F
### INSERT INTO `README_TO_RECOVER_A`.`RECOVER_YOUR_DATA`
### SET
###   @1='You can buy bitcoin in https://binance.com After paying write to us in the mail with your DB IP: rasmus+2rps0@onionmail.org and you will receive a link to download your database dump. CHECK YOUR SPAM FOLDER!'
# at 819484
#230317  2:22:14 server id 1  end_log_pos 819515 CRC32 0xd9a7afc1 	Xid = 138927
COMMIT/*!*/;
# at 819515
#230317  2:22:14 server id 1  end_log_pos 819592 CRC32 0xfa9fa3bd 	Anonymous_GTID	last_committed=805	sequence_number=806	rbr_only=no	original_committed_timestamp=1678990934381072	immediate_commit_timestamp=1678990934381072	transaction_length=241
# original_commit_timestamp=1678990934381072 (2023-03-17 02:22:14.381072 CST)
# immediate_commit_timestamp=1678990934381072 (2023-03-17 02:22:14.381072 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990934381072*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 819592
#230317  2:22:14 server id 1  end_log_pos 819756 CRC32 0xde317be1 	Query	thread_id=9048	exec_time=0	error_code=0	Xid = 138930
SET TIMESTAMP=1678990934/*!*/;
REVOKE INSERT, DELETE, CREATE, DROP ON *.* FROM 'root'
/*!*/;
# at 819756
#230317  2:22:14 server id 1  end_log_pos 819833 CRC32 0xb128744c 	Anonymous_GTID	last_committed=806	sequence_number=807	rbr_only=no	original_committed_timestamp=1678990934564760	immediate_commit_timestamp=1678990934564760	transaction_length=186
# original_commit_timestamp=1678990934564760 (2023-03-17 02:22:14.564760 CST)
# immediate_commit_timestamp=1678990934564760 (2023-03-17 02:22:14.564760 CST)
/*!80001 SET @@session.original_commit_timestamp=1678990934564760*//*!*/;
/*!80014 SET @@session.original_server_version=80028*//*!*/;
/*!80014 SET @@session.immediate_server_version=80028*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 819833
#230317  2:22:14 server id 1  end_log_pos 819942 CRC32 0x16becbcb 	Query	thread_id=9048	exec_time=0	error_code=0
SET TIMESTAMP=1678990934/*!*/;
FLUSH PRIVILEGES
/*!*/;
# at 819942
#230317  2:22:15 server id 1  end_log_pos 819965 CRC32 0x8b38ef1d 	Stop
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

通过上面详细的日志看到这厮drop了俺的库，那条新库子也在此时创建，后面还修改了root用户的权限? 有意思，为啥留了update权限给俺:)

```sql
mysql> select * from user where user = 'root';
+-----------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------------------+--------------------------+----------------------------+---------------+-------------+-----------------+----------------------+-----------------------+------------------------------------------------------------------------+------------------+-----------------------+-------------------+----------------+------------------+----------------+------------------------+---------------------+--------------------------+-----------------+
| Host      | User | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Reload_priv | Shutdown_priv | Process_priv | File_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Show_db_priv | Super_priv | Create_tmp_table_priv | Lock_tables_priv | Execute_priv | Repl_slave_priv | Repl_client_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Create_user_priv | Event_priv | Trigger_priv | Create_tablespace_priv | ssl_type | ssl_cipher             | x509_issuer              | x509_subject               | max_questions | max_updates | max_connections | max_user_connections | plugin                | authentication_string                                                  | password_expired | password_last_changed | password_lifetime | account_locked | Create_role_priv | Drop_role_priv | Password_reuse_history | Password_reuse_time | Password_require_current | User_attributes |
+-----------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------------------+--------------------------+----------------------------+---------------+-------------+-----------------+----------------------+-----------------------+------------------------------------------------------------------------+------------------+-----------------------+-------------------+----------------+------------------+----------------+------------------------+---------------------+--------------------------+-----------------+
| %         | root | Y           | N           | Y           | N           | N           | N         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          | 0x                     | 0x                       | 0x                         |             0 |           0 |               0 |                    0 | caching_sha2_password | $A$005$j>
.7bh<"x7q-ZALT24E1ClL.JAtNSE4Bjd4.BAam2pFOYp90WiH0bxGpc7 | N                | 2023-03-09 17:01:43   |              NULL | N              | Y                | Y              |                   NULL |                NULL | NULL                     | NULL            |
| localhost | root | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          | 0x                     | 0x                       | 0x                         |             0 |           0 |               0 |                    0 | caching_sha2_password | $A$005$&7Pz3w XDiVG9Fd0jiC8o/4Gz6j0QZbPwUEdAiSfgO.T8re68rW0gdJAOMm5 | N                | 2023-03-09 17:01:43   |              NULL | N              | Y                | Y              |                   NULL |                NULL | NULL                     | NULL            |
+-----------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------------------+--------------------------+----------------------------+---------------+-------------+-----------------+----------------------+-----------------------+------------------------------------------------------------------------+------------------+-----------------------+-------------------+----------------+------------------+----------------+------------------------+---------------------+--------------------------+-----------------+
2 rows in set (0.05 sec)
```

### 恢复root权限

因为`root@%` 的create权限被收回，旧库子不能被直接创建，此时可以进入mysql容器通过`root@localhost`直接操作，但业务上还需要远程登录root账号的权限，所以需要使用好心人留下的update权限含泪重新给自己加回create权限及其他被收回的权限：

```sql
update mysql.user set Create_priv = 'Y', ... where User = 'root'
```

### 恢复数据

将所有有用的binlog重新导入db，记得编号**000017**这条需要特殊处理舍弃后面的黑化日志：

```bash
$: ...
$: mysqlbinlog data/mysql/binlog.000016 | mysql -h 127.0.0.1 -uroot -p

$: mysqlbinlog --stop-position=817941 data/mysql/binlog.000017 | mysql -h 127.0.0.1 -uroot -p
```

重新打开网站数据恢复如初～

## 反击

反击？我嚼得这兄弟还是手下留情了，是不是可以做的更绝一点，后面得想想怎么再黑化黑化这个脚本

## 总结

遇事不慌，别老想着打钱打钱，你有钱吗？binlog在mysql就在，一步一步来能救一个算一个

**最重要的是mysql端口不要暴露到外网！！**

> [MySQL 5.7 - 通过 BINLOG 恢复数据 - 来份锅包肉 - 博客园](https://www.cnblogs.com/michael9/p/11923483.html)