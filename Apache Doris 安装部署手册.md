## Apache Doris 安装部署手册

### 1. 下载

下载地址：http://doris.apache.org/master/zh-CN/downloads/downloads.html

<img src="/Users/superzhang/Library/Application Support/typora-user-images/image-20210904011703306.png" alt="image-20210904011703306" style="zoom:50%;" align="left"/>

我们选择最新版本`0.14.0`下载源码包

### 2. 编译

官方推荐使用 Docker 开发镜像编译

###### 注：不要用自己的macbook安装docker去编译，亲测坑很多，最终失败！

#### 2.1 安装docker

登录服务器安装，安装过程自行百度！

#### 2.2 下载镜像

###### 注: 针对不同的 Doris 版本，需要下载对应的镜像版本,doris 0.14.0 版本仍然使用apache/incubator-doris:build-env-1.2 编译，之后的代码将使用apache/incubator-doris:build-env-1.3.1。

<img src="/Users/superzhang/Library/Application Support/typora-user-images/image-20210904013110243.png" alt="image-20210904013110243" style="zoom:50%;" align="left"/>

我们选择的是`0.14.0`版本，因此要选择`apache/incubator-doris:build-env-1.2`的镜像

**示例**：

登录服务器

```
 ssh root@172.22.5.76
```

拉取镜像

```
docker pull apache/incubator-doris:build-env-1.2
```

#### 2.3 运行镜像

查看镜像

```
docker images
```

![image-20210904015427406](/Users/superzhang/Library/Application Support/typora-user-images/image-20210904015427406.png)

运行镜像

```
docker run -itv /opt/dockerpace:/app  -p 80:80 apache/incubator-doris:build-env-1.2 /bin/bash  
```

###### 释义：挂载目录，映射端口，指定镜像名称和版本，创建并启动进入容器。

#### 2.4 上传

进入容器后查看列表，`/app`则为共享目录，在宿主机端将源码包上传至` /opt/dockerpace` 目录，则在容器中 `/app` 可见。

<img src="/Users/superzhang/Library/Application Support/typora-user-images/image-20210904020752580.png" alt="image-20210904020752580"  align="left" />

#### 2.5 解压

```
tar -vxf apache-doris-0.14.0-incubating-src.tar.gz
```

#### 2.6 开始编译

```
cd apache-doris-0.14.0-incubating-src
sh build.sh
```

#### 2.7 编译成功

可以看到success 并且生成output目录

![image-20210915170402203](/Users/superzhang/Library/Application Support/typora-user-images/image-20210915170402203.png)

![image-20210915170605071](/Users/superzhang/Library/Application Support/typora-user-images/image-20210915170605071.png)

低版本可能会出现错误：

![image-20210904021911303](/Users/superzhang/Library/Application Support/typora-user-images/image-20210904021911303.png)

需要做如下操作：

进入fe目录，修改pom.xml

```
cd /app/apache-doris-0.12.0-incubating-src/fe
```

```
<repository>
                            <id>cloudera-thirdparty</id>
                             <url>https://repository.cloudera.com/content/repositories/third-party/</url>
                            // **将上面两行配置替换为下面两行**
                            <id>cloudera-public</id>
                            <url>https://repository.cloudera.com/artifactory/public/</url>
  </repository>
```

保存并继续编译 `sh build.sh`

##### 

#### 2.8 编译broker

```
cd /app/apache-doris-0.14.0-incubating-src/fs_brokers/apache_hdfs_broker
sh build.sh
```

![image-20210915170926126](/Users/superzhang/Library/Application Support/typora-user-images/image-20210915170926126.png)

编译成功！

![image-20210915170829258](/Users/superzhang/Library/Application Support/typora-user-images/image-20210915170829258.png)

### 3.部署

服务在节点的分布：

| 部署节点         | FE   | BE   | observer | broker |
| ---------------- | ---- | ---- | -------- | ------ |
| taia-3.novalocal | √    |      |          | √      |
| taia-4.novalocal |      | √    | √        | √      |
| taia-5.novalocal |      | √    |          | √      |
| taia-6.novalocal |      | √    |          | √      |

#### 3.1 文件分发

1.分别在`taia-3.novalocal`，`taia-4.novalocal`，`taia-5.novalocal`，`taia-6.novalocal` 的**/opt** 目录下新建 **doris**目录

2.编译后的文件上传至 `taia-3.novalocal` 下边的 **/opt**目录

```
cd /opt/apache-doris-0.14.0-incubating-src/output
```

![image-20210916150036965](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916150036965.png)

3.分发 **fe**  至`taia-3.novalocal`，`taia-4.novalocal`的**/opt/doris**

4.分发 **be**至`taia-4.novalocal`，`taia-5.novalocal`，`taia-6.novalocal`的**/opt/doris**

5.broker 的分发：

```
 cd /opt/apache-doris-0.14.0-incubating-src/fs_brokers/apache_hdfs_broker/output/
```

分发 **apache_hdfs_broker**至`taia-3.novalocal`，`taia-4.novalocal`,`taia-4.novalocal`，`taia-6.novalocal`的**/opt/doris**

完成分发！！！

#### 3.2 fe的安装（master）

 **follower**，**master**，**observer** 只是 **fe**的角色，都需要启动**fe**，然后由**mater**分配角色

follower和master只是选举的结果，先启动的角色就是master，在此基础上，可以添加若干 Follower 和 Observer。

1.修改配置文件呢

```
cd /opt/doris/fe/conf
vim fe.conf
```

![image-20210916152100673](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916152100673.png)

如图：配置文件中1的位置默认注释，放开即可，**元数据的存储位置，可以自定义**。

配置文件中2的位置为新增，配置本机的ip，**一定要配置**！！！

2.创建 **/opt/doris-meta**目录

3.启动fe

```
sh bin/start_fe.sh --daemon
```

4.查看日志

###### 注意：启动失败往往是因为端口被占用，修改配置文件中的端口

```
tail -f /opt/doris/fe/log/fe.out
```

5.测试连接

登录mysql

```
mysql -h 172.22.5.13 -P 9030 -uroot
```

连接成功后，执行 

```
SHOW PROC '/frontends';
```

![image-20210916154919027](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916154919027.png)

fe部署完成！！！

#### 3.3 observer的安装

observer的fe配置和master的一样，配置文件中`priority_networks`要配置本机的就行；

配置完成后启动：

```
./bin/start_fe.sh --helper taia-3.novalocal:9010 --daemon

加入到集群中并分配角色
ALTER SYSTEM ADD OBSERVER "taia-4.novalocal:9010"

```

master启动后，follower和observer启动都需要指定 master(或者其他正常的fe)的ip和端口号

查看

```

ALTER SYSTEM ADD OBSERVER "172.22.5.14:9010"

SHOW PROC '/frontends';
```

#### 3.4 be的安装

1.修改配置文件

```
cd /opt/doris/be/conf
vim be.conf
```

![image-20210916172830681](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916172830681.png)

开放注释，配置本机的ip

2.新建目录 **/opt/doris/be/storage**

3.启动

```
sh bin/start_be.sh --daemon
```

三台都要启动

4.节点添加到集群

```
ALTER SYSTEM ADD BACKEND "172.22.5.14:9050";
ALTER SYSTEM ADD BACKEND "172.22.5.15:9050";
ALTER SYSTEM ADD BACKEND "172.22.5.16:9050";
```

5.查看状态

```
SHOW PROC '/backends';
```

![image-20210916174013024](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916174013024.png)

#### 3.5 broker的安装

1.在 **/opt/doris/apache_hdfs_broker/conf **中放入hadoop的配置文件

![image-20210916180418702](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916180418702.png)

2.启动broker

```
sh bin/start_broker.sh --daem
```

3.加入到集中

```
 ALTER SYSTEM ADD BROKER broker1 "taia-3.novalocal:8000";
 ALTER SYSTEM ADD BROKER broker2 "taia-4.novalocal:8000";
 ALTER SYSTEM ADD BROKER broker3 "taia-5.novalocal:8000";
 ALTER SYSTEM ADD BROKER broker4 "taia-6.novalocal:8000";
```

4.查看broker节点状态

```
 SHOW PROC '/brokers';
```

![image-20210916180651756](/Users/superzhang/Library/Application Support/typora-user-images/image-20210916180651756.png)

5.部署完毕！！

### 4 使用

#### 4.1 数据模型

1、Duplicate 模型

```
CREATE TABLE IF NOT EXISTS example_db.testry

(

    `rksj` VARCHAR(50)  COMMENT "入库时间",

    `name` VARCHAR(50)  COMMENT "姓名",

    `sfzhm` VARCHAR(50) COMMENT "错误码",

    `jqdz` VARCHAR(500) COMMENT "地址",

    `id` VARCHAR(50) COMMENT "主键id"

)

DUPLICATE KEY(`rksj`, `name`) //随便指定前几列

DISTRIBUTED BY HASH(rksj) BUCKETS 10

PROPERTIES("replication_num" = "1");
```

2、Uniq 模型

Uniq 模型

联合唯一所以去重，索引相同，新数据覆盖旧数据，可以设置多个键为联合唯一索引，主键一定要放在第一列，注意顺序！

```
CREATE TABLE example_db. tt(

  `sfzhm` varchar(50) NULL COMMENT "错误码",

  `name` varchar(50) NULL COMMENT "姓名",

  `rksj` varchar(50) NULL COMMENT "入库时间",

  `jqdz` varchar(500) NULL COMMENT "地址",

  `ids` varchar(50) NULL COMMENT "主键ids"

) ENGINE=OLAP

UNIQUE KEY(`sfzhm`)

COMMENT "OLAP"

DISTRIBUTED BY HASH(`sfzhm`) BUCKETS 10

PROPERTIES ("replication_num" = "1");  
```

3、Aggregate 模型

```text
CREATE TABLE IF NOT EXISTS example_db.expamle_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)

DISTRIBUTED BY HASH(user_id) BUCKETS 10

PROPERTIES("replication_num" = "1");
```

#### 4.2 rollup

ROLLUP 在多维分析中是“上卷”的意思，即将数据按某种指定的粒度进行进一步聚合。

在 Base 表之上，我们可以创建任意多个 ROLLUP 表。这些 ROLLUP 的数据是基于 Base 表产生的，并且在物理上是**独立存储**的。

```
alter table example_db.expamle_tbl add rollup rollup_userid(user_id,cost);
```

当执行：

```
select user_id,cost from example_db.expamle_tbl
```

实际命中的事rollup表。

#### 4.3 索引

##### 4.3.1前缀索引

Doris 会把 Base/Rollup 表中的前 36 个字节（有 varchar 类型则可能导致前缀索引不满 36 个字节，varchar 会截断前缀索引，并且最多使用 varchar 的 20 个字节）

```text
       -----> 从左到右匹配
+----+----+----+----+----+----+
| c1 | c2 | c3 | c4 | c5 |... |
```

##### 4.3.2 bitmap索引

bitmap index：位图索引，是一种快速数据结构，能够加快查询速度

1、创建索引（只能单列）

```
CREATE INDEX index_name ON testindex (id) USING BITMAP COMMENT 'idindex';
```

2、查看索引

```
  SHOW INDEX FROM example_db.table_name;
```

3、删除索引

```
 DROP INDEX index_name ON example_db.table_name;
```



###### 注：ROLLUP来调整列，前边的列才有机会被创建索引。

###### Aggregate表：只有维度列可以建bitmap索引

###### Uniq表：只有Uniq列可以创建

###### Duplicate表：所有列都可以创建索引



#### 4.4 数据导入

##### 4.4.1 Broker Load

Broker load 是一个异步的导入方式，支持的数据源取决于 Broker 进程支持的数据源。

用户需要通过 MySQL 协议 创建 Broker load 导入，并通过查看导入命令检查导入结果。

```
LOAD LABEL testry_20210997
(
    DATA INFILE("hdfs://172.22.5.12:8020/test/*") 目录之后必须加‘ / ' ，不可以直接写目录
    INTO TABLE test
    FORMAT as "parquet"
)
WITH BROKER broker1 

PROPERTIES
(
    "timeout"="3600"
);
```

##### 4.4.2 Routine Load

当前我们仅支持从 Kafka 系统进行例行导入。

```text
CREATE ROUTINE LOAD db1.job1 on tbl1
PROPERTIES
(
    "desired_concurrent_number"="1"
)
FROM KAFKA
(
    "kafka_broker_list"= "broker1:9091,broker2:9091",
    "kafka_topic" = "my_topic",
    "property.security.protocol" = "ssl",
    "property.ssl.ca.location" = "FILE:ca.pem",
    "property.ssl.certificate.location" = "FILE:client.pem",
    "property.ssl.key.location" = "FILE:client.key",
    "property.ssl.key.password" = "abcdefg"
);
```

##### 4.4.3 Spark Load

Spark load 通过外部的 Spark 资源实现对导入数据的预处理，提高 Doris 大数据量的导入性能并且节省 Doris 集群的计算资源。主要用于初次迁移，大数据量导入 Doris 的场景。

### 基本流程

用户通过 MySQL 客户端提交 Spark 类型导入任务，FE记录元数据并返回用户提交成功。

Spark load 任务的执行主要分为以下5个阶段。

1. FE 调度提交 ETL 任务到 Spark 集群执行。
2. Spark 集群执行 ETL 完成对导入数据的预处理。包括全局字典构建（BITMAP类型）、分区、排序、聚合等。
3. ETL 任务完成后，FE 获取预处理过的每个分片的数据路径，并调度相关的 BE 执行 Push 任务。
4. BE 通过 Broker 读取数据，转化为 Doris 底层存储格式。
5. FE 调度生效版本，完成导入任务。

```
-- yarn cluster 模式
CREATE EXTERNAL RESOURCE "spark0"
PROPERTIES
(
  "type" = "spark",
  "spark.master" = "yarn",
  "spark.submit.deployMode" = "cluster",
  "spark.jars" = "xxx.jar,yyy.jar",
  "spark.files" = "/tmp/aaa,/tmp/bbb",
  "spark.executor.memory" = "1g",
  "spark.yarn.queue" = "queue0",
  "spark.hadoop.yarn.resourcemanager.address" = "127.0.0.1:9999",
  "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
  "working_dir" = "hdfs://127.0.0.1:10000/tmp/doris",
  "broker" = "broker0",
  "broker.username" = "user0",
  "broker.password" = "password0"
);
```

##### 4.4.4 Stream load

Stream load 是一个同步的导入方式，用户通过发送 HTTP 协议发送请求将本地文件或数据流导入到 Doris 中。Stream load 同步执行导入并返回导入结果。用户可直接通过请求的返回体判断本次导入是否成功。

Stream load 主要适用于导入本地文件，或通过程序导入数据流中的数据。

```
curl --location-trusted -u root -T date -H "label:123" http://abc.com:8030/api/test/date/_stream_load
```

##### 4.4.5 S3 Load

从0.14 版本开始，Doris 支持通过S3协议直接从支持S3协议的在线存储系统导入数据。

```
 LOAD LABEL example_db.exmpale_label_1
    (
        DATA INFILE("s3://your_bucket_name/your_file.txt")
        INTO TABLE load_test
        COLUMNS TERMINATED BY ","
    )
    WITH S3
    (
        "AWS_ENDPOINT" = "AWS_ENDPOINT",
        "AWS_ACCESS_KEY" = "AWS_ACCESS_KEY",
        "AWS_SECRET_KEY"="AWS_SECRET_KEY",
        "AWS_REGION" = "AWS_REGION"
    )
    PROPERTIES
    (
        "timeout" = "3600"
    );
```

