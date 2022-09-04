# Spark、Hive UDF函数编程

# Hadoop

https://blog.csdn.net/fenglailea/article/details/53318459?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1-53318459-blog-124488643.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.2&utm_relevant_index=4

# Hadoop中HDFS的文件到底存储在集群节点本地文件系统哪里

https://blog.csdn.net/weixin_43114954/article/details/115571939?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-115571939-blog-91553693.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-115571939-blog-91553693.pc_relevant_antiscanv2&utm_relevant_index=2

# spark

## Spark 快速开始

```shell
sbin/start-all.sh
jps 查看节点
bin/spark-shell
 spark-shell --master yarn
 
//启动spark
scala> :quit
```

## Spark  UDF、UDAF、UDTF

### UDF

用户自定义函数，1对1；

```java
package spark;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.api.java.UDF2;
import org.apache.spark.sql.types.DataTypes;


public class UDF_1 {
    public static void main(String[] args) {
        SparkSession sparkSession = SparkSession
                .builder()
                .appName("udf_1")
                //.master("local") spark-submit --master local 
                .getOrCreate();

      spark sql  DataSet sql
      
     rdd sparkContext.textFile("")
        
        4040 webUI
        
        spark  driver  excutor
        
        Dataset<Row> json = sparkSession.read().json("src/main/resources/json");
        json.printSchema();
      
        json.createOrReplaceTempView("person");
      
        // 能不能 通过 全类名 反射 调用 udf
        // map、flatmap  下次 整理 spark 常用算子
        sparkSession.udf().register("nameLenAddInt", new UDF2<String, Integer, Integer>() {
            @Override
            public Integer call(String s, Integer integer) throws Exception {
                return s.length()+integer;
            }
        }, DataTypes.IntegerType);
        sparkSession.sql("select name,nameLenAddInt(name,100) as lenght, age from person")
                .show();
      
      
        sparkSession.stop();
    }
}

```

### UDAF

用户自定义聚合函数

```java
package spark;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.expressions.MutableAggregationBuffer;
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction;
import org.apache.spark.sql.types.DataType;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

import java.util.Arrays;

public class UDAF {
    public static void main(String[] args) {
        SparkSession sparkSession = SparkSession.builder()
                .appName("udaf")
                .master("local")
                .getOrCreate();

        Dataset<Row> json = sparkSession.read().json("src/main/resources/json");
        json.printSchema();
        json.createOrReplaceTempView("person");

        StructField[] structTypes = {
                //DataTypes.createStructField("name",DataTypes.StringType,true),
                DataTypes.createStructField("age",DataTypes.IntegerType,true)
        };
        sparkSession.udf().register("count1", new UserDefinedAggregateFunction() {
            @Override
            public StructType inputSchema() {
                return DataTypes.createStructType(structTypes);
            }

            @Override
            public StructType bufferSchema() {
                return DataTypes.createStructType(Arrays.asList(
                        DataTypes.createStructField("count",DataTypes.IntegerType,true)));
            }

            @Override
            public DataType dataType() {
                return DataTypes.IntegerType;
            }
//相同输入是否输出相同的结果
            @Override
            public boolean deterministic() {
                return true;
            }

            @Override
            public void initialize(MutableAggregationBuffer buffer) {
                buffer.update(0,0);
            }

            @Override
            public void update(MutableAggregationBuffer buffer, Row input) {

                int updateValue = buffer.getInt(0)+ 1;
                buffer.update(0,updateValue);

            }

            @Override
            public void merge(MutableAggregationBuffer buffer1, Row buffer2) {
               int  mergeValue = buffer1.getInt(0)+buffer2.getInt(0);
               buffer1.update(0,mergeValue);
            }

            @Override
            public Object evaluate(Row buffer) {
                return buffer.getInt(0);
            }
        });
        sparkSession.sql("select age,count1(age)  from person group by age")
                .show();
      
        sparkSession.stop();
    }
}

```



## Spark Bug

**Spark：3.0.3版本报错“**

**: JAVA_9“**

```
https://blog.csdn.net/suwei825/article/details/120448219
```

# Hive

## Hive Quick Start

hdfs 网址：http://hadoop102:50070



内部表

​	迁移 / usr/hive/warehouse/库名/xxx

外部表

​	不迁移

Hadoop集群配置

（1）必须启动hdfs和yarn

```shell
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh

[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

（2）在HDFS上创建/tmp和/user/hive/warehouse两个目录并修改他们的同组权限可写

```shell
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir /tmp

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir -p /user/hive/warehouse

 

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /tmp

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /user/hive/warehouse
```

3．Hive基本操作

（1）启动hive

```shell
[atguigu@hadoop102 hive]$ bin/hive
```

（2）查看数据库

```shell
hive> show databases;
```

（3）打开默认数据库

```
hive> use default;
```

（4）显示default数据库中的表

```
hive> show tables;
```

（5）创建一张表

```
hive> create table student(id int, name string);
```

（6）显示数据库中有几张表

```
hive> show tables;
```

（7）查看表的结构

```
hive> desc student;
```

（8）向表中插入数据

```
hive> insert into student values(1000,"ss");
```

（9）查询表中数据

```
hive> select * from student;
```

（10）退出hive

```
hive> quit;
```

文件存储结构

```
dfs -ls /user/hive/warehouse;
```

## Hive使用场景

在大数据的发展当中，大数据技术生态的组件，也在不断地拓展开来，而其中的Hive组件，作为Hadoop的数据仓库工具，可以实现对Hadoop集群当中的大规模数据进行相应的数据处理。今天我们的大数据入门分享，就主要来讲讲，Hive应用场景。

关于Hive，首先需要明确的一点就是，Hive并非数据库，Hive所提供的数据存储、查询和分析功能，本质上来说，并非传统数据库所提供的存储、查询、分析功能。



Hive数据仓库工具将结构化的数据文件映射为一张数据库表，并提供SQL查询功能，能将SQL语句转变成MapReduce任务来执行。通过类SQL语句实现快速MapReduce统计，使MapReduce编程变得更加简单易行。

Hive应用场景

总的来说，Hive是十分适合数据仓库的统计分析和Windows注册表文件。

Hive在Hadoop中扮演数据仓库的角色。Hive添加数据的结构在HDFS(Hive superimposes structure on data in HDFS)，并允许使用类似于SQL语法进行数据查询。

Hive更适合于数据仓库的任务，主要用于静态的结构以及需要经常分析的工作。Hive与SQL相似促使其成为Hadoop与其他BI工具结合的理想交集。

Hive使用

Hive在Hadoop之上，使用Hive的前提是先要安装Hadoop。



Hive要分析的数据存储在HDFS，Hive为数据创建的表结构(schema)，存储在RDMS(relevant database manage system关系型数据库管理系统，比如mysql)。

Hive构建在基于静态批处理的Hadoop之上，Hadoop通常都有较高的延迟并且在作业提交和调度的时候需要大量的开销。因此，Hive并不能够在大规模数据集上实现低延迟快速的查询，例如，Hive在几百MB的数据集上执行查询一般有分钟级的时间延迟。

Hive查询操作过程严格遵守Hadoop MapReduce的作业执行模型，Hive将用户的HiveQL语句通过解释器转换为MapReduce作业提交到Hadoop集群上，Hadoop监控作业执行过程，然后返回作业执行结果给用户。Hive的最佳使用场合是大数据集的批处理作业，例如，网络日志分析。

Hive优缺点

优点：

操作接口采用类SQL语法，提供快速开发的能力(简单、容易上手)。

Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。

Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。

Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。



缺点：

1．Hive的HQL表达能力有限

(1)迭代式算法无法表达递归算法

(2)数据挖掘方面不擅长(数据挖掘和算法机器学习)

2．Hive的效率比较低

(1)Hive自动生成的MapReduce作业，通常情况下不够智能化

(2)Hive调优比较困难，粒度较粗(快)

关于大数据入门，Hive应用场景，以上就为大家做了大致的介绍了。在大数据应用场景下，Hive更多是作为Hadoop的一个数据仓库工具，并不直接存储数据，但是却不可或缺。

```
原文链接：https://blog.csdn.net/weixin_29811891/article/details/111947054
```



## Hive操作

1. 

```
hive (default)> insert overwrite local directory '/usr/local/datas/student'
              > select * from student;
```

结果：在`'/usr/local/datas/student'`目录下 有一个文件：

![image-20220620150335171](E:\image\image-20220620150335171.png)

点击vim查看这个`000000_0`：

![image-20220620150508533](E:\image\image-20220620150508533.png)

右图是使用下面的命令得到的结果，对其进行了而格式化

```
hive (default)> insert overwrite local directory '/usr/local/datas/student'
              >  ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
              > select * from student;
```

2.

```
hive (default)> insert overwrite directory '/user/hive/student2'
              > ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
              > select * from student;
```

![image-20220620151320256](E:\image\image-20220620151320256.png)

这个文件是否只是存在内存里，下次启动再来看一下，是否还存在。

试验验证，是写在内存里，但是路径会被创建且会保留。

## hive多个列排序**

按照部门和工资升序排序

```
hive (default)> select ename, deptno, sal from emp order by deptno, sal ;
```

例子：

```
hive (default)> select * from student;
OK
student.id	student.name
1001	Wangwenpeng
1002	Liuchiyu
1003	HarryPotter
1004	awadasomingzhou
1005	Anser
1006	lili
1006	cc
Time taken: 0.122 seconds, Fetched: 7 row(s)
```

```
hive (default)> select id, name from student order by id, name;
OK
id	name
1001	Wangwenpeng
1002	Liuchiyu
1003	HarryPotter
1004	awadasomingzhou
1005	Anser
1006	cc
1006	lili
Time taken: 1.534 seconds, Fetched: 7 row(s)
```

发现：先按照第一个进行排序，然后按照第一个排序结果按照id进行了分组，在id组内再按照第二个排序规则进行排序。

## Hive自定义函数

### (1) UDF（User-Defined-Function）

1.jar 包的准备；

maven项目的pom依赖

```xml
<dependencies>
		<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.3</version>
		</dependency>
</dependencies>
```

```java
//这是我们自定义的UDF函数，我们使用maven插件打包后，将jar包放在/usr/local/hive_shell/目录下
// 将jar包添加到hive的classpath上，通过包名和类名来定位位置，在本例中：org.exzample.Lower;
package org.example;
import org.apache.hadoop.hive.ql.exec.UDF;

public class Lower extends UDF {
public String evaluate (final String s) {
		
		if (s == null) {
			return null;
		}
		
		return s.toLowerCase();
	}
}
```

我们准备的表，将名字这一字段全变成小写；

```
hive (default)> select * from student;
OK
student.id	student.name
1001	Wangwenpeng
1002	Liushiyu
1003	HarryPotter
1004	awadasomingzhou
1005	Anser
1006	lili
1006	cc
Time taken: 0.165 seconds, Fetched: 7 row(s)
```

将jar包添加到hive的classpath路径上。

```
hive (default)> add jar /usr/local/hive_shell/hive-1.0-SNAPSHOT.jar;
Added [/usr/local/hive_shell/hive-1.0-SNAPSHOT.jar] to class path
Added resources: [/usr/local/hive_shell/hive-1.0-SNAPSHOT.jar]
```

定义函数，根据`"org.example.Lower"`来定位我们的UDF。

```
hive (default)> create temporary function mylower as "org.example.Lower";
OK
Time taken: 0.005 seconds

```

查询结果。

```
hive (default)> select name, mylower(name) lname from student;
OK
name	lname
Wangwenpeng	wangwenpeng
Liuchiyu	liuchiyu
HarryPotter	harrypotter
awadasomingzhou	awadasomingzhou
Anser	anser
lili	lili
cc	cc
Time taken: 0.424 seconds, Fetched: 7 row(s)
```

使用心得：

将jar包添加到hive的classpath 里只有再这个命令窗口有效，一旦窗口关闭，就需要将上述步骤重新再来一遍。如果你有你修改了jar包里的东西，需要关闭hive，再重新打开，重复上述步骤，否则不会生效；在这里本人感觉是hive，将这个jar临时保存了起来，访问jar包不是访问本地的jar包，而是他的一份复制。

使用UDF一般创建的就是临时函数（temperary），对于像hive自带的一些全局函数（global），极少使用，一般是架构师来写的。

问题：

idea的maven打包插件原理，根据pom文件进行打包，如果没有设置，是不是就不把依赖的包打进jar包里了？

### (2) UDAF（User-Defined Aggregation Function）

- in:out=n:1,即输入N条数据，返回一条处理结果，即列转行。
- 最常见的系统聚合函数，如count，sum，avg，max等

实现步骤

- 自定义一个java类
- 继承UDAF类
- 内部定义一个静态类，实现UDAFEvaluator接口
- 实现方法init，iterate，terminatePartial，merge，terminate共5个方法

![image-20220621175348746](E:\image\image-20220621175348746.png)



```java
package org.example;

import org.apache.hadoop.hive.ql.exec.UDAF;
import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;
import org.apache.log4j.Logger;


public class DIYCountUDAF extends UDAF {
    
    public static  Logger logger = Logger.getLogger(DIYCountUDAF.class);
    
    public static class DIYEvaluator implements UDAFEvaluator{
        public int totalCounter = 0;
        //init方法解决map和reduce阶段变量的初始化问题
        @Override
        public void init() {
            totalCounter = 0;
            logger.info(" in init method,totalCount = "+totalCounter);
        }
        //解决map阶段，sort完成后以组为单位逐条进行处理
        public boolean iterate(String line){
            if (line != null && line.trim().length()>0){
                totalCounter++;
            }
            return true;
        }
        //相当于compiler，对iterate的处理结果进行最后的处理操作
        public int terminatePartial(){
            return totalCounter;
        }
        //merge的过程中，即逐组处理n个map的terminatePartial的输出
        public boolean merge(int mapOutput){
            totalCounter +=mapOutput;
            return true;
        }
        //最后的返回结果
        public int terminate(){
            logger.info("in terminate,totalCounter="+totalCounter);
            return totalCounter;
        }
    }
}

```

和UDF一样，打包

```
hive (default)> add jar /usr/local/hive_shell/hive-1.0-SNAPSHOT.jar;
Added [/usr/local/hive_shell/hive-1.0-SNAPSHOT.jar] to class path
Added resources: [/usr/local/hive_shell/hive-1.0-SNAPSHOT.jar]

```

建立临时函数

```
hive (default)> create temporary function mycount as "org.example.DIYCountUDAF"; 
OK
Time taken: 0.428 seconds
```

执行查询

```
hive (default)> select count(1),mycount(1) from student;
Query ID = root_20220621022953_3486aa9c-9288-47e2-ae5d-0228fe155f7b
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2022-06-21 02:29:55,473 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local999637676_0002
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 392 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
_c0	_c1
7	7
Time taken: 1.991 seconds, Fetched: 1 row(s)
```

### (3) UDTF（User-Defined Table-Generating Functions）

- 解决一行输入多行输出，即1：n，即行专列应用
- 往往被lateral view explode + udf等替代实现，比直接用udtf更简单直接灵活

```sql
select id,name,score from user_score lateral view explode(split(score_list,',')) score_table as score;
// explode里面只能是一个集合，其返回一个表，这个表是集合的表，如果有多个字段，在后面添加score、K1、K2
//就好
```

![0a767b6b564fee8378b1303222e4da6](E:\image\0a767b6b564fee8378b1303222e4da6.jpg)

自定义的

```java
package org.example;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;

public class UDTF extends GenericUDTF {
    @Override
    public StructObjectInspector initialize(ObjectInspector[] args)
            throws UDFArgumentException {
        if (args.length != 1) {
            throw new UDFArgumentLengthException("ExplodeMap takes only one argument");
        }
        if (args[0].getCategory() != ObjectInspector.Category.PRIMITIVE) {
            throw new UDFArgumentException("ExplodeMap takes string as a parameter");
        }

        ArrayList<String> fieldNames = new ArrayList<String>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();
        fieldNames.add("col1");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        fieldNames.add("col2");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }

    @Override
    public void process(Object[] args) throws HiveException {
        String input = args[0].toString();
        String[] test = input.split(";");
        for(int i=0; i<test.length; i++) {
            try {
                String[] result = test[i].split(":");
                forward(result);
            } catch (Exception e) {
                continue;
            }
        }
    }

    @Override
    public void close() throws HiveException {

    }
}

```

输出

```sql
col1	col2
key1	value1
key2	value2
key3	value3
key4	value4
key5	value5
ke6	    value6
key7	value7
```



## Hive 使用发现

1. 

```
hive (default)> dfs -put /usr/local/datas/student.txt /user/hive;
```

会将本地文件发送到hive上去，此时你可以再web UI上看到这个文件，接下来你对这个文件进行装载：

```
 load data inpath '/user/hive/student.txt' into table default.student;
```

这时你会发现 `/user/hive/student.txt`文件不见了，此时数据文件已经写入student table 里了，这时在web UI 中点开stuent 表，你会发现这个文件被存储在了hdfs 的datanode里面了，无论是从本地装载还是将本地文件上传到hdfs，再将数据写入table，都会将文件存储到hdfs的data node 里面。

![image-20220620142137262](E:\image\image-20220620142137262.png)

2.



![image-20220620110547239](E:\image\image-20220620110547239.png)



hive 默认是将/user/hive/warehouse当作默认存放位置，创建的表、数据库等存储结构都放在这里，当创建了数据库后，我们就可以在数据库里，再建立表。与MySQL不同的是，MySQL只可以再数据库中建立表，而hive可以不在数据库里见表，或者是默认的default就是一个数据库，允许再数据库里再建数据库，这就与MySQL一样了，MySQL也允许再数据库里再建数据库。

3.

hive本身并不具有存储数据的能力，其只是提供了一种查询的方式，类似mysql。通过hive的hql语句进行数据的建表，删表，等数据操作，操作的数据表的源数据时存储再hdfs里的，其保存的数据modified的文本，类似于日记，因该可以通过这个实现检查点机制,如果可以的话，hive是否有自己的日志文件，hive是否存储了表，如果重启hive，查看一张表，是否是按照日志进行查询，如果进行查询，是从头开始，还是最新？联想到kafka也是有这个设置的，如果猜测正确，那么两者有什么异同呢？以上是个人见解，正确性有待考证。



## Hive错误

```
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeMo                 deException): Cannot create directory /tmp/hive/root/9dc0dfcd-a02d-41f3-b524-dc85089e4c3f. Name node is in safe mode.
```


解决办法：没有关闭安全模式，直接强制离开安全模式就行了

```
hdfs dfsadmin -safemode leave
```

## 案例实操

1） 假设某表有如下一行，我们用JSON格式来表示其数据结构。在Hive下访问的格式为

```
{  "name": "songsong",  "friends": ["bingbing" , "lili"] ,    //列表Array,   "children": {            //键值Map,    "xiao song": 18 ,    "xiaoxiao song": 19  }  "address": {            //结构Struct,    "street": "hui long guan" ,    "city": "beijing"   }}
```

2）基于上述数据结构，我们在Hive里创建对应的表，并导入数据。 

创建本地测试文件test.txt

```
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijingyangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```

注意：MAP，STRUCT和ARRAY里的元素间关系都可以用同一个字符表示，这里用“_”。

3）Hive上创建测试表test

```
create table test(name string,friends array<string>,children map<string, int>,address struct<street:string, city:string>)row format delimited fields terminated by ','collection items terminated by '_'map keys terminated by ':'lines terminated by '\n';
```

字段解释：

```
row format delimited fields terminated by ','  -- 列分隔符

collection items terminated by '_'  --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)

map keys terminated by ':'				-- MAP中的key与value的分隔符

lines terminated by '\n';					-- 行分隔符
```

4）导入文本数据到测试表

```
hive (default)> load data local inpath ‘/opt/module/datas/test.txt’into table test
```

5）访问三种集合列里的数据，以下分别是ARRAY，MAP，STRUCT的访问方式

```
hive (default)> select friends[1],children['xiao song'],address.city from testwhere name="songsong";OK_c0   _c1   citylili   18    beijingTime taken: 0.076 seconds, Fetched: 1 row(s)
```

# 问题

1.

![image-20220620125232709](E:\image\image-20220620125232709.png)

这个/user和/tmp是啥？

虽说再web UI 上能看到，但是呢，是否是具体的文件路径，感觉比较抽象。

以下是提示：

Hadoop集群配置

（1）必须启动hdfs和yarn

```
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh

[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

（2）在HDFS上创建/tmp和/user/hive/warehouse两个目录并修改他们的同组权限可写

```
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir /tmp

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir -p /user/hive/warehouse
```

 

```
[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /tmp

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /user/hive/warehouse
```

2.

大数据的学习感觉自己需要了学习他们的架构，这样才能有一个更清楚的认知，现在学习了Flink，hdfs，hive，kafka，这几个插件，有一种朦胧的感觉，但又说不出来，还自我感觉不错！！！

```


update.buffer[0]
 update.input[18]
update.buffer[0]
 update.input[19]
update.buffer[0]
 update.input[28]
update.buffer[0]
 update.input[38]
update.buffer[1]
 update.input[28]
update.buffer[1]
 update.input[38]
update.buffer[0]
 update.input[null]
2022-06-22 01:28:33,809   INFO --- [            Executor task launch worker for task 1]  org.apache.spark.sql.catalyst.expressions.codegen.CodeGenerator                 (line:    

[0]
[0]
merge1,2:[0]
 [2]
evaluate:[2]



[0]
[0]
merge1,2:[0]
 [1]
evaluate:[1]

[0]
[0]
merge1,2:[0]
 [1]
evaluate:[1]

[0]
[0]
merge1,2:[0]
 [1]
evaluate:[1]

[0]
[0]
merge1,2:[0]
 [2]
evaluate:[2]


    
+----+------------------+
| age|(CAST(age AS INT))|
+----+------------------+
|  28|                 2|
|null|                 1|
|  18|                 1|
|  19|                 1|
|  38|                 2|
+----+------------------+



Process finished with exit code 0

```

```
init:[0]
init:[0]

update.buffer[0]
 update.input[18]
update.buffer[0]
 update.input[19]
update.buffer[0]
 update.input[28]
update.buffer[0]
 update.input[38]
update.buffer[1]
 update.input[28]
update.buffer[1]
 update.input[38]
update.buffer[0]
 update.input[null]
 
 init:[0]
 init:[0]
merge1,2:[0]
 [2]
evaluate:[2]

init:[0]
init:[0]
merge1,2:[0]
 [1]
evaluate:[1]

init:[0]
init:[0]
merge1,2:[0]
 [1]
evaluate:[1]

init:[0]
init:[0]
merge1,2:[0]
 [1]
evaluate:[1]

init:[0]
init:[0]
merge1,2:[0]
 [2]
evaluate:[2]

| age|(CAST(age AS INT))|
+----+------------------+
|  28|                 2|
|null|                 1|
|  18|                 1|
|  19|                 1|
|  38|                 2|
+----+------------------+
```

```
{"name":"zhangsan","age":"18"}
{"name":"zhangsi","age":"19"}
{"name":"zhangwu","age":"28"}
{"name":"zhang6","age":"38"}
{"name":"zhang7777","age":"28"}
{"name":"zhang88","age":"38"}
{"name":"zhangSSS"}
```

经过观察 发现初始化了12次，数总共7条，分组的话是5组，j 