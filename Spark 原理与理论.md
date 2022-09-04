# Spark 原理与理论

## Spark RDD

RDD是弹性分布式数据集，即一个RDD代表一个被分区的只读数据集，一个RDD的生成只有两种途径，一是来自于内存集合和外部存储系统，另一种是通过转换操作，从其他的RDD获得。比如：map，filter，join等。

RDD没必要随时被实例化，由于RDD的接口只支持粗粒度的操作（即一个操作会被应用在RDD的所有数据上），所以只要通过记录下这些作用在RDD之上的转换操作，来构建RDD的继承关系 ，就可以有效的进行容错处理，而不需要将实际的RDD数据进行记录拷贝。在一个Spark程序中，我们所用到的每一个RDD，在丢失或者操作失败后都是可以重建的。

**RDD的控制操作：持久化和分区**

**持久化：**开发者可以指明他们需要重用哪些RDD，然后选择一种存储策略将它们保存起来 。开发者还可以让RDD根据记录中的键值在集群的机器之间重新分区。



**抽象RDD需要包含的5个接口**

partition：分区，一个RDD会有一个或者多个分区

preferredLocation：对于分区P而言，返回数据本地化计算的节点

dependencies：RDD的依赖关系

compute：对于分区P而言，进行迭代计算

partitioner：RDD的分区函数

## RDD分区

对于一个RDD而言，分区的多少涉及对这个RDD进行并行计算的粒度，每一个RDD分区的计算操作都在一个单独的任务中被执行。对于RDD分区而言，用户可以自行指定多少分区，如果没有指定，那么将会使用默认值。可以利用RDD的成员变量partitions所返回的partition数组的大小来查询一个RDD被划分的分区数。

```
rdd.partitions.size;
```

系统默认的分区数是这个程序所分配到的资源的CPU核的个数。

## RDD优先位置（preferredLocations）

RDD优先位置属性与Spark中的调度相关，返回的是此RDD的每个partition所存储的位置，按照“移动数据不如移动计算”的理念，在Spark进行任务调度的时候，尽可能地将任务分配到数据块所存储地位置。

## RDD依赖关系（dependencies）

由于RDD是粗粒度地操作数据集，每一个转换操作都会生成一个心得RDD，所以RDD之间就会形成类似于流水线一样地前后依赖关系。在spark中存在两种类型地依赖，窄依赖和宽依赖。

**窄依赖：**每一个父RDD的分区最多只被子RDD的一个分区所使用

**宽依赖：**多个子RDD的分区会依赖于同一个父RDD的分区

![img](https://img-blog.csdnimg.cn/9b3a4ff4c23c496ab658433b0ae1d431.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARGF0Yei3s-WKqA==,size_20,color_FFFFFF,t_70,g_se,x_16)

在Spark中需要明确地区分这两种依赖关系有两个方面地原因：

1. 窄依赖可以在集群地一个节点上如流水线一般地执行，可以计算所有父RDD的分区；相反地，宽依赖需要取得父RDD的所有分区上的数据进行计算，将会执行类似于Map Reduce一样的Shuffle操作。
2. 对于窄依赖来说，节点计算失败后的恢复会更加有效，只需要重新计算对应的父RDD的分区，而且可以在其他的节点上并行地计算；相反的，在有宽依赖的继承关系中，一个节点的失败将会导致其父节点的多个分区的重新计算，这个代价是非常高的。



# Shuffle流程

## 1、Shuffle定义

Spark之所以出现Shuffle，主要是因为具有某种共同特征的一类数据需要最终汇聚到一个计算节点上进行计算。这些数据分布在各个存储节点上，并且由不用的计算单元进行处理。这种数据打乱然后汇聚到不同节点的过程就是Shuffle。

## 2、Shuffle流程

Spark是以Shuffle为边界，将一个Job划分为不同的Stage，这些Stage构成了一个大粒度的DAG。Spark的Shuffle主要分为Shuffle Write和Shuffle Read两个阶段。

执行Shuffle的主体是Stage中的并发任务，这些任务分为ShuffleMapTask和ResultTask两大类。ShuffleMapTask要进行Shuffle，ResultTask负责返回计算结果，一个Job中只有最后一个Stage采用ResultTask，其它均为ShuffleMapTask。

如果按照map端和reduce端来分析的话，ShuffleMapTask可以即是map端任务，又是reduce端任务。因为Spark中的Shuffle可以是串行的，ResultTask则只能充当reduce端任务的角色。

![img](https:////upload-images.jianshu.io/upload_images/17264076-26a882a89fdd7d34.png?imageMogr2/auto-orient/strip|imageView2/2/w/674/format/webp)

Shuffle流程图

Spark Shuffle流程简单抽象为如下几个步骤：

- Shuffle Write
- 如果需要的话，先在map端做数据预聚合
- 写入本地输出文件
- Shuffle Read
- fetch数据块
- reduce端做数据预聚合
- 如果需要的话，进行数据排序
- Shuffle Write阶段：发生于ShuffleMapTask对该Stage的最后一个RDD完成了map端的计算之后，首先会判断是否需要对计算结果进行聚合，然后将最终结果按照不同的reduce端进行区分，写入前节点的本地磁盘。
- Shuffle Read阶段：开始于reduce端的任务读取ShuffledRDD之后，首先通过远程或者本地数据拉取获得Write阶段各个节点中属于当前任务的数据，根据数据的Key进行聚合，然后判断是否需要排序，最后生成新的RDD。

# Shuffle技术演进

在Spark Shuffle的具体实现上，主要经历了：hash-based shuffle、sort-based shuffle、Tungsten-sort shuffle 三个大的阶段。

## 1、hash-based shuffle V1

在Spark 0.8 及之前的版本采用：hash-based shuffle机制。

### 1）Shuffle流程介绍

#### （1）Shuffle Write

在Shuffle Write过程会按照hash的方式重组partition的数据，不进行排序。每个map端的任务为每个reduce端的任务都生成一个文件，通过会产生大量的文件（假如map端task数量为m，reduce端task数量为n，则对应 m * n个中间文件），其中伴随着大量的随机磁盘IO操作与大量的内存开销。

#### （2）Shuffle Read

Reduce端任务首先将Shuffle write生成的文件fetch到本地节点，如果Shuffle Read阶段有combiner操作，则它会把拉到的数据保存在一个Spark封装的哈希表（AppendOnlyMap）中进行合并。

![img](https:////upload-images.jianshu.io/upload_images/17264076-e3edbdae8449570f.png?imageMogr2/auto-orient/strip|imageView2/2/w/919/format/webp)

Shuffle Read流程图

### 2）源码结构

在代码结构上：

- org.apache.spark.storage.ShuffleBlockManager 负责 Shuffle Write
- org.apache.spark.BlockStoreShuffleFetcher 负责 Shuffle Read
- org.apache.spark.Aggregator 负责 combine，依赖于 AppendOnlyMap

### 3）优缺点

该版本的Spark Shuffle机制存在如下两个严重问题：

1. 生成大量的文件，占用文件描述符，同时引入DiskObjectWriter带来的Writer Handler的缓存也非常消耗内存；
2. 如果在Reduce Task时需要合并操作的话，会把数据放在一个HashMap中进行合并，如果数据量较大，很容易引发OOM。

## 2、Hash Shuffle V2

在Spark 0.8.1 针对原来的hash-based shuffle机制，引入了 File Consolidation 机制。

一个 Executor 上所有的 Map Task 生成的分区文件只有一份，即将所有的 Map Task 相同的分区文件合并，这样每个 Executor 上最多只生成 N 个分区文件。

![img](https:////upload-images.jianshu.io/upload_images/17264076-721fdda049689f70.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Shuffle Read流程图

这样就减少了文件数，但是假如下游 Stage 的分区数 N 很大，还是会在每个 Executor 上生成 N 个文件，同样，如果一个 Executor 上有 K 个 Core，还是会开 K*N 个 Writer Handler，所以这里仍然容易导致OOM。

是否采用File Consolidation机制，需要配置 spark.shuffle.consolidateFiles 参数。

## 3、Hash Shuffle V3

在Spark 0.9 引入了ExternalAppendOnlyMap。

在combine的时候，可以将数据spill到磁盘，然后通过堆排序merge。

## 4、Sort Shuffle V1

为了更好的解决上面的问题，Spark 参考了 MapReduce 中的 Shuffle 的处理方式，

在Spark 1.1 引入了 sort-based shuffle ，但是默认仍为 hash-based shuffle。在Spark1.2 将默认的Shuffle方式修改为sort-based shuffle。

每个 Task 不会为后续的每个 Task 创建单独的文件，而是将所有对结果写入同一个文件。该文件中的记录首先是按照 Partition Id 排序，每个 Partition 内部再按照 Key 进行排序，Map Task 运行期间会顺序写每个 Partition 的数据，同时生成一个索引文件记录每个 Partition 的大小和偏移量。

在Reduce阶段，Reduce Task拉取数据做Combine时不再是采用HashMap，而是采用ExternalAppendOnlyMap，该数据结构在做Combine时，如果内存不足，会刷写磁盘，很大程度上保证了系统的鲁棒性，避免了大数据情况下的OOM。

总体来看，Sort Shuffle解决了 Hash Shuffle 的所有弊端，但是因为需要其Shuffle过程需要对记录进行排序，所以在性能上有所损失。

### 代码结构

在代码结构上：

- 从以前的 ShuffleBlockManager 中分离出 ShuffleManager 专门管理 Shuffle Write 和 Shuffle Read。两种Shuffle方式分别对应：
- org.apache.spark.shuffle.hash.HashShuffleManager
- org.apache.spark.shuffle.sort.SortShuffleManager
- org.apache.spark.util.collection.ExternalSorter 实现排序功能。可通过对spark.shuffle.spill 参数配置，决定是否可以在排序时将临时数据Spill到磁盘。

## 5、Tungsten-Sort Based Shuffle

从 Spark 1.5.0 开始，Spark 开始了钨丝计划（Tungsten），目的是优化内存和CPU的使用，进一步提升spark的性能。由于使用了堆外内存，而它基于 JDK Sun Unsafe API，故 Tungsten-Sort Based Shuffle 也被称为 Unsafe Shuffle。

它的做法是将数据记录用二进制的方式存储，直接在序列化的二进制数据上 Sort 而不是在 Java 对象上，这样一方面可以减少内存的使用和 GC 的开销，另一方面避免 Shuffle 过程中频繁的序列化以及反序列化。在排序过程中，它提供 cache-efficient sorter，使用一个 8 bytes 的指针，把排序转化成了一个指针数组的排序，极大的优化了排序性能。

但是使用 Tungsten-Sort Based Shuffle 有几个限制，Shuffle 阶段不能有 aggregate 操作，分区数不能超过一定大小（2^24-1，这是可编码的最大 Parition Id），所以像 reduceByKey 这类有 aggregate 操作的算子是不能使用 Tungsten-Sort Based Shuffle，它会退化采用 Sort Shuffle。

## 6、Sort Shuffle V2

从 Spark-1.6.0 开始，把 Sort Shuffle 和 Tungsten-Sort Based Shuffle 全部统一到 Sort Shuffle 中，如果检测到满足 Tungsten-Sort Based Shuffle 条件会自动采用 Tungsten-Sort Based Shuffle，否则采用 Sort Shuffle。从Spark-2.0.0开始，Spark 把 Hash Shuffle 移除，可以说目前 Spark-2.0 中只有一种 Shuffle，即为 Sort Shuffle。

# 三、总结

## 1、Shuffle Read相关问题

关于Shuffle Read，主要了解以下问题：

**1）在什么时候获取数据，Parent Stage的一个ShuffleMapTask执行完还是等全部ShuffleMapTask执行完？**

当Parent Stage的所有ShuffleMapTasks结束后再fetch。

**2）边获取边处理还是一次性获取完再处理？**

因为Spark不要求Shuffle后的数据全局有序，因此没必要等到全部数据shuffle完成后再处理，所以是边fetch边处理。

**3）获取来的数据存放在哪里？**

刚获取来的 FileSegment 存放在 softBuffer 缓冲区，经过处理后的数据放在内存 + 磁盘上。

内存使用的是AppendOnlyMap ，类似 Java 的HashMap，内存＋磁盘使用的是ExternalAppendOnlyMap，如果内存空间不足时，ExternalAppendOnlyMap可以将 records 进行 sort 后 spill（溢出）到磁盘上，等到需要它们的时候再进行归并

**4）怎么获得数据的存放位置？**

通过请求 Driver 端的 MapOutputTrackerMaster 询问 ShuffleMapTask 输出的数据位置。

## 2、Shuffle触发机制

如下算子会触发Shuffle：

1. repartition类：repartition、coalesce
2. *ByKey类：groupByKey、reduceByKey、combineByKey、aggregateByKey等
3. join相关：cogroup、

# 四、Spark Shuffle版本变更

- Spark 0.8 及以前 Hash Based Shuffle
- Spark 0.8.1 为 Hash Based Shuffle引入File Consolidation机制
- Spark 0.9 引入 ExternalAppendOnlyMap
- Spark 1.1 引入 Sort Based Shuffle，但默认仍为 Hash Based Shuffle
- Spark 1.2 默认的 Shuffle 方式改为 Sort Based Shuffle
- Spark 1.4 引入 Tungsten-Sort Based Shuffle
- Spark 1.6 Tungsten-Sort Based Shuffle 并入 Sort Based Shuffle
- Spark 2.0 Hash Based Shuffle 退出历史舞台

# 参考资料

1. [深入理解Spark Shuffle](https://links.jianshu.com/go?to=https%3A%2F%2Fzhangchenchen.github.io%2F2018%2F09%2F26%2Fdeep-in-spark-shuffle%2F)

2. [Spark Shuffle工作原理详解](https://links.jianshu.com/go?to=http%3A%2F%2Flionheartwang.github.io%2Fblog%2F2018%2F03%2F11%2Fspark-shuffle-implementation%2F)

3. [彻底搞懂 Spark 的 shuffle 过程（shuffle write）](https://links.jianshu.com/go?to=https%3A%2F%2Ftoutiao.io%2Fposts%2Feicdjo%2Fpreview)

4. [Spark Shuffle 详解](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F67061627)

5. [Spark源码解读(6)——Shuffle过程](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fscalahome%2Farticle%2Fdetails%2F52057982)

   

## 实验

## PageRank

### PageRank介绍

![image-20220707102701467](https://s2.loli.net/2022/07/07/KSbmPpL1cyCuWV9.png)



算法步骤实现介绍：

- 将每一个URL的权重都设置为1.
- 对于每一次迭代，将每一个URL的权重贡献（权重/指向的URL个个数）发送给邻居
- 对于每一个URL，将收到的权重贡献相加成contribs，重新计算ranks=0.15+0.85*contribs。

```
[(A,D), (B,A), (C,A), (C,B), (D,A), (D,C)]
[(A,1.0), (B,1.0), (C,1.0), (D,1.0)]

(A,D)
(B,A)
(C,A)
(C,B)
(D,A)
(D,C)

(A,1.0)
(B,1.0)
(C,1.0)
(D,1.0)

(B,(A,1.0))
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
(D,(A,1.0))
(D,(C,1.0))

(B,(A,1.0))
(A,(D,1.0))
(C,(A,0.5))
(C,(B,0.5))
(D,(A,0.5))
(D,(C,0.5))
得到一个contribs
(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)
：


(B,[(A,1.0)])
(A,[(D,1.0)])
(C,[(A,1.0), (B,1.0)])
(D,[(A,1.0), (C,1.0)])
程序结果：
(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)

(B,0.575)
(A,1.8499999999999999)
(C,0.575)
(D,1.0)
```

```java
package spark_example;

import avro.shaded.com.google.common.collect.Iterables;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.*;
import org.apache.spark.sql.types.Decimal;
import scala.Tuple2;

import java.io.Serializable;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public class PageRank_1 implements Serializable {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
//边的关系
        Tuple2[] links = {Tuple2.apply('A', 'D'), Tuple2.apply('B', 'A'),
                Tuple2.apply('C', 'A'), Tuple2.apply('C', 'B')
                , Tuple2.apply('D', 'A'), Tuple2.apply('D', 'C')};
        List<Tuple2> tuple2s = Arrays.asList(links);
       
        System.out.println(tuple2s);
        
    //权重    
        JavaRDD<Tuple2> linksRdd = sc.parallelize(tuple2s);

        Tuple2[] ranks = {Tuple2.apply('A', 1.0), Tuple2.apply('B', 1.0),
                Tuple2.apply('C', 1.0), Tuple2.apply('D', 1.0)};
    
        List<Tuple2> tuple2s1 = Arrays.asList(ranks);
        System.out.println(tuple2s1);
        
        
        JavaRDD<Tuple2> RanksRdd = sc.parallelize(tuple2s1);
        JavaPairRDD<Character, Character> linkPairRDD = linksRdd.mapToPair(s -> {
            return new Tuple2<Character, Character>((Character) s._1, (Character) s._2);
        });
        JavaPairRDD<Character, Double> rankPairRDD = RanksRdd.mapToPair(s -> {
            return new Tuple2<Character, Double>((Character) s._1, (Double) s._2);
        });
        linkPairRDD.foreach(x -> System.out.println(x));
        System.out.println();
        
        
        rankPairRDD.foreach(x -> System.out.println(x));
        
        //进行join构造边权值的关系
        JavaPairRDD<Character, Tuple2<Character, Double>> join = linkPairRDD.join(rankPairRDD);
        System.out.println();
        join.foreach(x -> System.out.println(x));
        
        //根据key进行分组，确定一个节点自己的邻居
        JavaPairRDD<Character, Iterable<Tuple2<Character, Double>>> characterIterableJavaPairRDD = join.groupByKey();
        characterIterableJavaPairRDD.foreach(x -> System.out.println(x));
        
        //根据values进行计算contribs；
        JavaRDD<Object> objectJavaRDD = characterIterableJavaPairRDD.values().flatMap(new FlatMapFunction<Iterable<Tuple2<Character, Double>>, Object>() {

            @Override
            public Iterator<Object> call(Iterable<Tuple2<Character, Double>> tuple2s) throws Exception {
                int count = Iterables.size(tuple2s);
                List results = new ArrayList<>();
                for (Tuple2<Character, Double> tuple2 : tuple2s) {
                   // System.out.println(tuple2._1+": "+tuple2._2 / count);
                    results.add(new Tuple2<>(tuple2._1, tuple2._2 / count));
                }
                return results.iterator();
            }
        });
        objectJavaRDD.foreach(x-> System.out.println(x));
        
        
        //最后一部根据ruduceBykey进行计算，但是flatmap之后只能进行reduce操作，因此需要进行mapToPair操作，转换成键值对的形式。也是在上一步直接使用flatmaptoPair函数一步到位。
        JavaPairRDD<String, Double> objectObjectJavaPairRDD = objectJavaRDD.mapToPair(
                s -> {
                    String[] split = s.toString().replaceAll("\\(","").replaceAll("\\)","")
                                    .split(",");
                    return new Tuple2<String,Double>(split[0], Double.parseDouble(split[1]));
                }
        );
        JavaPairRDD<String, Double> stringDoubleJavaPairRDD = objectObjectJavaPairRDD.reduceByKey(new Function2<Double, Double, Double>() {
            @Override
            public Double call(Double v1, Double v2) throws Exception {
                return v1 + v2;
            }
        }).mapValues(new Function<Double, Double>() {

            @Override
            public Double call(Double v1) throws Exception {
                return 0.15 + 0.85 * v1;
            }
        });
        stringDoubleJavaPairRDD.foreach(x-> System.out.println(x));
    }
}

```

输出结果

```
[(A,D), (B,A), (C,A), (C,B), (D,A), (D,C)]
[(A,1.0), (B,1.0), (C,1.0), (D,1.0)]

(A,D)
(B,A)
(C,A)
(C,B)
(D,A)
(D,C)

(A,1.0)
(B,1.0)
(C,1.0)
(D,1.0)

(B,(A,1.0))
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
(D,(A,1.0))
(D,(C,1.0))

(B,[(A,1.0)])
(A,[(D,1.0)])
(C,[(A,1.0), (B,1.0)])
(D,[(A,1.0), (C,1.0)])

(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)

(B,0.575)
(A,1.8499999999999999)
(C,0.575)
(D,1.0)
```

缺点分析：

上述算法经历了join,flatMap,reduceBykey,mapValue等过程。

在spark内部会为每个操作都生成对应的RDD对象。整个PageRank算法的RDD依赖取决于迭代循环的次数的大小。每一次迭代中有两次Shuffle过程。

![image-20220706193125674](https://s2.loli.net/2022/07/07/TiByrnlVF2acEqG.png)

改良第二版

![image-20220706193102728](https://s2.loli.net/2022/07/07/ecEKGZjs8U7QoLl.png)

```java
package spark_example;

import avro.shaded.com.google.common.collect.Iterables;
import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public class PageRank_2 {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);

        Tuple2[] links = {Tuple2.apply('A', 'D'), Tuple2.apply('B', 'A'),
                Tuple2.apply('C', 'A'), Tuple2.apply('C', 'B')
                , Tuple2.apply('D', 'A'), Tuple2.apply('D', 'C')};
        List<Tuple2> tuple2s = Arrays.asList(links);

        JavaRDD<Tuple2> linksRdd = sc.parallelize(tuple2s,2);

        Tuple2[] ranks = {Tuple2.apply('A', 1.0), Tuple2.apply('B', 1.0),
                Tuple2.apply('C', 1.0), Tuple2.apply('D', 1.0)};

        List<Tuple2> tuple2s1 = Arrays.asList(ranks);

        JavaRDD<Tuple2> RanksRdd = sc.parallelize(tuple2s1,2);


        JavaPairRDD<Character, Character> linkPairRDD = linksRdd.mapToPair(s -> {
            return new Tuple2<Character, Character>((Character) s._1, (Character) s._2);
        });
        JavaPairRDD<Character, Character> linksPartion = linkPairRDD.partitionBy(new HashPartitioner(2));
        JavaPairRDD<Character, Double> rankPairRDD = RanksRdd.mapToPair(s -> {
            return new Tuple2<Character, Double>((Character) s._1, (Double) s._2);
        });
//        linkPairRDD.foreach(x -> System.out.println(x));
//        System.out.println();
//        rankPairRDD.foreach(x -> System.out.println(x));
        linksPartion.foreach(x-> System.out.println(x));
        JavaPairRDD<Character, Tuple2<Character, Double>> join = linksPartion.join(rankPairRDD);
//        System.out.println();
        join.foreach(x -> System.out.println(x));
        JavaPairRDD<Character, Iterable<Tuple2<Character, Double>>> characterIterableJavaPairRDD = join.groupByKey();
        //characterIterableJavaPairRDD.foreach(x -> System.out.println(x));
        JavaRDD<Object> objectJavaRDD = characterIterableJavaPairRDD.values().flatMap(
                new FlatMapFunction<Iterable<Tuple2<Character, Double>>, Object>() {
                    @Override
                    public Iterator<Object> call(Iterable<Tuple2<Character, Double>> tuple2s) throws Exception {
                        int count = Iterables.size(tuple2s);
                        List results = new ArrayList<>();
                        for (Tuple2<Character, Double> tuple2 : tuple2s) {
                            // System.out.println(tuple2._1+": "+tuple2._2 / count);
                            results.add(new Tuple2<>(tuple2._1, tuple2._2 / count));
                        }
                        return results.iterator();
                    }
                }
        );
        objectJavaRDD.foreach(x-> System.out.println(x));
        JavaPairRDD<String, Double> objectObjectJavaPairRDD = objectJavaRDD.mapToPair(
                s -> {
                    String[] split = s.toString().replaceAll("\\(","").replaceAll("\\)","")
                            .split(",");
                    //System.out.println(split[0]+"::"+split[1]);
                    return new Tuple2<String,Double>(split[0], Double.parseDouble(split[1]));
                }
        );
        JavaPairRDD<String, Double> stringDoubleJavaPairRDD = objectObjectJavaPairRDD.reduceByKey(new Function2<Double, Double, Double>() {
            @Override
            public Double call(Double v1, Double v2) throws Exception {
                return v1 + v2;
            }
        }).mapValues(new Function<Double, Double>() {

            @Override
            public Double call(Double v1) throws Exception {
                return 0.15 + 0.85 * v1;
            }
        });
        stringDoubleJavaPairRDD.foreach(x-> System.out.println(x));
    }
}

```

```
第一个分区：
(B,A)
(D,A)
(D,C)
第二个分区：
(A,D)
(C,A)
(C,B)
###################
(B,(A,1.0))
(D,(A,1.0))
(D,(C,1.0))

2.
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
###################
(A,1.0)
(A,0.5)
(C,0.5)
2.
(D,1.0)
(A,0.5)
(B,0.5)
###################
(B,0.575)
(D,1.0)
2.
(A,1.8499999999999999)
(C,0.575)

```



![微信图片_20220706211855](https://s2.loli.net/2022/07/07/jTsouw5SpQfLZHJ.jpg)

![7e36d32631b442dbd508d0c3eaedf13](https://s2.loli.net/2022/07/07/s87DVhvnNMPIY1c.jpg)



![image-20220711151508039](C:\Users\ZWH\AppData\Roaming\Typora\typora-user-images\image-20220711151508039.png)



![image-20220711151535951](https://s2.loli.net/2022/07/11/KcBCbuWXjIzOHLt.png)