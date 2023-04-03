# Mysql 文件系统

## 1 概述

- MySQL 是通过文件系统对数据索引后进行存储的

- MySQL 从物理结构上可以分为日志文件和数据及索引文件

- MySQL 在 Linux 中的数据索引文件和日志文件通常放在/var/lib/mysql目录下

- MySQL 通过日志记录了数据库操作信息和错误信息。

## 2 MySQL 日志文件

- [服务日志官方文档](https://dev.mysql.com/doc/refman/5.7/en/server-logs.html)

- 可通过 `show variables like'log_%';` 查看相关配置

- 分类

  - 错误日志: error.log

  - 二进制日志: mysql-bin

  - 查询日志: general_query.log

  - 慢查询日志: slow_query_log.log

  - 事务重做日志: Redolog

  - 中继日志: Relaylog

> 查看数据库配置变量命令
>
> ```mysql
> mysql> show variables like'log_%';
> +----------------------------------------+---------------------+
> | Variable_name                          | Value               |
> +----------------------------------------+---------------------+
> | log_bin                                | OFF                 |
> | log_bin_basename                       |                     |
> | log_bin_index                          |                     |
> | log_bin_trust_function_creators        | OFF                 |
> | log_bin_use_v1_row_events              | OFF                 |
> | log_builtin_as_identified_by_password  | OFF                 |
> | log_error                              | /var/log/mysqld.log |
> | log_error_verbosity                    | 3                   |
> | log_output                             | FILE                |
> | log_queries_not_using_indexes          | OFF                 |
> | log_slave_updates                      | OFF                 |
> | log_slow_admin_statements              | OFF                 |
> | log_slow_slave_statements              | OFF                 |
> | log_statements_unsafe_for_binlog       | ON                  |
> | log_syslog                             | OFF                 |
> | log_syslog_facility                    | daemon              |
> | log_syslog_include_pid                 | ON                  |
> | log_syslog_tag                         |                     |
> | log_throttle_queries_not_using_indexes | 0                   |
> | log_timestamps                         | UTC                 |
> | log_warnings                           | 2                   |
> +----------------------------------------+---------------------+
> 21 rows in set (0.01 sec)
> ```

> 数据库配置文件参数调整
>
> 参数说明可参考[官方文档](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_log_error_verbosity)
>
> ```shell
> # 修改数据配置参数文件，将相关变量写入文件
> vim /etc/my.cnf
> # 重启 MySQL 使参数生效
> systemctl restart mysqld
> ```

### 2.1 MySQL 错误日志

- [官方文档 The Error Log](https://dev.mysql.com/doc/refman/5.7/en/error-log.html)
- [官方文档 Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html)
- 默认开启，且从5.5.7以后无法关闭错误日志

- 记录运行过程中遇到的所有严重的错误信息，以及每次启动和关闭的详细信息

- 参数配置

  - log_error: 指定错误日志存储位置

  - log_warnings: 是否将警告信息输出到错误日志中，默认值在MySQL 5.7.2之前为1，从5.7.2开始为2
    - 0 告警信息不写入错误日志中 可理解为 java 程序日志级别设置为 ERROR
    - 1 告警信息写入错误日志中 可理解为 java 程序日志级别设置为 WARN
    - 若 >=2，告警及提示信息都将写入错误日志中 可理解为 java 程序日志级别设置为 INFO
  - log_error_verbosity
    - 在 MySQL 5.7.2 及更高版本中，仍然允许使用 log_warnings，但映射到 log_error_verbosity 的使用上，如下所示：
    - 设置 log_warnings=0 相当于 log_error_verbosity=1（仅限错误）。
    - 设置 log_warnings=1 等价于 log_error_verbosity=2（错误，警告）。
    - 设置 log_warnings=2（或更大）相当于 log_error_verbosity=3（错误、警告、信息），如果指定了更大的值，服务器将 log_warnings 设置为2

修改配置：

```shell
# 编辑配置文件
vim /etc/my.cnf
# 在 [mysqld] 下添加或改动参数 亲测配置 log_error 和 log-error 效果相同
log-error=/var/log/mysqld.log
log-warnings=2
# 重启生效
systemctl restart mysqld
```

查看当前配置：

```mysql
mysql> show variables like'log_error';
+---------------+---------------------+
| Variable_name | Value               |
+---------------+---------------------+
| log_error     | /var/log/mysqld.log |
+---------------+---------------------+
1 row in set (0.00 sec)

mysql> show variables like'log_warnings';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_warnings  | 2     |
+---------------+-------+
1 row in set (0.00 sec)
```

查看错误日志：

使用错误密码登录时错误日志会打印 Access denied 相关日志

```shell
[root@iZ2ze8o3e6gtnilf7twmf1Z ~]# tail -f /var/log/mysqld.log
2022-10-18T12:14:08.090613Z 0 [Note]   - '::' resolves to '::';
2022-10-18T12:14:08.090635Z 0 [Note] Server socket created on IP: '::'.
2022-10-18T12:14:08.102236Z 0 [Note] Event Scheduler: Loaded 0 events
2022-10-18T12:14:08.102537Z 0 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.7.40'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
2022-10-18T12:19:10.225527Z 3 [Warning] IP address '34.78.6.216' has been resolved to the host name '216.6.78.34.bc.googleusercontent.com', which resembles IPv4-address itself.
2022-10-18T12:19:10.440972Z 3 [Note] Got an error reading communication packets
2022-10-18T12:19:10.835817Z 4 [Warning] IP address '130.211.54.158' has been resolved to the host name '158.54.211.130.bc.googleusercontent.com', which resembles IPv4-address itself.
2022-10-18T12:19:11.525541Z 4 [Note] Access denied for user 'root'@'130.211.54.158' (using password: NO)
2022-10-18T12:43:32.673955Z 5 [Note] Access denied for user 'root'@'localhost' (using password: YES)
```



### 2.2 MySQL 二进制日志

- [官方文档：The Binary Log](https://dev.mysql.com/doc/refman/5.7/en/binary-log.html)
- 默认关闭

- 记录数据库所有的 DDL 语句和 DML 语句，以事件的形式保存，描述了数据的变更顺序

- 还包括了每个更新语句的执行时间信息。

- 如果是 DDL 语句，则直接记录

- 而 DML 语句，必须通过事务提交才能记录

- 主要用于实现mysql主从复制、数据备份、数据恢复

配置中 bin 是binlog 日志文件的 basename，binlog 日志文件的完整名称: bin.000001

- 参数配置
  - log-bin 设置的是 basename 日志文件的完整名称类似于 bin.000001

修改配置：

```shell
# 编辑配置文件
vim /etc/my.cnf
# 在 [mysqld] 下添加或改动参数，注意 server_id 不配置会报错，另外如果重启报错需要看一下日志并给相应文件夹赋权
server_id=2
log-bin=bin
# 重启生效
systemctl restart mysqld
```

查看配置：

```mysql
mysql> show variables like'log_bin%';
+---------------------------------+--------------------+
| Variable_name                   | Value              |
+---------------------------------+--------------------+
| log_bin                         | ON                 |
| log_bin_basename                | /var/log/bin       |
| log_bin_index                   | /var/log/bin.index |
| log_bin_trust_function_creators | OFF                |
| log_bin_use_v1_row_events       | OFF                |
+---------------------------------+--------------------+
5 rows in set (0.01 sec)
```

查看实际生成的日志文件：

```shell
[root@iZ2ze8o3e6gtnilf7twmf1Z log]# ll | grep 'bin'
-rw-r-----  1 mysql  mysql              177 Oct 18 21:48 bin.000001
-rw-r-----  1 mysql  mysql              177 Oct 18 21:58 bin.000002
-rw-r-----  1 mysql  mysql              177 Oct 18 22:04 bin.000003
-rw-r-----  1 mysql  mysql              154 Oct 18 22:04 bin.000004
-rw-r-----  1 mysql  mysql               80 Oct 18 22:04 bin.index
```



### 2.3 MySQL 通用查询日志

- [官方文档 The General Query Log](https://dev.mysql.com/doc/refman/5.7/en/query-log.html)
- 默认关闭，调试数据库临时打开，不建议长期开启

- 通用查询日志会记录用户的所有操作，其中还包含增删查改等信息，在并发操作大的环境下会产生大量的信息从而导致不必要的磁盘IO，影响mysql的性能的。 

- 参数配置
  - general_log 通用查询日志开关
    - 1 或 ON 表示打开
    - 0 或 OFF 表示关闭
  - general_log_file 通用查询日志记录路径

修改配置：

```shell
# 编辑配置文件
vim /etc/my.cnf
# 在 [mysqld] 下添加或改动参数
general_log=ON
general_log_file=/var/log/mysql-query.log
# 重启生效
systemctl restart mysqld
```

查看配置：

```mysql
mysql> show global variables like'%general_log%';
+------------------+--------------------------+
| Variable_name    | Value                    |
+------------------+--------------------------+
| general_log      | ON                       |
| general_log_file | /var/log/mysql-query.log |
+------------------+--------------------------+
2 rows in set (0.01 sec)
```

查看日志：

```shell
[root@iZ2ze8o3e6gtnilf7twmf1Z ~]# tail -f /var/log/mysql-query.log 
2022-10-18T14:04:25.973092Z         2 Field List        user 
2022-10-18T14:04:25.974630Z         2 Query     show global status like '%Slow_queries%'
2022-10-18T14:04:30.585753Z         2 Query     show global status like '%Slow_queries%'
2022-10-18T14:04:32.232933Z         2 Query     show variables like '%slow_query%'
2022-10-18T14:04:33.187046Z         2 Query     show variables like 'long_query_time%'
2022-10-18T14:04:56.895972Z         2 Query     select sleep(2)
2022-10-18T14:05:09.171403Z         2 Query     show global status like '%Slow_queries%'
```



### 2.4 MySQL 慢查询日志

- [官方文档 The Slow Query Log](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)

- 默认关闭
- 参数配置
  - slow_query_log 慢查询日志开关
    - 默认为 OFF
  - slow_query_log_file 慢查询日志文件路径
    - 默认为 机器名-slow.log
  - long_query_time 慢查询阈值，SQL执行时间超过该值，则会记录到慢查询日志中，SQL的执行耗时不包含锁等待时间
    - 默认为 10
  -  log_queries_not_using_indexes 将没有使用索引的SQL记录到慢查询日志
    - 默认值 OFF

修改配置：

```shell
# 编辑配置文件
vim /etc/my.cnf

# 在 [mysqld] 下添加或改动参数
slow_query_log=ON
long_query_time=1
slow_query_log_file=slow_query_log.log

# 重启生效
systemctl restart mysqld

```

查看配置：

```mysql
mysql> show variables like '%slow_query%';
+---------------------+--------------------+
| Variable_name       | Value              |
+---------------------+--------------------+
| slow_query_log      | ON                 |
| slow_query_log_file | slow_query_log.log |
+---------------------+--------------------+
2 rows in set (0.00 sec)

mysql> show variables like 'long_query_time%';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
1 row in set (0.00 sec)
```

查看慢查询数量：

```mysql
mysql> show global status like '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 0     |
+---------------+-------+
1 row in set (0.01 sec)

mysql> select sleep(2);
+----------+
| sleep(2) |
+----------+
|        0 |
+----------+
1 row in set (2.01 sec)

mysql> show global status like '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 1     |
+---------------+-------+
1 row in set (0.00 sec)
```

查看慢查询日志内容：

```shell
[root@iZ2ze8o3e6gtnilf7twmf1Z ~]# tail /var/lib/mysql/slow_query_log.log 
/usr/sbin/mysqld, Version: 5.7.40-log (MySQL Community Server (GPL)). started with:
Tcp port: 0  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 2022-10-18T14:04:58.896187Z
# User@Host: root[root] @ localhost []  Id:     2
# Query_time: 2.000310  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
use mysql;
SET timestamp=1666101898;
select sleep(2);
```

使用 pt-query-digest 分析慢查询日志：

```shell
[root@iZ2ze8o3e6gtnilf7twmf1Z ~]# pt-query-digest /var/lib/mysql/slow_query_log.log

# 170ms user time, 30ms system time, 25.73M rss, 220.39M vsz
# Current date: Wed Oct 19 14:24:24 2022
# Hostname: iZ2ze8o3e6gtnilf7twmf1Z
# Files: /var/lib/mysql/slow_query_log.log
# Overall: 1 total, 1 unique, 0 QPS, 0x concurrency ______________________
# Time range: all events occurred at 2022-10-18T14:04:58
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time             2s      2s      2s      2s      2s       0      2s
# Lock time              0       0       0       0       0       0       0
# Rows sent              1       1       1       1       1       0       1
# Rows examine           0       0       0       0       0       0       0
# Query size            15      15      15      15      15       0      15

# Profile
# Rank Query ID                            Response time Calls R/Call V/M 
# ==== =================================== ============= ===== ====== ====
#    1 0x59A74D08D407B5EDF9A57DD5A41825CA  2.0003 100.0%     1 2.0003  0.00 SELECT

# Query 1: 0 QPS, 0x concurrency, ID 0x59A74D08D407B5EDF9A57DD5A41825CA at byte 0
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: all events occurred at 2022-10-18T14:04:58
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count        100       1
# Exec time    100      2s      2s      2s      2s      2s       0      2s
# Lock time      0       0       0       0       0       0       0       0
# Rows sent    100       1       1       1       1       1       0       1
# Rows examine   0       0       0       0       0       0       0       0
# Query size   100      15      15      15      15      15       0      15
# String:
# Databases    mysql
# Hosts        localhost
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms
#    1s  ################################################################
#  10s+
# EXPLAIN /*!50100 PARTITIONS*/
select sleep(2)\G
```

## 3 MySQL 数据文件

- ibdata文件:使用系统表空间存储表数据和索引信息，所有表共同使用一个或者多个ibdata文件
- InnoDB存储引擎的数据文件
  - .frm文件:主要存放与表相关的数据信息，主要包括表结构的定义信息 
  - .ibd:使用独享表空间存储表数据和索引信息，一张表对应一个ibd文件。
- MyISAM存储引擎的数据文件:
  - .frm文件:主要存放与表相关的数据信息，主要包括表结构的定义信息 
  - .myd文件:主要用来存储表数据信息。
  - .myi文件:主要用来存储表数据文件中任何索引的数据树。

查看配置：

```mysql
mysql> show variables like '%datadir%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.00 sec)
```

