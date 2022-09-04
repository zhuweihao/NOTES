# Hive数据类型

## 基本数据类型

| Hive 数据类型 | Java 数据类型 | 长度                                                  | 例子                                  |
| ------------- | ------------- | ----------------------------------------------------- | ------------------------------------- |
| TINYINT       | byte          | 1byte 有符号整数                                      | 20                                    |
| SMALINT       | short         | 2byte 有符号整数                                      | 20                                    |
| ***INT***     | int           | 4byte 有符号整数                                      | 20                                    |
| ***BIGINT***  | long          | 8byte 有符号整数                                      | 20                                    |
| BOOLEAN       | boolean       | 布尔类型，true 或者 false                             | TRUE FALSE                            |
| FLOAT         | float         | 单精度浮点数                                          | 3.14159                               |
| ***DOUBLE***  | double        | 双精度浮点数                                          | 3.14159                               |
| ***STRING***  | string        | 字符系列。可以指定字符集。 可以使用单引号或者双引号。 | ‘now is the time’ “for all  good men” |
| TIMESTAMP     |               | 时间类型                                              |                                       |
| BINARY        |               | 字节数组                                              |                                       |

对于 Hive 的 String 类型相当于数据库的 varchar 类型，该类型是一个可变的字符串， 不过它不能声明其中最多能存储多少个字符，理论上它可以存储 2GB 的字符数。



## 集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ |
| STRUCT   | 和 c 语言中的 struct 类似，都可以通过“点”符号访问元素 内容。例如，如果某个列的数据类型是 STRUCT{first STRING,  last STRING},那么第 1 个元素可以通过字段.first 来引用。 | struct() 例如 struct<street:string, city:string> |
| MAP      | MAP 是一组键-值对元组集合，使用数组表示法可以访问数 据。例如，如果某个列的数据类型是 MAP，其中键->值对 是’first’->’John’和’last’->’Doe’，那么可以通过字 段名[‘last’]获取最后一个元素 | map() 例如 map<string, int>                      |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量 称为数组的元素，每个数组元素都有一个编号，编号从零开 始。例如，数组值为[‘John’, ‘Doe’]，那么第 2 个元素 可以通过数组名[1]进行引用。 | Array() 例如 array<string>                       |

Hive 有三种复杂数据类型 ARRAY、MAP 和 STRUCT。ARRAY 和 MAP 与 Java 中的 Array 和 Map 类似，而 STRUCT 与 C 语言中的 Struct 类似，它封装了一个命名字段集合， 复杂数据类型允许任意层次的嵌套。

实例：

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```



```hive
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```

字段解释： 

row format delimited fields terminated by ',' 		-- 列分隔符 

collection items terminated by '_' 							--MAP STRUCT 和 ARRAY 的分隔符(数据分割 符号) 

map keys terminated by ':' 										-- MAP 中的 key 与 value 的分隔符 

lines terminated by '\n'; 												-- 行分隔符

导入数据：

```hive
load data local inpath '/home/zhuweihao/opt/data/test.txt' into table test;
```

访问三种集合列里的数据，以下分别是 ARRAY，MAP，STRUCT 的访问方式：

```
select friends[1],children['xiao song'],address.city from test
where name="songsong";
```

![image-20220831232113277](HiveSQL.assets/image-20220831232113277.png)



## 类型转换

Hive 的原子数据类型是可以进行隐式转换的，类似于 Java 的类型转换，例如某表达式 使用 INT 类型，TINYINT 会自动转换为 INT 类型，但是 Hive 不会进行反向转化，例如， 某表达式使用 TINYINT 类型，INT 不会自动转换为 TINYINT 类型，它会返回错误，除非 使用 CAST 操作。

隐式类型转换规则如下：

- 任何整数类型都可以隐式地转换为一个范围更广的类型，如 TINYINT 可以转 换成 INT，INT 可以转换成 BIGINT。 
- 所有整数类型、FLOAT 和 STRING 类型都可以隐式地转换成 DOUBLE。 
- TINYINT、SMALLINT、INT 都可以转换为 FLOAT。
- BOOLEAN 类型不可以转换为任何其它的类型。

---

可以使用 CAST 操作显示进行数据类型转换：

例如，CAST('1' AS INT)将把字符串'1' 转换成整数 1；如果强制类型转换失败，如执行 CAST('X' AS INT)，表达式返回空值 NULL。

```hive
select '1'+2, cast('1'as int) + 2;
```

![image-20220831232430848](HiveSQL.assets/image-20220831232430848.png)



# DDL数据定义



## 创建表

```hive
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
[TBLPROPERTIES (property_name=property_value, ...)]
[AS select_statement]
```

字段解释：

- CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异 常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。 
- EXTERNAL 关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向 实际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删 除，而外部表只删除元数据，不删除数据。 
- COMMENT：为表和列添加注释。 
- PARTITIONED BY 创建分区表 
- CLUSTERED BY 创建分桶表 
- SORTED BY 不常用，对桶中的一个或多个列另外排序 
- ROW FORMAT  
  - DELIMITED 
    - [FIELDS TERMINATED BY char]
    - [COLLECTION ITEMS  TERMINATED BY char] 
    - [MAP KEYS TERMINATED BY char] 
    - [LINES TERMINATED BY char]  | 
    - SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value,  property_name=property_value, ...)] 
  - 用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW  FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，用户 还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive 通过 SerDe 确 定表的具体的列的数据。 SerDe 是 Serialize/Deserilize 的简称， hive 使用 Serde 进行行对象的序列与反序列化。 
- STORED AS 指定存储文件类型 常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、 RCFILE（列式存储格式文件） 如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使 用 STORED AS SEQUENCEFILE。 
- LOCATION ：指定表在 HDFS 上的存储位置。 
- AS：后跟查询语句，根据查询结果创建表。 
- LIKE 允许用户复制现有的表结构，但是不复制数据。

### 管理表

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive 会（或多或 少地）控制着数据的生命周期。Hive 默认情况下会将这些表的数据存储在由配置项 hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。当我们 删除一个管理表时，Hive 也会删除这个表中数据。管理表不适合和其他工具共享数据。

### 外部表

因为表是外部表，所以 Hive 并非认为其完全拥有这份数据。删除该表并不会删除掉这 份数据，不过描述表的元数据信息会被删除掉。

### 分区表

分区表实际上就是对应一个 HDFS 文件系统上的独立的文件夹，该文件夹下是该分区 所有的数据文件。Hive 中的分区就是分目录，把一个大的数据集根据业务需要分割成小的 数据集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。