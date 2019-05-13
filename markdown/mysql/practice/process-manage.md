## 应用场景
1. 排查 MySQL 相关问题时，需要查看当前到数据库的连接及状态
2. 查看实时的慢查询
3. 执行某查询后，手动终止此查询

## 如何查看 MySQL 实时连接

可使用如下命令：

```sql
SHOW PROCESSLIST;
```

若执行的 SQL 较长导致查询出的 Info 被截取，可使用如下命令查看全部信息：

```
SHOW FULL PROCESSLIST;
```

该命令等同于如下命令：

```
SELECT * FROM infomation_scheme.PROCESSLIST
```

实际上就是查询系统表 infomation_scheme.PROCESSLIST

## 查询结果说明

`SHOW PROCESSLIST;`查询出的信息如下：

管理员权限可查看全部，其他用户只能查看当前用户的连接

![](/assets/images/markdown/practice/process-manage/1.png)

关于 PROCESSLIST 表的字段说明如下：可参考[官方文档](<https://dev.mysql.com/doc/refman/5.6/en/processlist-table.html>)

| 字段    | 解释                                                         |
| ------- | ------------------------------------------------------------ |
| id      | 连接的唯一标识<br/>等同于`SHOW PROCESSLIST;`命令结果中的 ID 列<br/>等同于 performance_schema.threads 表的 processlist_id<br/>等同于函数 `CONNECTION_ID();`的返回结果 |
| user    | 连接的用户                                                   |
| host    | 主机信息                                                     |
| db      | 连接的数据库                                                 |
| command | 命令类型                                                     |
| time    | 时长，单位：秒                                               |
| state   | 状态                                                         |
| info    | 执行的语句                                                   |

## 如何结束进行中的连接

可以使用 `KILL processlist_id` 命令结束指定连接，参考[官方文档](https://dev.mysql.com/doc/refman/5.6/en/kill.html)

processlist_id 就是 `SHOW PROCESSLIST` 查询结果中的 Id 列