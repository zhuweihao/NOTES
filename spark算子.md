seatunnel、datax打包形式

打包运行

打包结构

assemble文档



#### Spark任务提交

##### 任务分析

![image-20220628152526658](https://s2.loli.net/2022/06/28/6S1CrEAsX7JHlcx.png)

##### 重要角色

Driver（驱动器）

Spark 的驱动器是执行开发程序中的 main 方法的进程。它负责开发人员编写的用来创建 SparkContext、创建 RDD，以及进行 RDD 的转化操作和行动操作代码的执行。如果你是用 spark shell，那么当你启动 Spark shell 的时候，系统后台自启了一个 Spark 驱动器程序，就是在 Spark shell 中预加载的一个叫作 sc 的 SparkContext 对象。如果驱动器程序终止，那么 Spark 应用也就结束了。主要负责： 

- 把用户程序转为任务 
- 跟踪 Executor 的运行状况
- 为执行器节点调度任务 

- UI 展示应用运行状况 

Executor（执行器） 

Spark Executor 是一个工作进程，负责在 Spark 作业中运行任务，任务间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点上继续运行。主要负责： 

- 负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程； 
- 通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式
  存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加
  速运算。 

# Spark核心编程

Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- Resilient Distributed Datasets (RDDs)：弹性分布式数据集
- Shared Variables：共享变量
  - Broadcast Variables：广播变量，分布式共享只读变量
  - Accumulators：累加器，分布式共享只写变量

##  RDD概述

相关论文：Resilient Distributed Datasets: A Fault-tolerant Abstraction for In-memory Cluster Computing

论文链接：https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf

---

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据抽象。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、可以并行操作的元素的容错集合。 

### 特点

- 弹性

  - 存储的弹性：内存与磁盘的自动切换
  - 容错的弹性：数据丢失可以自动恢复
  - 计算的弹性：计算出错重试机制
  - 分片的弹性：可根据需要重新分片

- 分布式：数据存储的大数据集群不同节点上

- 数据集：RDD封装了计算逻辑，并不保存数据

- 数据抽象：RDD是一个抽象类，需要子类具体实现

- 分区：RDD 逻辑上是分区的，每个分区的数据是抽象存在的，计算的时候会通过一个 compute函数得到每个分区的数据。如果 RDD 是通过已有的文件系统构建，则 compute 函数是读取指定文件系统中的数据，如果 RDD 是通过其他 RDD 转换而来，则 compute 函数是执行转换逻辑将其他 RDD 的数据进行转换。 

- 只读：RDD 是只读的，要想改变 RDD 中的数据，只能在现有的 RDD 基础上创建新的 RDD。

  由一个 RDD 转换到另一个 RDD，可以通过丰富的操作算子实现，不再像 MapReduce那样只能写 map 和 reduce 了。 

  RDD 的操作算子包括两类，一类叫做 transformations，它是用来将 RDD 进行转化，构建 RDD 的血缘关系；另一类叫做 actions，它是用来触发 RDD 的计算，得到 RDD 的相关计算结果或者将 RDD 保存的文件系统中。

### 核心属性

![image-20220628164150389](https://s2.loli.net/2022/06/28/YNm8vpnKcLSVFPR.png)

- 分区列表

一组分区（Partition），即数据集的基本组成单位; 

RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性

![image-20220628164649687](https://s2.loli.net/2022/06/28/uUfv3CYatGTl7Vk.png)

- 分区计算函数

spark在计算时，是使用分区函数对每一个分区进行计算

![image-20220628164912097](https://s2.loli.net/2022/06/28/UGQIMvmxFiSzVXa.png)

- RDD之间的依赖关系

RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系。

![image-20220628165455863](https://s2.loli.net/2022/06/28/RQy2Sae9Mrx4FsI.png)

- 分区器

 一个 Partitioner，即 RDD 的分片函数; 

![image-20220628165730329](https://s2.loli.net/2022/06/28/7gs3OxRNK1zIJum.png)

- 一个列表，计算每个分区的优先位置

![image-20220628165745577](https://s2.loli.net/2022/06/28/KZPlNQLFGAHzVRs.png)

## RDD编程

在Spark中，RDD被表示为对象，通过对象上的方法调用来对RDD进行转换。经过一系列的transformations定义 RDD之后，就可以调用actions触发RDD的计算，action可以是向应用程序返回结果(count, collect 等)，或者是向存储系统保存数据(saveAsTextFile等)。在Spark中，只有遇到action，才会执行RDD的计算(即延迟计算)，这样在运行时可以通过管道的方式传输多个转换。 

要使用Spark，开发者需要编写一个Driver程序，它被提交到集群以调度运行Worker，如下图所示。Driver中定义了一个或多个RDD，并调用RDD上的action，Worker则执行RDD分区计算任务。 

### RDD创建

在Spark中创建RDD的方式可以分为三种：

- 从集合（内存）中创建RDD

  - parallelize
  - makeRDD

- 从外部存储创建RDD

  包括本地的文件系统，还有所有 Hadoop 支持的数据集，比如 HDFS、Cassandra、HBase

- 从其他RDD创建

### RDD算子

RDD算子主要分为两类：转换算子、行动算子

- 转换算子：功能的补充和封装，将旧的RDD包装成新的RDD
- 行动算子：触发任务的调度和作业的执行

```scala
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
val sparkContext = new SparkContext(sparkConf)

sparkContext.stop()
```

#### 转换算子-Transformations

RDD根据数据处理方式的不同将算子分为：Value类型、双Value类型和Key-Value类型

##### Value类型

###### map

返回一个新的 RDD，该RDD由每一个输入元素经过func函数转换后组成

在进行处理时数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。

```scala
  /**
   * Return a new RDD by applying a function to all elements of this RDD.
   */
  def map[U: ClassTag](f: T => U): RDD[U] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[U, T](this, (_, _, iter) => iter.map(cleanF))
  }
```

```scala
object TransformMap {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
    val sparkContext = new SparkContext(sparkConf)

    //TODO 算子-map
    val rdd = sparkContext.makeRDD(List(1, 2, 3, 4))
    //转换函数
    def mapFunction(num: Int): Int = {
      num * 2
    }
    //val mapRDD:RDD[Int] = rdd.map(mapFunction)
    //val mapRDD:RDD[Int] = rdd.map((num:Int)=>{num*2})
    //函数代码逻辑只有一行花括号省略
    //val mapRDD:RDD[Int] = rdd.map((num:Int)=>num*2)
    //参数类型可以自动推断出来时类型可以省略
    //val mapRDD:RDD[Int] = rdd.map((num)=>num*2)
    //参数列表中只有一个参数时小括号可以省略
    //val mapRDD:RDD[Int] = rdd.map(num=>num*2)
    //参数在逻辑中只出现一次且按照顺序出现
    val mapRDD: RDD[Int] = rdd.map(_ * 2)
    mapRDD.collect().foreach(println)
    sparkContext.stop()
  }
}
```

小功能：从日志数据apache.log中获取用户请求URL资源路径

```scala
//TODO 算子-map 实现提取用户请求URL资源路径
val rdd=sparkContext.textFile("Spark/src/main/resources/apache.log")
val mapRDD:RDD[String]=rdd.map(
    line => {
        val datas=line.split(" ")
        datas(6)
    }
)
mapRDD.collect().foreach(println)
```

体现RDD方法的并行计算

```scala
//rdd在计算一个分区内数据时是串行有序的
//不同分区数据计算是无序的
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4),1)
val mapRDD=rdd.map(
  num=>{
    println("--------"+num)
    num
  }
)
val mapRDD1=mapRDD.map(
  num=>{
    println("+++++++++"+num)
    num
  }
)
mapRDD1.collect()
```

###### 如何使mapPartions不内存溢出

###### mapPartitions

将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，如过滤数据

类似于 map，但独立地在 RDD 的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是 Iterator[T] => Iterator[U]。假设有N个元素，有M个分区，那么map的函数的将被调用 N 次,而mapPartitions被调用 M 次,一个函数一次处理所有分区。

```scala
  /**
   * Return a new RDD by applying a function to each partition of this RDD.
   *
   * `preservesPartitioning` indicates whether the input function preserves the partitioner, which
   * should be `false` unless this is a pair RDD and the input function doesn't modify the keys.
   */
  def mapPartitions[U: ClassTag](
      f: Iterator[T] => Iterator[U],
      preservesPartitioning: Boolean = false): RDD[U] = withScope {
    val cleanedF = sc.clean(f)
    new MapPartitionsRDD(
      this,
      (_: TaskContext, _: Int, iter: Iterator[T]) => cleanedF(iter),
      preservesPartitioning)
  }
```

如下代码的输出结果中会有两行”--------“，因为有两个分区

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4),2)
//mapPartitions：以分区为单位进行数据转换操作，但是会将整个分区的数据加载到内存进行引用，每次处理一个分区的数据，这个分区的数据处理完后，原RDD中分区的数据才能释放，这可能会导致内存溢出
val mapPartitionsRDD=rdd.mapPartitions(
    iterator=>{
        println("------")
        //iterator.filter(_==2)
        iterator.map(_*2)
    }
)
mapPartitionsRDD.collect().foreach(println)
```

map()和mapPartitions()的区别

- map算子是分区内一个一个数据的执行，类似于串行操作，而mapPartitions算子是以分区为单位进行批处理操作
- map算子主要目的是将数据源中的数据进行转换和改变，但是不会减少或者增多数据。mapPartitions算子需要传递一个迭代器，返回一个迭代器，可以增加或者减少数据。
- map算子串行操作，性能较低。mapPartitions类似于批处理操作，性能较高，当内存空间较大的时候建议使用 mapPartition()，以提高处理效率。 

小功能：获取每个数据分区的最大值

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val mapPartitionsRDD = rdd.mapPartitions(
  iterator => {
    List(iterator.max).iterator
  }
)
mapPartitionsRDD.collect().foreach(println)
```

###### mapPartitionsWithIndex

```scala
def mapPartitionsWithIndex[U: ClassTag](
    f: (Int, Iterator[T]) => Iterator[U],
    preservesPartitioning: Boolean = false): RDD[U]
```

将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理可以进行任意的处理，在处理的同时可以获取当前分区索引

类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是(Int, Interator[T]) => Iterator[U]

```scala
//获取第二个分区的数据
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val mpirdd = rdd.mapPartitionsWithIndex(
    (index, iterator) => {
        if (index == 1) {
            iterator
        } else {
            //Nil为空集合
            Nil.iterator
        }
    }
)
mpirdd.collect().foreach(println)
```

###### flatMap

```scala
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]
```

将处理的数据进行扁平化后再进行映射处理，所以算子也称为扁平映射

类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素）

```scala
val rdd=sparkContext.makeRDD(List(
    "Hello World","Hello Spark"
))
val flatRDD = rdd.flatMap(
    s=>{
        s.split(" ")
    }
)
flatRDD.collect().foreach(println)
```

小功能：将List（List（1，2），3，List（4，5））进行扁平化操作

```scala
val rdd = sparkContext.makeRDD(List(
    List(1, 2), 3, List(4, 5)
))
val flatRDD = rdd.flatMap(
    data => {
        data match {
            case list: List[_] => list
            case num => List(num)
        }
    }
)
flatRDD.collect().foreach(println)
```

###### glom

```scala
def glom(): RDD[Array[T]]
```

将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

```java
val rdd:RDD[Int] = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val glomRDD:RDD[Array[Int]] = rdd.glom()
glomRDD.collect().foreach(data=>println(data.mkString(",")))
```

小功能：计算所有分区最大值求和（分区内取最大值，分区间最大值求和）

```scala
val rdd: RDD[Int] = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val glomRDD: RDD[Array[Int]] = rdd.glom()
val maxRDD: RDD[Int] = glomRDD.map(_.max)
println(maxRDD.collect().sum)
```

###### groupby

```scala
def groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
```

将数据根据指定的规则进行分组，分区默认不变，但是数据会被打乱重新组合，这样的操作称之为shuffle。极限情况下，数据可能被分在同一个分区中

分组，按照传入函数的返回值（key）进行分组。将相同的返回值（key）对应的数据放入一个迭代器（组）。 

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val groupRDD: RDD[(Int, Iterable[Int])] = rdd.groupBy(
  num => {
    println("-----------")
    num % 2
  }
)
groupRDD.collect().foreach(println)
```

![image-20220629151214593](https://s2.loli.net/2022/06/29/zPTtoFIQKuCVk1G.png)

```scala
val rdd1 = sparkContext.makeRDD(List("Hello", "World", "Spark", "Hadoop"), 2)
val groupRDD1 = rdd1.groupBy(_.charAt(0))
groupRDD1.collect().foreach(println)
```

![image-20220629151803247](https://s2.loli.net/2022/06/29/JiB5EYnMDGbFSKC.png)

注意分组和分区没有必然关系

```scala
val rdd1 = sparkContext.makeRDD(List("Hello", "World", "Spark", "Hadoop"), 2)
val groupRDD1 = rdd1.groupBy(_.charAt(0))
val groupRDD2 = groupRDD1.mapPartitionsWithIndex(
  (index, iterator) => {
    iterator.map(
      data => {
        (index, data)
      }
    )
  }
)
groupRDD2.collect().foreach(println)
```

![image-20220629152855898](https://s2.loli.net/2022/06/29/jL3ZMDAGixfR4ON.png)

小功能：从日志数据apache.log中获取每个时间段访问量

```scala
val rdd = sparkContext.textFile("Spark/src/main/resources/apache.log")

val timeRDD = rdd.map(
  line => {
    val datas = line.split(" ")
    val time = datas(3)
    val simpleDateFormat = new SimpleDateFormat("dd/MM/yyyy:HH:mm:ss")
    val date = simpleDateFormat.parse(time)
    val simpleDateFormat1 = new SimpleDateFormat("dd/MM/yyyy:HH")
    val hour = simpleDateFormat1.format(date)
    (hour, 1)
  }
).groupBy(_._1)
timeRDD.map {
  case (hour, iter) => {
    (hour, iter.size)
  }
}.collect.foreach(println)
```

###### filter

```scala
def filter(f: T => Boolean): RDD[T]
```

过滤。返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成

当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4))
val filterRDD = rdd.filter(num => num % 2 != 0)
filterRDD.collect().foreach(println)
```

小功能：从日志数据apache.log中获取2015年5与17日的请求路径

```scala
val rdd = sparkContext.textFile("Spark/src/main/resources/apache.log")

rdd.filter(
  line => {
    val strings = line.split(" ")
    strings(3).startsWith("17/05/2015")
  }
).map(
  line => {
    val datas = line.split(" ")
    datas(6)
  }
).collect().foreach(println)
```

###### sample

```scala
  /**
   * Return a sampled subset of this RDD.
   *
   * @param withReplacement can elements be sampled multiple times (replaced when sampled out)
   * @param fraction expected size of the sample as a fraction of this RDD's size
   *  without replacement: probability that each element is chosen; fraction must be [0, 1]
   *  with replacement: expected number of times each element is chosen; fraction must be greater
   *  than or equal to 0
   * @param seed seed for the random number generator
   *
   * @note This is NOT guaranteed to provide exactly the fraction of the count
   * of the given [[RDD]].
   */
  def sample(
      withReplacement: Boolean,
      fraction: Double,
      seed: Long = Utils.random.nextLong): RDD[T] = {
    require(fraction >= 0,
      s"Fraction must be nonnegative, but got ${fraction}")

    withScope {
      require(fraction >= 0.0, "Negative fraction value: " + fraction)
      if (withReplacement) {
        new PartitionwiseSampledRDD[T, T](this, new PoissonSampler[T](fraction), true, seed)
      } else {
        new PartitionwiseSampledRDD[T, T](this, new BernoulliSampler[T](fraction), true, seed)
      }
    }
  }
```

根据指定的规则从数据集中抽取数据

withRepacement表示抽取出的数据是否放回，true为有放回的抽样，false为无放回的抽样，seed用于指定随机数生成器种子

withReplacement为false时，fraction表示每个元素被抽到的概率，[0,1]

withReplacement为true时，fraction表示每个元素期望被抽取到的次数，大于等于零

如果不传递seed，那么将默认使用系统当前时间

注：泊松分布，伯努利分布

```scala
rdd.sample(
  false, 0.4, 1
).collect().foreach(println)

rdd.sample(
    true, 1.1, 1
).collect().foreach(println)
```

###### distinct

```scala
def distinct(): RDD[T]
def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
```

对源 RDD 进行去重后返回一个新的 RDD。默认情况下，只有 8 个并行任务来操
作，但是可以传入一个可选的 numTasks 参数改变它。 

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4))
val distinctRDD = rdd.distinct()
distinctRDD.collect().foreach(println)
```

distinct的主要实现逻辑如下

```scala
//(1,null)(2,null)(3,null)(4,null)(1,null)(2,null)(3,null)(4,null)
//(1,null)(1,null)
//(null,null)=>null
//(1,null)
map(x => (x, null)).reduceByKey((x, _) => x, numPartitions).map(_._1)
```

###### coalesce

```scala
def coalesce(numPartitions: Int, shuffle: Boolean = false,
             partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
            (implicit ord: Ordering[T] = null)
    : RDD[T]
```

缩减分区数，用于大数据集过滤后，提高小数据集的执行效率

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4, 5, 6), 3)
val newRDD = rdd.coalesce(2)
//输出为1，2
println(newRDD.mapPartitionsWithIndex(
  (index, iter) => {
    if (index == 0) {
      iter
    } else {
      Nil.iterator
    }
  }
).collect().mkString(","))
//输出为3，4，5，6
println(newRDD.mapPartitionsWithIndex(
  (index, iter) => {
    if (index == 1) {
      iter
    } else {
      Nil.iterator
    }
  }
).collect().mkString(","))
```

coalesce默认情况下不会将分区中数据打乱重新组合

这种情况下的缩减分区可能会导致数据倾斜，如果想要数据均衡，可以进行shuffle处理

```scala
val newRDD = rdd.coalesce(2, true)
```

###### repartition

```scala
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
  coalesce(numPartitions, shuffle = true)
}
```

根据分区数，重新通过网络随机洗牌所有数据。

注意repartition实现是通过coalesce实现的，在此说明

- coalesce重新分区，可以扩大可以也可以缩小，如果选择扩大分区，必须进行shuffle过程，否则没有意义
- repartition实际上时调用了coalesce，进行shuffle

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4, 5, 6), 2)
val newRDD = rdd.repartition(3)
for (i <- 0 to 2) {
  println(newRDD.mapPartitionsWithIndex(
    (index, iter) => {
      if (index == i) {
        iter
      } else {
        Nil.iterator
      }
    }
  ).collect().mkString(","))
}
```

######  sortBy

```scala
/**
 * Return this RDD sorted by the given key function.
 */
def sortBy[K](
    f: (T) => K,
    ascending: Boolean = true,
    numPartitions: Int = this.partitions.length)
    (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T] = withScope {
  this.keyBy[K](f)
      .sortByKey(ascending, numPartitions)
      .values
}
```

使用 func 先对数据进行处理，按照处理后的数据比较结果排序，默认为正序，ascending=false时可以进行降序排列。 分区数量不变，但是存在shuffle操作

```scala
val rdd = sparkContext.makeRDD(List(5, 1, 2, 6, 3, 4)， 2)
rdd.sortBy(num => num).collect().foreach(println)
```

```scala
val rdd = sparkContext.makeRDD(List(("1", 1), ("11", 2), ("2", 3)), 2)
rdd.sortBy(t=>t._1).collect().foreach(println)
```

```scala
val rdd = sparkContext.makeRDD(List(("1", 1), ("11", 2), ("2", 3)), 2)
rdd.sortBy(t=>t._1.toInt).collect().foreach(println)
```

##### 双Value类型

```scala
val rdd1 = sparkContext.makeRDD(List(1, 2, 3, 4))
val rdd2 = sparkContext.makeRDD(List(1, 2, 5, 6))

println(rdd1.intersection(rdd2).collect().mkString(","))
println(rdd1.union(rdd2).collect().mkString(","))
println(rdd1.subtract(rdd2).collect().mkString(","))
println(rdd1.zip(rdd2).collect().mkString(","))
println(rdd1.cartesian(rdd2).collect().mkString(","))
```

###### intersection

```scala
def intersection(other: RDD[T]): RDD[T]
```

对源RDD和参数RDD求交集后返回一个新的RDD 

###### union

```scala
def union(other: RDD[T]): RDD[T]
```

对源 RDD 和参数 RDD 求并集后返回一个新的 RDD 

###### substract

```scala
def subtract(other: RDD[T]): RDD[T]
```

计算差的一种函数，去除两个 RDD 中相同的元素，不同的元素保留下来 

###### zip

```scala
def zip[U: ClassTag](other: RDD[U]): RDD[(T, U)]
```

将两个 RDD 组合成 Key/Value 形式的 RDD,这里默认两个 RDD 的 partition 数量以及元素数量都相同，否则会抛出异常。

```
//分区数量不一致
Can't zip RDDs with unequal numbers of partitions: List(2, 4)
//元素数量不一致
org.apache.spark.SparkException: Can only zip RDDs with same number of elements in each partition
```

###### cartesian

```scala
def cartesian[U: ClassTag](other: RDD[U]): RDD[(T, U)]
```

创建两个 RDD，计算两个 RDD 的笛卡尔积 

##### Key-Value类型

###### partitionBy

```scala
def partitionBy(partitioner: Partitioner): RDD[(K, V)]
```

将数据按照Partitioner重新进行分区。Spark默认的分区器是HashPartitioner

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4), 2)
val mapRDD = rdd.map((_, 1))
/**
 * RDD=>PairRDDFunctions
 * RDD类型的mapRDD调用了PairRDDFunctions里面的方法
 * 隐式转换（二次编译）
 * class RDD里面有一个RDD伴生对象，里面存在一个隐式函数可以实现上述类型转换
 * implicit def rddToPairRDDFunctions[K, V](rdd: RDD[(K, V)])
 * (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null): PairRDDFunctions[K, V] = {
 * new PairRDDFunctions(rdd)
 * }
 */
    val newRDD = mapRDD.partitionBy(new HashPartitioner(4))
    for (i <- 0 to 3) {
      println(newRDD.mapPartitionsWithIndex(
        (index, iter) => {
          if (index == i) {
            iter
          } else {
            Nil.iterator
          }
        }
      ).collect().mkString(","))
    }
```

如果重分区的分区器和当前RDD的分区器是一致（类型和分区数量）的话就不进行分区， 否则会生成ShuffleRDD，即会产生 shuffle 过程。 

Spark中的分区器

![image-20220629222902526](https://s2.loli.net/2022/06/29/uNZn6Y5VCWwBOAl.png)

###### reduceByKey

```scala
def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)]
```

在一个(K,V)的RDD上调用，返回一个(K,V)的 RDD，使用指定的reduce函数，将相同key的值聚合到一起，reduce任务的个数可以通过第二个可选的参数来设置。

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("a", 2), ("a", 3), ("b", 4),
))
//reduceByKey:相同的key的数据进行value数据的聚合操作
//scala语言中一般的聚合操作都是两两聚合，spark基于scala开发，所以他的聚合也是两两聚合
//[1,2,3]=>[3,3]=>[6]
//如果key的数据只有一个，则不会参与运算
val reduceRDD = rdd.reduceByKey((x: Int, y: Int) => {
  println(s"x=${x},y=${y}")
  x + y
})
reduceRDD.collect().foreach(println)
```

![image-20220629231134973](https://s2.loli.net/2022/06/29/RaqhDBwIon5YKNP.png)

###### groupByKey

```scala
def groupByKey(): RDD[(K, Iterable[V])]
def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]
def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]
```

groupByKey 也是对每个 key 进行操作，但只生成一个 seq

```scala
val rdd = sparkContext.makeRDD(List(
    ("a", 1), ("a", 2), ("a", 3), ("b", 4),
))
val newRDD: RDD[(String, Iterable[Int])] = rdd.groupByKey()
newRDD.collect().foreach(println)
val value: RDD[(String, Iterable[(String, Int)])] = rdd.groupBy(_._1)
value.collect().foreach(println)
```

![image-20220629230141293](https://s2.loli.net/2022/06/29/wHhctZGfaeJ9TL5.png)

![image-20220629230153268](https://s2.loli.net/2022/06/29/qONC4L3Q8c5rKGx.png)



reduceByKey和groupByKey的区别

- reduceByKey：按照key进行聚合，在shuffle之前有combine（预聚合）操作，返回结果是RDD[K,V]
- groupByKey：按照key进行分组，直接进行shuffle



![image-20220629232311403](https://s2.loli.net/2022/06/29/lR5J2vPx8KBLOdE.png)



![image-20220629232616266](https://s2.loli.net/2022/06/29/lo1KGRBUsZeLEA7.png)

从shuffle的角度：reduceByKey和groupByKey都存在shuffle操作，但是reduceByKey可以在shuffle前对分区内相同key的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而groupByKey知识进行分组，不存在数据量减少，故reduceByKey性能更高

从功能的角度：reduceByKey器时包含分组和聚合的功能，groupByKey只能分组，不能聚合，所以在分组聚合的场合下，推荐使用reduceByKey，如果仅仅是分组而不需要聚合，那么则使用groupByKey。

###### aggregateByKey

```scala
  def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U,
      combOp: (U, U) => U): RDD[(K, U)] = self.withScope {
    aggregateByKey(zeroValue, defaultPartitioner(self))(seqOp, combOp)
  }
```

```scala
  def aggregateByKey[U: ClassTag](zeroValue: U, numPartitions: Int)(seqOp: (U, V) => U,
      combOp: (U, U) => U): RDD[(K, U)] = self.withScope {
    aggregateByKey(zeroValue, new HashPartitioner(numPartitions))(seqOp, combOp)
  }
```

```scala
  def aggregateByKey[U: ClassTag](zeroValue: U, partitioner: Partitioner)(seqOp: (U, V) => U,
      combOp: (U, U) => U): RDD[(K, U)] = self.withScope {
    // Serialize the zero value to a byte array so that we can get a new clone of it on each key
    val zeroBuffer = SparkEnv.get.serializer.newInstance().serialize(zeroValue)
    val zeroArray = new Array[Byte](zeroBuffer.limit)
    zeroBuffer.get(zeroArray)

    lazy val cachedSerializer = SparkEnv.get.serializer.newInstance()
    val createZero = () => cachedSerializer.deserialize[U](ByteBuffer.wrap(zeroArray))

    // We will clean the combiner closure later in `combineByKey`
    val cleanedSeqOp = self.context.clean(seqOp)
    combineByKeyWithClassTag[U]((v: V) => cleanedSeqOp(createZero(), v),
      cleanedSeqOp, combOp, partitioner)
  }
```

将数据根据不同的规则进行分区内计算和分区间计算

分区内取最大值，分区间求和

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("a", 2), ("a", 3), ("a", 4)
),2)
//函数编程中，接受多个参数的函数都可以转化为接受单个参数的函数，这个转化过程就叫柯里化
/**
 * aggregateByKey存在函数柯里化，有两个参数列表
 * 第一个参数列表，需要传递一个参数，表示为初始值，主要用于当碰见第一个key的时候，和value进行分区内计算
 * 第二个参数列表，第一个参数表示分区内计算规则，第二个参数表示分区间计算规则
 */
val value: RDD[(String, Int)] = rdd.aggregateByKey(0)(
  (x, y) => math.max(x, y),
  (x, y) => x + y
)
value.collect().foreach(println)
```

![image-20220630093604921](https://s2.loli.net/2022/06/30/FgR8uVCxlJybfYd.png)

aggregateByKey最终返回的数据结果应该和初始值的类型保持一致

小功能：获取相同key的数据的平均值

```scala
val newRDD: RDD[(String, (Int, Int))] = rdd.aggregateByKey((0, 0))(
  (tuple, value) => {
    (tuple._1 + value, tuple._2 + 1)
  },
  (tuple1, tuple2) => {
    (tuple1._1 + tuple2._1, tuple1._2 + tuple2._2)
  }
)
val resultRDD: RDD[(String, Int)] = newRDD.mapValues {
  case (num, cnt) => {
    num / cnt
  }
}
resultRDD.collect().foreach(println)
```

 ![image-20220630100115381](https://s2.loli.net/2022/06/30/FlayBvXxDjhCIoT.png)



###### foldByKey

```scala
def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]
```

aggregateByKey 的简化操作，seqOp 和 combOp 相同 ，即分区内和分区间计算规则相同

###### combineByKey

```scala
def combineByKey[C](
    createCombiner: V => C,
    mergeValue: (C, V) => C,
    mergeCombiners: (C, C) => C,
    numPartitions: Int): RDD[(K, C)] = self.withScope {
  combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners, numPartitions)(null)
}
```

```scala
def combineByKey[C](
    createCombiner: V => C,
    mergeValue: (C, V) => C,
    mergeCombiners: (C, C) => C,
    partitioner: Partitioner,
    mapSideCombine: Boolean = true,
    serializer: Serializer = null): RDD[(K, C)] = self.withScope {
  combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners,
    partitioner, mapSideCombine, serializer)(null)
}
```

- createCombiner: combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没遇到过， 要么就和之前的某个元素的键相同。如果这是一个新的元素,combineByKey()会使用一个叫作 createCombiner()的函数来创建那个键对应的累加器的初始值 
- mergeValue: 如果这是一个在处理当前分区之前已经遇到的键，它会使用 mergeValue()方将该键的 累加器对应的当前值与这个新的值进行合并 
- mergeCombiners: 由于每个分区都是独立处理的， 因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器， 就需要使用用户提供的 mergeCombiners() 方法将各个分区的结果进行合并。 



```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("a", 2), ("b", 3),
  ("b", 4), ("b", 5), ("a", 6)
), 2)
//第一个参数：将相同key的第一个数据进行结构的转换
//第二个参数：分区内的计算规则
//第三个参数：分区间的计算规则
val newRDD: RDD[(String, (Int, Int))] = rdd.combineByKey(
  v => (v, 1),
  (tuple: (Int, Int), value) => {
    (tuple._1 + value, tuple._2 + 1)
  },
  (tuple1: (Int, Int), tuple2: (Int, Int)) => {
    (tuple1._1 + tuple2._1, tuple1._2 + tuple2._2)
  }
)
newRDD.collect().foreach(println)
val resultRDD: RDD[(String, Int)] = newRDD.mapValues {
  case (num, cnt) => {
    num / cnt
  }
}
resultRDD.collect().foreach(println)
```



```scala
//wordCount
rdd.reduceByKey(_ + _)
rdd.aggregateByKey(0)(_ + _, _ + _)
rdd.foldByKey(0)(_ + _)
rdd.combineByKey(v => v, (x: Int, y) => x + y, (x: Int, y: Int) => x + y)
```

上面四个方法的底层都是通过下面的函数实现的

```scala
def combineByKeyWithClassTag[C](
    createCombiner: V => C,
    mergeValue: (C, V) => C,
    mergeCombiners: (C, C) => C,
    partitioner: Partitioner,
    mapSideCombine: Boolean = true,
    serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)]
```

###### join

```scala
def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))]
```

在类型为（K，V）和（K，W）的RDD上调用，返回一个相同key对应的所有元素连接在一起的RDD，类型为（K，（V，W））

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("b", 2), ("c", 3), ("e", 8)
), 2)
val rdd1 = sparkContext.makeRDD(List(
  ("a", "4"), ("b", "5"), ("c", 6), ("d", 7), ("a", 3)
), 2)
val newRDD: RDD[(String, (Int, Any))] = rdd.join(rdd1)
newRDD.collect().foreach(println)
```

如果两个数据源中key没有匹配上，那么数据不会出现在结果中

如果两个数据源中key有多个相同的，会依次匹配，数据量会出现几何性增长，会导致性能降低，要谨慎使用

###### cogroup

```scala
def cogroup[W1, W2, W3](other1: RDD[(K, W1)],
    other2: RDD[(K, W2)],
    other3: RDD[(K, W3)],
    partitioner: Partitioner)
    : RDD[(K, (Iterable[V], Iterable[W1], Iterable[W2], Iterable[W3]))] = self.withScope {
  if (partitioner.isInstanceOf[HashPartitioner] && keyClass.isArray) {
    throw SparkCoreErrors.hashPartitionerCannotPartitionArrayKeyError()
  }
  val cg = new CoGroupedRDD[K](Seq(self, other1, other2, other3), partitioner)
  cg.mapValues { case Array(vs, w1s, w2s, w3s) =>
     (vs.asInstanceOf[Iterable[V]],
       w1s.asInstanceOf[Iterable[W1]],
       w2s.asInstanceOf[Iterable[W2]],
       w3s.asInstanceOf[Iterable[W3]])
  }
}
```

join底层是通过cogroup实现的

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("b", 2), ("c", 3), ("e", 8)
), 2)
val rdd1 = sparkContext.makeRDD(List(
  ("a", "4"), ("b", "5"), ("c", 6), ("d", 7), ("a", 3)
), 2)
val newRDD: RDD[(String, (Iterable[Int], Iterable[Any]))] = rdd.cogroup(rdd1)
newRDD.collect().foreach(println)
```

![image-20220630142601091](https://s2.loli.net/2022/06/30/AMjJF4bRGY2k9m7.png)

###### sortByKey

```scala
def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
    : RDD[(K, V)]
```

在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口，返回一个按照 key 进行排序的(K,V)的 RDD

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("f", 2), ("c", 3), ("e", 8)
), 2)
val value: RDD[(String, Int)] = rdd.sortByKey(false)
value.collect().foreach(println)
```



#### 行动算子-Actions

所谓的行动算子，其实就是触发作业（Job）执行的方法

底层代码调用的是环境对象的runJob方法，会创建ActiveJob，并提交执行

###### reduce

```scala
def reduce(f: (T, T) => T): T
```

通过func函数聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据。 

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4),2)
//分区一：1-2=-1，分区二：3-4=-1
//分区间：-1-（-1）=0
val i: Int = rdd.reduce(_ - _)
println(i)
```

###### collect

```scala
def collect(): Array[T] 
```

将不同分区的数据按照分区顺序采集到Driver端内存中，形成数组

```scala
val rdd = sparkContext.makeRDD(List(1, 2, 3, 4),2)
val ints: Array[Int] = rdd.collect()
println(ints.mkString(","))
```

###### count

```scala
def count(): Long = sc.runJob(this, Utils.getIteratorSize _).sum
```

返回RDD中元素的个数

```scala
val l: Long = rdd.count()
println(l)
```

###### first

```scala
def first(): T = withScope {
  take(1) match {
    case Array(t) => t
    case _ => throw SparkCoreErrors.emptyCollectionError()
  }
}
```

返回RDD中的第一个元素

```scala
val i: Int = rdd.first()
println(i)
```

###### take

```scala
def take(num: Int): Array[T] 
```

返回一个由RDD的前n个元素组成的数组

```scala
val ints: Array[Int] = rdd.take(3)
println(ints.mkString(","))
```

###### takeOrdered

```scala
def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]
```

返回该RDD排序后的前n个元素组成的数组

```scala
val ints1: Array[Int] = rdd.takeOrdered(3)
println(ints1.mkString(","))
```

###### aggregate

```scala
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U 
```

aggregate 函数将每个分区里面的元素通过 seqOp 和初始值进行聚合，然后用combine 函数将每个分区的结果和初始值(zeroValue)进行 combine 操作。这个函数最终返回的类型不需要和 RDD 中元素类型一致。 

```scala
val i1: Int = rdd.aggregate(1)(_ + _, _ + _)
println(i1)
```

注意：初始值会参与分区间计算

###### fold

```scala
def fold(zeroValue: T)(op: (T, T) => T): T
```

aggregate的简化操作，当分区内和分区间计算规则相同时，可以使用fold

```scala
println(rdd.fold(1)(_ + _))
```

###### countByKey



```scala
def countByKey(): Map[K, Long]
```

针对(K,V)类型的 RDD，返回一个(K,Int)的 map，表示每一个 key 对应的元素个数。 

```scala
val rdd = sparkContext.makeRDD(List(
  ("a", 1), ("a", 2), ("a", 3), ("b", 4)
), 2)
val value: collection.Map[(String, Int), Long] = rdd.countByValue()
println(value)
val value1: collection.Map[String, Long] = rdd.countByKey()
println(value1)
```

###### foreach

```scala
def foreach(f: T => Unit): Unit
```

在数据集的每一个元素上，运行函数 func 。 

```scala
//foreach Driver端内存集合的循环遍历方法
rdd.collect().foreach(println)
//foreach Executor端内存数据打印
rdd.foreach(println)
```

注：上面的两个foreach不是同一个方法，第二个才是本次讲的foreach算子，第一个是scala语言中Array的方法 



```scala
object foreach {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local").setAppName("Operator")
    val sparkContext = new SparkContext(sparkConf)
    val rdd = sparkContext.makeRDD(List(1, 2, 4, 3), 2)
    val user = new User()
    //闭包就是一个函数和与其相关的引用环境组合的一个整体(实体)。
    //例如，一个匿名函数 ，该函数引用到到函数外的变量x,那么该函数和变量x整体形成一个闭包
    //RDD算子中传递的函数（函数式编程）存在闭包操作就会进行检测，称为闭包检测，所传递的变量必须可以序列化
    rdd.foreach(
      num => {
        println("age=" + (user.age + num))
      }
    )
    sparkContext.stop()
  }

  //由于Driver端和Executor端发生通信，故User必须序列化，否则会报错NotSerializableException
  class User extends Serializable {
    var age: Int = 30
  }
  /*
  //样例类在编译时，会自动实现可序列化接口
  case class User() {
    var age: Int = 30
  }*/
}
```

###### save相关算子

```scala
//保存成text文件
rdd.saveAsTextFile("output")
//序列化成对象保存到文件
rdd.saveAsObjectFile("output")
//要求数据格式必须为K-V类型，保存成SequenceFile文件
rdd.saveAsSequenceFile("ouput")
```

#### 注意

RDD的方法和Scala集合对象的方法不一样

Scala集合对象的方法都是在同一个节点的内存中完成的

RDD的方法可以将计算逻辑发送到Executor端（分布式节点）执行

为了区分不同的处理效果，所以将RDD的方法称之为算子

RDD的方法外部的操作都是在Driver端执行的，而方法内部的逻辑代码是在Executor端执行

#### 案例

agent.log：时间戳，省份，城市，用户，广告，中间字段使用空格分隔

需求：统计出每一个省份中被点击数量排行的Top3的广告

```scala
object Case {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local").setAppName("Operator")
    val sparkContext = new SparkContext(sparkConf)
    
    //1.获取原始数据：时间戳，省份，城市，用户，广告
    val dataRDD: RDD[String] = sparkContext.textFile("Spark/src/main/resources/agent.log")
    //2.将原始数据进行结构的转换。方便统计
    //时间戳，省份，城市，用户，广告 =》 （（省份，广告），1）
    val mapRDD: RDD[((String, String), Int)] = dataRDD.map(
      line => {
        val datas: Array[String] = line.split(" ")
        ((datas(1), datas(4)), 1)
      }
    )
    //3.将转换结构后的数据进行分组聚合
    //（（省份，广告），1）=》（（省份，广告），sum）
    val reduceRDD: RDD[((String, String), Int)] = mapRDD.reduceByKey(_ + _)
    //4.将聚合的结果进行结构的转换
    //（（省份，广告），sum）=》（省份，（广告，sum））
    val mapRDD1: RDD[(String, (String, Int))] = reduceRDD.map {
      case ((prv, ad), sum) => {
        (prv, (ad, sum))
      }
    }
    //5.将转换结构后的数据根据省份进行分组
    //（省份，【（广告A，sumA）,（广告B，sumB）,（广告C，sumC）】）
    val groupRDD: RDD[(String, Iterable[(String, Int)])] = mapRDD1.groupByKey()
    //6.将分组后的数据进行组内排序（降序），取前三名
    val resultRDD: RDD[(String, List[(String, Int)])] = groupRDD.mapValues(
      iter => {
        iter.toList.sortBy(_._2)(Ordering.Int.reverse).take(3)
      }
    )
    //7.采集数据打印在控制台
    resultRDD.collect().foreach(println)
    
    sparkContext.stop()
  }
}
```





寻找案例

shuffle过程优化

 page

结合sparkUI拓扑流程图
