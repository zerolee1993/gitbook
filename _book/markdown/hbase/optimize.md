# HBase 优化策略

## 什么导致 HBase 性能下降

- JVM 内存分配及 GC 回收策略
- 与 HBase 运行机制相关的部分配置不合理
- 表结构设计及使用方式不合理

## HBase 概念

- HBase 写入时，MemStore 达到一定的大小是会 flush 到 HFile

- HFile 小文件太多时会执行 compact 操作进行合并

  - minor compaction 选取小的、相邻的 StoreFile 合并成一个大的 StoreFile

  - major compaction 将所有 StoreFile 合并成一个 StoreFile，清理失效数据(被删除的数据、TTL 过期数据、版本号超过设定阈值的数据)

  - Compact 何时进行

    - MemStore flush 到 HFile 时
    - 用户执行 shell 命令 compact、major_compact 或调用相关 API 时

    - HBase 后台线程周期性触发检查

- 当 Region 的大小达到某一阈值之后，会执行 split 操作，切分成两个 Region

## 服务端配置优化

- JVM 设置与 GC 设置
  
- 尽量将 RegionServer 内存划分大一些，避免频繁 FullGC
  
- hbase-site.xml 属性配置

  - `hbase.regionserver.handler.count` rpc 请求的线程数量，默认为 10，提升此配置，有利于 RegionServer 接受更多的请求，但设置过多可能导致内存占用大，频繁 FullGC，甚至出现内存溢出
  - `hbase.hregion.max.filesize` 当 region 的大小大于设定值后，hbase 就会开始 split，默认为10G，可根据存储内容适当调整
  - `hbase.hregion.majorcompaction` major compaction 的执行周期，默认为 1 天，建议设置为 0，禁止自动执行，生产环境低峰期手动执行或脚本执行，不影响正常业务
  - `hbase.hstore.compaction.min` 一个 store 中的 StoreFile 总数超过此值，则触发默认的 compact 操作，默认 3
  - `hbase.hstore.compaction.max` 一次最多合并多少个 StoreFile，如果 StoreFile 较大，应该适当减小此值，避免合并时内存溢出
  - `hbase.hstore.blockingStoreFiles` 一个 Region 中的 Store 内超过 xx 个 StoreFile 时，则 block 所有的写请求进行 compact
  - `hfile.block.cache.size` RegionServer 的 block cache 的内存大小限制
  - `hbase.hregion.memstore.flush.size` memstore 超过此值会被 flush
  - `hbase.hregion.memstore.block.multiplier` 如果 memstore 的内存大小超过 flush.size * multiplier 的值，会 block 该 memstore 的写操作

  > 总体策略：避免自动的 major compaction 和 split 操作，通过开发脚本在业务低峰时进行手动触发



## 常用优化策略(建表、RowKey 设计等)

- 预先分区
  - 创建 HBase 表时默认在一个 RegionServer 上创建一个 Region，写数据时，导致该 Region 达到一定大小是，进行 split 操作，分为连个 Region，进行负载均衡，split 十分耗时，且会造成 Region 无法访问
  - 创建 HBase 表时预先创建一些空的 Regions，并且规定每个 Region 存储的 RowKey 的范围
  - 通过预先分区还可解决数据倾斜的问题
- RowKey 优化
  - 利用 HBase 默认排序的特点，将一起访问的数据放到一起
  - 防止热点问题，避免使用时序或单调的递增递减
- Column 优化
  - 列族的名称和列的描述名称尽量简短
  - 同一张表的 ColumnFamily 不要超过 3 个
- Schema 优化
  - 宽表：列多行少
  - 高表：列少行多

## HBase 读写性能优化(Java API 开发过程)

- 写优化策略
  - 同步批量提交 or 异步批量提交，异步可能会丢失数据，在业务允许的情况下，开启异步提交以提高性能
  - 关闭 WAL 或采用异步 WAL，损失数据完整性，提高吞吐量

- 读优化策略
  - 客户端：Scan 缓存设置，批量获取
  - 服务端：BlockCache 配置是否合理，HFile 是否过多
  - 表结构设计问题