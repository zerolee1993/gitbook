---

---

# 搭建 Hadoop 环境

## 1. 下载 hadoop

```shell
wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz
tar -zxvf hadoop-3.1.2.tar.gz
```

## 2. 配置 hdfs-site.xml

```shell
vi hadoop-3.1.2/etc/hadoop/hdfs-site.xml
```

在默认空的 configuration 节点中添加如下内容

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/xxx/hadoop-3.1.2/data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/xxx/hadoop-3.1.2/data/datanode</value>
    </property>
</configuration>
```

| 属性                  | 说明                    |
| --------------------- | ----------------------- |
| dfs.replication       | hdfs 备份个数，默认为 3 |
| dfs.namenode.name.dir | namenode 路径           |
| dfs.datanode.data.dir | datanode 路径           |

## 3. 配置 core-site.xml

```shell
vi hadoop-3.1.2/etc/hadoop/core-site.xml
```

在默认空的 configuration 节点中添加如下内容

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/xxx/hadoop-3.1.2/data/tmp</value>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9999</value>
    </property>
</configuration>
```

| 属性            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| hadoop.tmp.dir  | 默认为系统的 /tmp，避免自动清理导致一些问题，建议指定自己的文件夹 |
| fs.default.name | hdfs 文件系统的访问地址，在 HBase 的配置中需要通过 hbase.rootdir 属性指向这个路径，以便 HBase 使用 hdfs 的文件系统 |

## 4. 配置 jdk 环境变量

```shell
vi hadoop-3.1.2/etc/hadoop/hadoop-env.sh
```

找到文件中如下注释的位置，将 export JAVA_HOME 配置正确，记得确保这条配置不要被注释

```shell
# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
```

## 5. 格式化 hdfs

```shell
cd hadoop-3.1.2/bin
./hadoop namenode -format
```

## 6. 启动和关闭

```shell
cd hadoop-3.1.2/sbin
./start-all.sh
./stop-all.sh
```

## 7. 相关访问地址

1. Resource Manager: [http://localhost:9870](http://localhost:9870/)
2. JobTracker:[ http://localhost:8088](http://localhost:8088/)
3. Specific Node Information:[ http://localhost:8042](http://localhost:8042/)

---

# 搭建 HBase 环境 

## 1. 下载

```shell
wget http://mirror.bit.edu.cn/apache/hbase/2.2.0/hbase-2.2.0-bin.tar.gz
tar -zxvf hbase-2.2.0-bin.tar.gz
```

## 2. 配置 hbase-env.sh

```shell
vi hbase-2.2.0/conf/hbase-env.sh
```

找到文件中如下注释的位置，将 export JAVA_HOME 配置正确，记得确保这条配置不要被注释

```shell
# The java implementation to use.  Java 1.8+ required.
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
```

找到文件中如下属性的位置，设置为 false，系统默认为 true，表示使用 HBase 自带的 Zookeeper，在这里我们使用自建的 Zookeeper，方便排查问题，在之后会演示如何安装对应的 Zookeeper 环境

```shell
# Tell HBase whether it should manage it's own instance of ZooKeeper or not.
export HBASE_MANAGES_ZK=false
```

## 3. 配置 hbase-site.xml

```shell
vi hbase-2.2.0/conf/hbase-site.xml
```

```xml
<configuration>
  	<property>
    		<name>hbase.rootdir</name>
    		<value>hdfs://localhost:9999/hbase</value>
  	</property>
  	<property>
        <name>hbase.tmp.dir</name>
        <value>/xxx/hbase-2.2.0/data/tmp</value>
  	</property>
  	<property>
    		<name>hbase.cluster.distributed</name>
  			<value>true</value>
		</property>
  	<property>
        <name>hbase.zookeeper.quorum</name>
        <value>127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183</value>
  	</property>
  	<!--
		<property>
    		<name>hbase.zookeeper.property.dataDir</name>
    		<value>/Users/lishaofei/Environment/hbase-2.2.0/zookeeper</value>
  	</property>
  	<property>
       <name>zookeeper.znode.parent</name>
       <value>/hbase/master</value>
		</property>
		-->
</configuration>
```

| 属性                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| hbase.rootdir                    | HBase存储文件的地址，使用 Hadoop 中配置的的 HDFS 地址        |
| hbase.tmp.dir                    | 默认为系统的 /tmp，避免自动清理导致一些问题，建议指定自己的文件夹 |
| hbase.cluster.distributed        | 是否开启分布式集群                                           |
| hbase.zookeeper.quorum           | 自建 zookeeper 需设置这个属性，并指定 Zookeeper 的地址       |
| hbase.zookeeper.property.dataDir | 使用内置 Zookeeper 时相关文件的存放目录<br />若使用自建 Zookeeper，这个属性不生效，无需配置<br />等同于自建 Zookeeper 的 cfg 配置中的 dataDir 指定目录 |
| zookeeper.znode.parent           | 在 Zookeeper 中 HBase 相关节点的根目录                       |

> 注释掉的属性不需要配置
>
> 1. hbase.zookeeper.property.dataDir 由于使用了自建 Zookeeper 所以不用配置
>
> 2. 网上一些教程将 zookeeper.znode.parent 设置为 /hbase/master，这样在使用 JavaAPI 的判断表是否存在时将抛出如下异常，原因是默认的 HBase 会在 Zookeeper 下建立节点 /hbase/table/hbase:meta，但是当日加入上述配置时，这个节点就建立为 /hbase/master/table/hbase:meta 导致找不到对应的表元数据
>
>     `No meta znode available, details=row '123' on table 'hbase:meta' at null` 

## 4. 启动，关闭

```shell
cd hbase-2.2.0/bin
./start-hbase.sh
./stop-hbase.sh
```

## 5. 进入 Hbase Shell 并查看状态

```shell
cd hbase-2.2.0/bin
./hbase shell
hbase(main):001:0> status
1 active master, 0 backup masters, 1 servers, 0 dead, 2.0000 average load
Took 0.4010 seconds
```

## 6. 问题处理

- 出现如下警告请直接忽略

```shell
WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

- 在 HBase Shell 命令行中执行 status 出现如下错误，到 hadoop/bin 目录下执行 `./hadoop dfsadmin -safemode leave`

```shell
ERROR: org.apache.hadoop.hbase.ipc.ServerNotRunningYetException: Server is not running yet
```



- 如下三个问题
  
  - HBase 启动后 HMaster 节点自己挂掉
  
- 无法使用 Stop 关闭 HBase，jps 查看会留下 HRegionServer 节点没有被关闭
  
  - 进入 Hbase Shell 执行命令报错
  
    `ERROR: KeeperErrorCode = NoNode for /hbase/master`；
  
- 解决方案
  - 进入 HBase 目录下的 logs 文件夹下，查看 master.log 相关日志定位问题（出现任何问题先查这个）
  - 如果是配置问题，请在更改配置后，重启前，删除 Zookeeper 指定的 dataDir 目录下的 version-2 文件夹、删除 HBase 指定的 tmp 目录的内容再重启
  - 还是解决不了，则：关闭所有 java 进程、清理 Hadoop tmp 文件下的内容、重新格式化HDFS、清理 HBase tmp 目录的内容、删除 Zookeeper 指定的 dataDir 目录下的 version-2 文件夹、删除 HBase 目录下的 logs 文件夹中的所有日志，重启，并观察master.log

---

# 自建 zookeeper 伪分布式集群

## 1. 下载

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz
tar -zxvf apache-zookeeper-3.5.5-bin.tar.gz
```

## 2. 伪分布式配置

- 复制三份配置文件

```shell
cd apache-zookeeper-3.5.5-bin/conf
cp zoo_sample.cfg zoo1.cfg
cp zoo_sample.cfg zoo2.cfg
cp zoo_sample.cfg zoo3.cfg
```

- 修改 zoo1.cfg 的如下两个属性

```shell
dataDir=/xxx/apache-zookeeper-3.5.5-bin/data/zk1
clientPort=2181
```

- 修改 zoo2.cfg 的如下两个属性

```shell
dataDir=/xxx/apache-zookeeper-3.5.5-bin/data/zk2
clientPort=2182
```

- 修改 zoo3.cfg 的如下两个属性

```shell
dataDir=/xxx/apache-zookeeper-3.5.5-bin/data/zk3
clientPort=2183
```

- 在三个配置文件末尾均加入如下内容

```shell
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890
```

## 3. 启动和关闭

- 创建启动/关闭脚本，可以手动一个一个启动，这里只是为了方便操作

```shell
cd apache-zookeeper-3.5.5-bin
# 创建一键启动脚本
vi start-all.sh
# 粘贴如下内容并保存
bin/zkServer.sh start conf/zoo1.cfg
bin/zkServer.sh start conf/zoo2.cfg
bin/zkServer.sh start conf/zoo3.cfg
# 保存后赋权
chmod 777 start-all.sh
# 创建一键关闭脚本
vi stop-all.sh
# 粘贴如下内容并保存
bin/zkServer.sh stop conf/zoo1.cfg
bin/zkServer.sh stop conf/zoo2.cfg
bin/zkServer.sh stop conf/zoo3.cfg
# 保存后赋权
chmod 777 stop-all.sh
```

- 启动/关闭

```shell
cd apache-zookeeper-3.5.5-bin
./start-all.sh
./stop-all.sh
```

