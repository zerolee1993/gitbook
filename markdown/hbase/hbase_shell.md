- 查看表目录

  ```shell
  hbase(main):006:0> list
  TABLE
  0 row(s) in 0.0150 seconds
  => []
  ```

- 创建表

  ```shell
  hbase(main):007:0> create 'FileTable','fileInfo','saveInfo'
  0 row(s) in 1.3810 seconds
  => Hbase::Table - FileTable
  
  hbase(main):008:0> list
  TABLE
  FileTable
  1 row(s) in 0.0120 seconds
  => ["FileTable"]
  
  hbase(main):009:0> desc 'FileTable'
  Table FileTable is ENABLED
  FileTable
  COLUMN FAMILIES DESCRIPTION
  {NAME => 'fileInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  {NAME => 'saveInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  2 row(s) in 0.0350 seconds
  ```

- 增加列族

  ```shell
  hbase(main):010:0> alter 'FileTable','cf'
  Updating all regions with the new schema...
  1/1 regions updated.
  Done.
  0 row(s) in 1.9260 seconds
  
  hbase(main):011:0> desc 'FileTable'
  Table FileTable is ENABLED
  FileTable
  COLUMN FAMILIES DESCRIPTION
  {NAME => 'cf', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_
  ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '
  65536', REPLICATION_SCOPE => '0'}
  {NAME => 'fileInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  {NAME => 'saveInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  3 row(s) in 0.0180 seconds
  ```

- 删除列族

  ```shell
  hbase(main):013:0> alter 'FileTable',{NAME=>'cf',METHOD=>'delete'}
  Updating all regions with the new schema...
  1/1 regions updated.
  Done.
  0 row(s) in 2.1460 seconds
  
  hbase(main):015:0> desc 'FileTable'
  Table FileTable is ENABLED
  FileTable
  COLUMN FAMILIES DESCRIPTION
  {NAME => 'fileInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  {NAME => 'saveInfo', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
  BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZ
  E => '65536', REPLICATION_SCOPE => '0'}
  2 row(s) in 0.0180 seconds
  ```

- 增加两条记录

  ```shell
  hbase(main):016:0> put 'FileTable','rowkey1','fileInfo:name','file1.txt'
  0 row(s) in 0.0640 seconds
  
  hbase(main):017:0> put 'FileTable','rowkey1','fileInfo:type','txt'
  0 row(s) in 0.0030 seconds
  
  hbase(main):018:0> put 'FileTable','rowkey1','fileInfo:size','1024'
  0 row(s) in 0.0060 seconds
  
  hbase(main):019:0> put 'FileTable','rowkey1','saveInfo:path','/home'
  0 row(s) in 0.0110 seconds
  
  hbase(main):020:0> put 'FileTable','rowkey1','saveInfo:creator','tom'
  0 row(s) in 0.0030 seconds
  
  hbase(main):021:0> put 'FileTable','rowkey2','fileInfo:name','file2.txt'
  0 row(s) in 0.0030 seconds
  
  hbase(main):022:0> put 'FileTable','rowkey2','fileInfo:type','txt'
  0 row(s) in 0.0030 seconds
  
  hbase(main):023:0> put 'FileTable','rowkey2','fileInfo:size','2048'
  0 row(s) in 0.0030 seconds
  
  hbase(main):024:0> put 'FileTable','rowkey2','saveInfo:path','/home'
  0 row(s) in 0.0040 seconds
  
  hbase(main):025:0> put 'FileTable','rowkey2','saveInfo:creator','jack'
  0 row(s) in 0.0040 seconds
  ```

- 查看表中的记录条数

  ```shell
  hbase(main):026:0> count 'FileTable'
  2 row(s) in 0.0240 seconds
  => 2
  ```

- 查看一行数据

  ```shell
  hbase(main):027:0> get 'FileTable','rowkey1'
  COLUMN                         CELL
   fileInfo:name                 timestamp=1562049913828, value=file1.txt
   fileInfo:size                 timestamp=1562049931731, value=1024
   fileInfo:type                 timestamp=1562049924426, value=txt
   saveInfo:creator              timestamp=1562049958992, value=tom
   saveInfo:path                 timestamp=1562049946815, value=/home
  1 row(s) in 0.0170 seconds
  ```

- 查看一行指定列族数据

  ```shell
  hbase(main):028:0>  get 'FileTable','rowkey1','fileInfo'
  COLUMN                         CELL
   fileInfo:name                 timestamp=1562049913828, value=file1.txt
   fileInfo:size                 timestamp=1562049931731, value=1024
   fileInfo:type                 timestamp=1562049924426, value=txt
  1 row(s) in 0.0170 seconds
  ```

- 查询表中数据

  ```shell
  hbase(main):001:0> scan 'FileTable'
  ROW                            COLUMN+CELL
   rowkey1                       column=fileInfo:name, timestamp=1562049913828, value=file1.txt
   rowkey1                       column=fileInfo:size, timestamp=1562049931731, value=1024
   rowkey1                       column=fileInfo:type, timestamp=1562049924426, value=txt
   rowkey1                       column=saveInfo:creator, timestamp=1562049958992, value=tom
   rowkey1                       column=saveInfo:path, timestamp=1562049946815, value=/home
   rowkey2                       column=fileInfo:name, timestamp=1562049974675, value=file2.txt
   rowkey2                       column=fileInfo:size, timestamp=1562049998454, value=2048
   rowkey2                       column=fileInfo:type, timestamp=1562049983426, value=txt
   rowkey2                       column=saveInfo:creator, timestamp=1562050020654, value=jack
   rowkey2                       column=saveInfo:path, timestamp=1562050007276, value=/home
  2 row(s) in 0.1720 seconds
  
  hbase(main):002:0> scan 'FileTable',{COLUMN=>'fileInfo:name'}
  ROW                            COLUMN+CELL
   rowkey1                       column=fileInfo:name, timestamp=1562049913828, value=file1.txt
   rowkey2                       column=fileInfo:name, timestamp=1562049974675, value=file2.txt
  2 row(s) in 0.0130 seconds
  
  hbase(main):004:0> scan 'FileTable',{STARTROW=>'rowkey1',LIMIT=>1,VERSIONS=>1}
  ROW                            COLUMN+CELL
   rowkey1                       column=fileInfo:name, timestamp=1562049913828, value=file1.txt
   rowkey1                       column=fileInfo:size, timestamp=1562049931731, value=1024
   rowkey1                       column=fileInfo:type, timestamp=1562049924426, value=txt
   rowkey1                       column=saveInfo:creator, timestamp=1562049958992, value=tom
   rowkey1                       column=saveInfo:path, timestamp=1562049946815, value=/home
  1 row(s) in 0.0100 seconds
  ```

- 删除一列数据

  ```shell
  hbase(main):005:0> delete 'FileTable','rowkey1','fileInfo:size'
  0 row(s) in 0.0450 seconds
  
  hbase(main):006:0> get 'FileTable','rowkey1'
  COLUMN                         CELL
   fileInfo:name                 timestamp=1562049913828, value=file1.txt
   fileInfo:type                 timestamp=1562049924426, value=txt
   saveInfo:creator              timestamp=1562049958992, value=tom
   saveInfo:path                 timestamp=1562049946815, value=/home
  1 row(s) in 0.0360 seconds
  ```

- 删除一行数据

  ```shell
  hbase(main):007:0> deleteall 'FileTable','rowkey1'
  0 row(s) in 0.0030 seconds
  
  hbase(main):008:0> get 'FileTable','rowkey1'
  COLUMN                         CELL
  0 row(s) in 0.0020 seconds
  ```

- 删除表，需要先禁用

  ```shell
  hbase(main):009:0> disable 'FileTable'
  0 row(s) in 2.3310 seconds
  
  hbase(main):010:0> is_enabled 'FileTable'
  false
  0 row(s) in 0.0050 seconds
  
  hbase(main):012:0> is_disabled 'FileTable'
  true
  0 row(s) in 0.0050 seconds
  
  hbase(main):013:0> drop 'FileTable'
  0 row(s) in 1.2600 seconds
  
  hbase(main):015:0> exists 'FileTable'
  Table FileTable does not exist
  0 row(s) in 0.0040 seconds
  
  hbase(main):017:0> list
  TABLE
  0 row(s) in 0.0170 seconds
  => []
  ```

