[官方文档 The InnoDB Storage Engine](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)

除非需要用到某些InnoDB不具备的特性，并且没有其他办法可以替代，否则都应该选择InnoDB引擎

## 架构图 5.7

![InnoDB architecture diagram showing in-memory and on-disk structures.](assets/innodb-architecture-5-7.png)

## 架构图 8.0

![InnoDB architecture diagram showing in-memory and on-disk structures. In-memory structures include the buffer pool, adaptive hash index, change buffer, and log buffer. On-disk structures include tablespaces, redo logs, and doublewrite buffer files.](assets/innodb-architecture-8-0.png)

## 结构概览 以 5.7 为例

- 内存结构 In-Memory Structures
  - 缓冲池 Buffer Pool
  - 修改缓冲 Change Buffer
  - 自适应哈希索引 Adaptive Hash
  - 日志缓冲 Log Buffer
- 磁盘结构 On-Disk Structures
  - 系统表空间 
  - 独立表空间
  - 通用表空间
  - 临时表空间