## 信息统计

<!--sec data-title="1.表数据量统计" data-id="a1" data-show=true data-collapse=true ces-->
- 需要更改 scheme_name 为自己需要统计的数据库名
```sql
SELECT
	TABLE_NAME 表名,
	TABLE_COMMENT 注释,
	TABLE_ROWS 数据量,
	INDEX_LENGTH 索引长度,
	TABLE_COLLATION 编码集,
	AVG_ROW_LENGTH 平均行长度,
	DATA_LENGTH 数据长度,
	MAX_DATA_LENGTH 最大数据长度,
	DATA_FREE 空间碎片,
	CREATE_TIME 创建时间
FROM
	information_schema.`TABLES`
WHERE TABLE_SCHEMA='scheme_name'
ORDER BY TABLE_ROWS DESC;
```
<!--endsec-->

<!--sec data-title="2.表字段信息统计" data-id="a2" data-show=true data-collapse=true ces-->
- 需要更改 scheme_name 为自己需要统计的数据库名
- 默认过滤备份表及临时表，可自行修改 sql 通过 table_name 限制查询的信息
```sql
SELECT
CONCAT(c.TABLE_NAME,'(',t.TABLE_COMMENT,')') as 表,
c.COLUMN_NAME as 列,
c.COLUMN_COMMENT 注释,
c.COLUMN_TYPE 数据类型,
c.IS_NULLABLE as 是否可为空,
c.COLUMN_DEFAULT as 默认值,
c.EXTRA as 额外信息
FROM
INFORMATION_SCHEMA.COLUMNS c LEFT JOIN INFORMATION_SCHEMA.TABLES t
ON c.TABLE_NAME = t.TABLE_NAME
where c.table_schema ='scheme_name' and c.TABLE_NAME not like '%bak%' and c.TABLE_NAME not like '%tmp_%'
order by c.TABLE_NAME,c.ORDINAL_POSITION
```
<!--endsec-->

## 性能监控

<!--sec data-title="1.查看实时连接" data-id="b1" data-show=true data-collapse=true ces-->
```sql
SHOW FULL PROCESSLIST;
```
详细说明参考[实时连接监控和管理](../practice/process-manage.md)
<!--endsec-->