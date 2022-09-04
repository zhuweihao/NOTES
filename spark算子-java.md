# Java版本

## 与 Spark 链接

Spark 3.3.0 支持 [lambda 表达式](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) 来简洁地编写函数，否则可以使用 [org.apache.spark.api.java.function](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/function/package-summary.html)包中的类。

请注意，在 Spark 2.2.0 中删除了对 Java 7 的支持。

要使用 Java 编写 Spark 应用程序，您需要添加对 Spark 的依赖项。Spark 可通过 Maven Central 在以下位置获得：

```
groupId = org.apache.spark
artifactId = spark-core_2.12
version = 3.3.0
```

此外，如果您希望访问 HDFS 集群，则需要 `hadoop-client`为您的 HDFS 版本添加依赖项。

```
groupId = org.apache.hadoop
artifactId = hadoop-client
version = <your-hdfs-version>
```

最后，您需要将一些 Spark 类导入您的程序。添加以下行：

```
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;
```

## 初始化 Spark

Spark 程序必须做的第一件事是创建一个[JavaSparkContext](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaSparkContext.html)对象，它告诉 Spark 如何访问集群。要创建一个`SparkContext`，您首先需要构建一个包含有关您的应用程序信息的[SparkConf对象。](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/SparkConf.html)

```
SparkConf conf = new SparkConf().setAppName(appName).setMaster(master);
JavaSparkContext sc = new JavaSparkContext(conf);
```

该`appName`参数是您的应用程序在集群 UI 上显示的名称。 `master`是[Spark、Mesos 或 YARN 集群 URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或者是在本地模式下运行的特殊“本地”字符串。实际上，当在集群上运行时，您不想`master`在程序中硬编码，而是[启动应用程序`spark-submit`](https://spark.apache.org/docs/latest/submitting-applications.html)并在那里接收它。但是，对于本地测试和单元测试，您可以传递“local”以在进程内运行 Spark。

## 弹性分布式数据集 (RDD)

Spark 围绕*弹性分布式数据集*(RDD) 的概念展开，RDD 是可以并行操作的元素的容错集合。有两种方法可以创建 RDD：*并行化* 驱动程序中的现有集合，或引用外部存储系统中的数据集，例如共享文件系统、HDFS、HBase 或任何提供 Hadoop InputFormat 的数据源。

## 并行集合

并行化集合是通过调用驱动程序中现有的`JavaSparkContext`'方法来创建的。复制集合的元素以形成可以并行操作的分布式数据集。例如，这里是如何创建一个包含数字 1 到 5 的并行化集合：`parallelize``Collection`

```
List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
JavaRDD<Integer> distData = sc.parallelize(data);
```

创建后，分布式数据集 ( `distData`) 可以并行操作。例如，我们可能会调用`distData.reduce((a, b) -> a + b)`将列表中的元素相加。我们稍后将描述对分布式数据集的操作。

并行集合的一个重要参数是将数据集切割成的*分区数量。*Spark 将为集群的每个分区运行一个任务。通常，您希望集群中的每个 CPU 有 2-4 个分区。通常，Spark 会尝试根据您的集群自动设置分区数。但是，您也可以通过将其作为第二个参数传递给`parallelize`（例如`sc.parallelize(data, 10)`）来手动设置它。注意：代码中的某些地方使用术语切片（分区的同义词）来保持向后兼容性。

## 外部数据集

Spark 可以从 Hadoop 支持的任何存储源创建分布式数据集，包括您的本地文件系统、HDFS、Cassandra、HBase、[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。Spark 支持文本文件、[SequenceFiles](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)和任何其他 Hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

可以使用`SparkContext`'`textFile`方法创建文本文件 RDD。此方法获取文件的 URI（机器上的本地路径，或`hdfs://`,`s3a://`等 URI）并将其作为行集合读取。这是一个示例调用：

```java
JavaRDD<String> distFile = sc.textFile("data.txt");
```

创建后，`distFile`可以通过数据集操作对其进行操作。例如，我们可以使用`map`and`reduce`操作将所有行的大小相加，如下所示`distFile.map(s -> s.length()).reduce((a, b) -> a + b)`：

使用 Spark 读取文件的一些注意事项：

- 如果使用本地文件系统上的路径，则该文件也必须可在工作节点上的同一路径上访问。将文件复制到所有工作人员或使用网络安装的共享文件系统。
- Spark 的所有基于文件的输入法，包括`textFile`，也支持在目录、压缩文件和通配符上运行。例如，您可以使用`textFile("/my/directory")`、`textFile("/my/directory/*.txt")`和`textFile("/my/directory/*.gz")`。
- 该`textFile`方法还采用可选的第二个参数来控制文件的分区数。默认情况下，Spark 为文件的每个块创建一个分区（在 HDFS 中，块默认为 128MB），但您也可以通过传递更大的值来请求更大数量的分区。请注意，您的分区不能少于块。

除了文本文件，Spark 的 Java API 还支持其他几种数据格式：

- `JavaSparkContext.wholeTextFiles`让您读取包含多个小文本文件的目录，并将每个文件作为 (filename, content) 对返回。这与 `textFile`形成对比，后者将在每个文件中每行返回一条记录。
- 对于[SequenceFiles](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)，使用 SparkContext 的`sequenceFile[K, V]`方法 where`K`和`V`are 文件中键和值的类型。这些应该是 Hadoop 的[Writable](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/io/Writable.html)接口的子类，例如[IntWritable](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/io/IntWritable.html)和[Text](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/io/Text.html)。
- 对于其他 Hadoop InputFormats，您可以使用该`JavaSparkContext.hadoopRDD`方法，该方法采用任意`JobConf`输入格式类、键类和值类。设置这些参数的方式与使用输入源的 Hadoop 作业相同。您还可以使用`JavaSparkContext.newAPIHadoopRDD`基于“新” MapReduce API ( `org.apache.hadoop.mapreduce`) 的 InputFormats。
- `JavaRDD.saveAsObjectFile`并`JavaSparkContext.objectFile`支持以由序列化 Java 对象组成的简单格式保存 RDD。虽然这不如 Avro 等专用格式高效，但它提供了一种简单的方法来保存任何 RDD。

## RDD 操作

RDD 支持两种类型的操作：*转换*（从现有数据集创建新数据集）和*操作*（在对数据集运行计算后将值返回给驱动程序）。例如，`map`是一种通过函数传递每个数据集元素并返回表示结果的新 RDD 的转换。另一方面，`reduce`是一种使用某个函数聚合 RDD 的所有元素并将最终结果返回给驱动程序的操作（尽管也有`reduceByKey`返回分布式数据集的并行操作）。

Spark 中的所有转换都是*惰性*的，因为它们不会立即计算结果。相反，他们只记得应用于某些基础数据集（例如文件）的转换。仅当操作需要将结果返回给驱动程序时才计算转换。这种设计使 Spark 能够更高效地运行。例如，我们可以意识到通过创建的数据集`map`将在 a 中使用，`reduce`并且仅将结果返回`reduce`给驱动程序，而不是更大的映射数据集。

默认情况下，每个转换后的 RDD 可能会在您每次对其运行操作时重新计算。但是，您也可以使用(or ) 方法将 RDD*持久*化到内存中，在这种情况下，Spark 会将元素保留在集群上，以便下次查询时更快地访问它。还支持在磁盘上持久化 RDD，或跨多个节点复制。`persist cache`

### 基本

为了说明 RDD 的基础知识，考虑下面的简单程序：

```
JavaRDD<String> lines = sc.textFile("data.txt");
JavaRDD<Integer> lineLengths = lines.map(s -> s.length());
int totalLength = lineLengths.reduce((a, b) -> a + b);
```

第一行定义了来自外部文件的基本 RDD。此数据集未加载到内存中或以其他方式执行：`lines`仅是指向文件的指针。第二行定义`lineLengths`为`map`转换的结果。同样，由于懒惰，不会立即`lineLengths` 计算*。*最后，我们运行`reduce`，这是一个动作。此时，Spark 将计算分解为在不同机器上运行的任务，每台机器都运行它的映射部分和本地归约，只将其答案返回给驱动程序。

如果我们以后还想`lineLengths`再次使用，我们可以添加：

```
lineLengths.persist(StorageLevel.MEMORY_ONLY());
```

before the `reduce`，这将导致`lineLengths`在第一次计算后保存在内存中。

### 将函数传递给 Spark

Spark 的 API 在很大程度上依赖于在驱动程序中传递函数来在集群上运行。[在 Java 中，函数由org.apache.spark.api.java.function](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/function/package-summary.html)包中接口的类表示 。有两种方法可以创建此类函数：

- 在您自己的类中实现 Function 接口，可以作为匿名内部类或命名的内部类，并将其实例传递给 Spark。
- 使用[lambda 表达式](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) 来简洁地定义一个实现。

虽然本指南的大部分内容为了简洁而使用 lambda 语法，但很容易以长格式使用所有相同的 API。例如，我们可以将上面的代码编写如下：

```java
JavaRDD<String> lines = sc.textFile("data.txt");
JavaRDD<Integer> lineLengths = lines.map(new Function<String, Integer>() {
  public Integer call(String s) { return s.length(); }
});
int totalLength = lineLengths.reduce(new Function2<Integer, Integer, Integer>() {
  public Integer call(Integer a, Integer b) { return a + b; }
});
```

或者，如果内联编写函数很笨拙：

```java
class GetLength implements Function<String, Integer> {
  public Integer call(String s) { return s.length(); }
}
class Sum implements Function2<Integer, Integer, Integer> {
  public Integer call(Integer a, Integer b) { return a + b; }
}
JavaRDD<String> lines = sc.textFile("data.txt");
JavaRDD<Integer> lineLengths = lines.map(new GetLength());
int totalLength = lineLengths.reduce(new Sum());
```

**请注意，Java 中的匿名内部类也可以访问封闭范围内的变量，只要它们被标记为`final`。Spark 会将这些变量的副本发送到每个工作节点，就像它为其他语言所做的那样。**

### 了解闭包

关于 Spark 的难点之一是在跨集群执行代码时了解变量和方法的范围和生命周期。修改其范围之外的变量的 RDD 操作可能是一个常见的混淆源。在下面的示例中，我们将查看`foreach()`用于递增计数器的代码，但其他操作也可能出现类似问题。

#### 例子

考虑下面简单的 RDD 元素总和，根据是否在同一个 JVM 中执行，它的行为可能会有所不同。一个常见的例子是在`local`模式 ( `--master = local[n]`) 下运行 Spark 与将 Spark 应用程序部署到集群（例如通过 spark-submit 到 YARN）：



```
int counter = 0;
JavaRDD<Integer> rdd = sc.parallelize(data);

// Wrong: Don't do this!!
rdd.foreach(x -> counter += x);

println("Counter value: " + counter);
```

#### 本地与集群模式

上述代码的行为未定义，可能无法按预期工作。为了执行作业，Spark 将 RDD 操作的处理分解为任务，每个任务都由一个 executor 执行。在执行之前，Spark 会计算任务的**闭包**。闭包是那些必须对执行程序可见的变量和方法才能在 RDD 上执行其计算（在这种情况下`foreach()`）。这个闭包被序列化并发送给每个执行器。

发送给每个执行程序的闭包中的变量现在是副本，因此，当在函数中引用**计数器**`foreach`时，它不再是驱动程序节点上的**计数器。**驱动程序节点的内存中仍有一个**计数器**，但执行程序不再可见！执行者只能看到来自序列化闭包的副本。因此，**counter**的最终值仍然为零，因为对**counter**的所有操作都引用了序列化闭包中的值。

在本地模式下，在某些情况下，`foreach`函数实际上会在与驱动程序相同的 JVM 中执行，并且会引用相同的原始**计数器**，并且可能会实际更新它。

为了确保在这些场景中定义明确的行为，应该使用[`Accumulator`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators). Spark 中的累加器专门用于提供一种机制，用于在集群中的工作节点之间拆分执行时安全地更新变量。本指南的累加器部分更详细地讨论了这些内容。

一般来说，闭包——像循环或本地定义的方法这样的结构，不应该被用来改变一些全局状态。Spark 不定义或保证从闭包外部引用的对象的突变行为。一些这样做的代码可能在本地模式下工作，但这只是偶然，这样的代码在分布式模式下不会像预期的那样运行。如果需要一些全局聚合，请改用累加器。

#### 打印 RDD 的元素

`rdd.foreach(println)`另一个常见的习惯用法是尝试使用or打印出 RDD 的元素`rdd.map(println)`。在一台机器上，这将生成预期的输出并打印所有 RDD 的元素。但是，在`cluster`模式下，`stdout`执行程序调用的输出现在正在写入执行`stdout`程序，而不是驱动程序上的输出，因此`stdout`驱动程序不会显示这些！要打印驱动程序上的所有元素，可以使用`collect()`首先将 RDD 带到驱动程序节点的方法：`rdd.collect().foreach(println)`. 但是，这可能会导致驱动程序耗尽内存，因为`collect()`会将整个 RDD 提取到单个机器上；如果您只需要打印 RDD 的几个元素，更安全的方法是使用`take()`: `rdd.take(100).foreach(println)`。

### 使用键值对

虽然大多数 Spark 操作适用于包含任何类型对象的 RDD，但少数特殊操作仅适用于键值对的 RDD。最常见的是分布式“shuffle”操作，例如通过键对元素进行分组或聚合。

在 Java 中，键值对使用 Scala 标准库中的[scala.Tuple2类表示。](http://www.scala-lang.org/api/2.12.15/index.html#scala.Tuple2)您可以简单地调用`new Tuple2(a, b)`来创建一个元组，然后使用 和 访问它的`tuple._1()`字段`tuple._2()`。

键值对的 RDD 由 [JavaPairRDD](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html)类表示。`map`您可以使用特殊版本的操作（如 `mapToPair`和）从 JavaRDD 构造 JavaPairRDD `flatMapToPair`。JavaPairRDD 将具有标准 RDD 函数和特殊键值函数。

例如，以下代码使用`reduceByKey`键值对的操作来计算文件中每行文本出现的次数：

```
JavaRDD<String> lines = sc.textFile("data.txt");
JavaPairRDD<String, Integer> pairs = lines.mapToPair(s -> new Tuple2(s, 1));
JavaPairRDD<String, Integer> counts = pairs.reduceByKey((a, b) -> a + b);
```

例如，我们还可以使用`counts.sortByKey()`来按字母顺序对对进行排序，最后 `counts.collect()`将它们作为对象数组带回驱动程序。

**注意：**在键值对操作中使用自定义对象作为键时，必须确保自定义`equals()`方法伴随匹配`hashCode()`方法。[有关完整的详细信息，请参阅Object.hashCode() 文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--)中概述的合同。



## （一）概述

https://spark.apache.org/docs/latest/rdd-programming-guide.html

算子从功能上可以分为Transformations转换算子和Action行动算子。转换算子用来做数据的转换操作，比如map、flatMap、reduceByKey等都是转换算子，这类算子通过懒加载执行。行动算子的作用是触发执行，比如foreach、collect、count等都是行动算子，只有程序运行到行动算子时，转换算子才会去执行。

本文将介绍开发过程中常用的转换算子和行动算子，Spark代码基于Java编写，前置代码如下：

```java
public class SparkTransformationTest {
    public static void main(String[] args) {
        // 前置准备
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
    }
}
```



## （二）转换算子

### map

map(func)：通过函数func传递的每个元素，返回一个新的RDD。

```java
JavaRDD<Object> map = javaRdd.map((Function<String, Object>) 
        item -> "new" + item);
map.foreach(x -> System.out.println(x));
```


返回一个新的RDD，数据是

```
newa
newb
newc
newd
newe
```

### filter

filter(func)：筛选通过func处理后返回 true 的元素，返回一个新的RDD。

```java
JavaRDD<String> filter = javaRdd.filter(item -> item.equals("a") || item.equals("b"));
filter.foreach(x -> System.out.println(x));
```


返回的新RDD数据是a和b。

### flatMap

flatMap(func)：类似于 map，但每个输入项可以映射到 0 个或更多输出项。

```java
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a,b", "c,d,e", "f,g"));
JavaRDD<String> flatMap = javaRdd.flatMap((FlatMapFunction<String, String>) 
        s -> Arrays.asList(s.split(",")).iterator());
flatMap.foreach(x -> System.out.println(x));
```


入参只有3个，经过flatMap映射后返回了长度为7的RDD。

入参只有3个，经过flatMap映射后返回了长度为7的RDD。

### mapPartitions

mapPartitions(func)：类似于map，但该函数是在RDD每个partition上单独运行，因此入参会是Iterator<Object>。

```java
SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a,b", "c,d,e", "f,g"));
        JavaRDD<String> mapPartitions = javaRdd.mapPartitions((FlatMapFunction<Iterator<String>, String>) stringIterator -> {
            ArrayList<String> list = new ArrayList<>();
            while (stringIterator.hasNext()) {
                list.add(stringIterator.next());
            }
            return list.iterator();
        });
        mapPartitions.foreach(x -> System.out.println(x));
    }
```

除了是对Iterator进行处理之外，其他的都和map一样。

```shell
a,b
c,d,e
f,g
```

### mapPartitionsWithIndex



### sample



### union

union(otherDataset)：返回一个新数据集，包含两个数据集合的并集。

```java
 public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
        JavaRDD<String> newJavaRdd = sc.parallelize(Arrays.asList("1", "2", "3", "4", "5"));
        JavaRDD<String> unionRdd = javaRdd.union(newJavaRdd);
        unionRdd.foreach(x-> System.out.println(x));
    }
```

输出

```
a
b
c
d
e
1
2
3
4
5
```

### intersection

intersection(otherDataset)：返回一个新数据集，包含两个数据集合的交集。

```java
public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
        JavaRDD<String> newJavaRdd = sc.parallelize(Arrays.asList("a", "b", "3", "4", "5"));
        JavaRDD<String> intersectionRdd = javaRdd.intersection(newJavaRdd);
        intersectionRdd.foreach(x-> System.out.println(x));
    }
```

输出

```
a
b
```

### groupByKey

groupByKey ([ numPartitions ])：在 (K, V) 对的数据集上调用时，返回 (K, Iterable) 对的数据集，可以传递一个可选numPartitions参数来设置不同数量的任务。

这里需要了解Java中的另外一种RDD，JavaPairRDD。JavaPairRDD是一种key-value类型的RDD，groupByKey就是针对JavaPairRDD的API。

```java
JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "a:2", "b:1", "c:3"));
JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
    String[] split = s.split(":");
    return new Tuple2<>(split[0], Integer.parseInt(split[1]));
});
JavaPairRDD<String, Iterable<Integer>> groupByKey = javaPairRDD.groupByKey();
groupByKey.foreach(x -> System.out.println(x._1()+x._2()));
```


最终输出结果：

```
a[1, 2]
b[1]
c[3]
```



### reduceByKey

reduceByKey(func, [numPartitions])：在 (K, V) 对的数据集上调用时，返回 (K, V) 对的数据集，其中每个键的值使用给定的 reduce 函数聚合。和groupByKey不同的地方在于reduceByKey对value进行了聚合处理。

reduceByKey
reduceByKey(func, [numPartitions])：在 (K, V) 对的数据集上调用时，返回 (K, V) 对的数据集，其中每个键的值使用给定的 reduce 函数func聚合。和groupByKey不同的地方在于reduceByKey对value进行了聚合处理。

```java
JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "a:2", "b:1", "c:3"));
JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
    String[] split = s.split(":");
    return new Tuple2<>(split[0], Integer.parseInt(split[1]));
});
JavaPairRDD<String, Integer> reduceRdd = javaPairRDD.reduceByKey((Function2<Integer, Integer, Integer>) (x, y) -> x + y);
reduceRdd.foreach(x -> System.out.println(x._1()+x._2()));
```


最终输出结果：

```
a3
b1
c3
```

### sortByKey

```java
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

import java.util.Arrays;

public class sortedbykey {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        JavaPairRDD<String, Integer> sortByKey = javaPairRDD.sortByKey();
        sortByKey.foreach(x-> System.out.println(x._1()+x._2()));
    }
}

```

```shell
a1
a2
a3
b1
c3
```

### join

When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through `leftOuterJoin`, `rightOuterJoin`, and `fullOuterJoin`.

```java
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

import java.util.Arrays;

public class join {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        JavaRDD<String> rdd1 = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3" ));
        JavaPairRDD<String, Integer> javaPairRDD1 = rdd1.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        JavaPairRDD<String, Tuple2<Integer, Integer>> join = javaPairRDD.join(javaPairRDD1);
        join.foreach(x-> System.out.println(x._1()+x._2()));
    }
}

```

结果：笛卡尔积的形式

```shell
a(1,1)
a(2,1)
a(3,1)
b(1,1)
c(3,3)
```

### cogroup

When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called `groupWith`.

```
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

import java.util.Arrays;

public class cogroup {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        JavaRDD<String> rdd1 = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3" ));
        JavaPairRDD<String, Integer> javaPairRDD1 = rdd1.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        JavaPairRDD<String, Tuple2<Iterable<Integer>, Iterable<Integer>>> cogroup = javaPairRDD.cogroup(javaPairRDD1);
        cogroup.foreach(x-> System.out.println(x._1()+x._2()));
        
    }

}


```

```
a([1, 2, 3],[1])
b([1],[1])
c([3],[3])
```

### cartesian

When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements).

```java
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.util.Arrays;

public class cartesian {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        JavaRDD<Integer> rdd1 = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        JavaPairRDD<String, Integer> cartesian = rdd.cartesian(rdd1);
        cartesian.foreach(x-> System.out.println(x._1()+x._2()));
    }
}

```

输出

```
a:11
a:12
a:13
a:14
a:15
b:11
b:12
b:13
b:14
b:15
c:31
c:32
c:33
c:34
c:35
a:21
a:22
a:23
a:24
a:25
a:31
a:32
a:33
a:34
a:35
```

### aggregateByKey

`aggregateByKey(zeroValue)(seqOp, combOp, [numPartitions])`：`aggregateByKey`这个算子相比上面这些会复杂很多，主要参数有`zeroValue、seqOp、combOp，numPartitions`可选。

aggregateByKey
aggregateByKey(zeroValue)(seqOp, combOp, [numPartitions])：aggregateByKey这个算子相比上面这些会复杂很多，主要参数有zeroValue、seqOp、combOp，numPartitions可选。

zeroValue是该算子设置的初始值，seqOp函数是将rdd中的value值和zeroValue进行处理，combOp是将相同key的数据进行处理。

```java
JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "a:2", "b:1", "c:3"));
JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
    String[] split = s.split(":");
    return new Tuple2<>(split[0], Integer.parseInt(split[1]));
});
JavaPairRDD<String, Integer> aggregateRdd = javaPairRDD.aggregateByKey(1,
        // seqOp函数中的第一个入参是 zeroValue，第二个入参是rdd的value，相同key的只会加一次zerovalue;
        (Function2<Integer, Integer, Integer>) (v1, v2) -> v1 + v2,
        // combOp函数是对同一个key的value进行处理，这里对相同key的value进行相加
        (Function2<Integer, Integer, Integer>) (v1, v2) -> v1 + v2);
aggregateRdd.foreach(x -> System.out.println(x._1()+":"+x._2()));
```


最终输出结果如下：

最终输出结果如下：

```
a:4
b:2
c:4
```



## （三）行动算子

### reduce

reduce(func)：使用函数func聚合数据集的元素（它接受两个参数并返回一个）。
下面这段代码对所有rdd进行相加：

（三）行动算子
reduce
reduce(func)：使用函数func聚合数据集的元素（它接受两个参数并返回一个）。
下面这段代码对所有rdd进行相加：

```java
String reduce = javaRdd.reduce((Function2<String, String, String>) (v1, v2) -> {
    System.out.println(v1 + ":" + v2);
    return v1+v2;
});
System.out.println("result:"+reduce);
```


最终结果如下，从结果可以看出，每次对v1都是上一次reduce运行之后的结果：

```
a:b
ab:c
abc:d
abcd:e
result:abcde
```



### collect()

collect()：将driver中的所有元素数据通过集合的方式返回，适合小数据量的场景，大数据量会导致内存溢出。

```java
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
List<String> collect = javaRdd.collect();
System.out.println(collect);
```

```
[a, b, c, d, e]
```



### count()

count()：返回一个RDD中元素的数量。

```
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
long count = javaRdd.count();
```



```
5
```



### first()

first()：返回第一个元素。

```
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
String first = javaRdd.first();
```

### take()

take(n)：返回前N个元素，take(1)=first()。

```java
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
List<String> take = javaRdd.take(3);
```

```
[a, b, c]
```

### takeOrdered()

takeOrdered(n, [ordering])：返回自然排序的前N个元素，或者指定排序方法后的前N个元素。首先写一个排序类。

```
public class MyComparator implements Comparator<String>, Serializable {
    @Override
    public int compare(String o1, String o2) {
        return o2.compareTo(o1);
    }
}
```


接着是调用方式：

```java
JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d", "e"));
List<String> take = javaRdd.takeOrdered(3, new MyComparator());
```

### 实例

```java
 public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> javaRdd = sc.parallelize(Arrays.asList("a", "d", "c", "f", "e"));
        List<String> strings = javaRdd.takeOrdered(4, new myComparator());
        System.out.println(strings);
    }
    static class  myComparator implements Comparator<String>, Serializable{

        @Override
        public int compare(String o1, String o2) {
            return o1.compareTo(o2);
        }
    }
```

输出

```
[a, c, d, e]
```

## takeSample

 这个方法中有三个参数：参数1:withReplace：抽样的时候应该是否放回，会根据这个采用不同抽样器计算概率，如果是true 会采用PoissonBounds抽样器，false会采用BiunomialBounds采样器；参数2:抽样的数量；参数3:随机种子，这个是固定多次抽样返回的结果，如果我们指定，那么即使我们执行多次后抽样的结果是一样的，即是根据随机种子来决定的；如果多次抽样的随机种子是一样的，那么结果都是一样的.


```java
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.util.Arrays;
import java.util.List;

public class takeSample {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        List<String> strings = rdd.takeSample(false, 2, 3);
        System.out.println(strings);
        rdd.foreach(x-> System.out.println(x));
        List<String> strings1 = rdd.takeSample(true, 2, 3);
        System.out.println(strings1);
        rdd.foreach(x-> System.out.println(x));


    }
}
```

```shell
[c:3, a:2]
a:1
b:1
c:3
a:2
a:3
[a:3, b:1]
a:1
b:1
c:3
a:2
a:3
```

### saveAsTextFile

Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file.

### saveAsSequenceFile()

Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc).

### SaveAsObjectFile()

Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using `SparkContext.objectFile()`.

### countByKey

Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key.

```
package spark_Suanzi;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Map;

public class CountBykey {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("a:1", "b:1", "c:3","a:2","a:3" ));
        JavaPairRDD<String, Integer> javaPairRDD = rdd.mapToPair(s -> {
            String[] split = s.split(":");
            return new Tuple2<>(split[0], Integer.parseInt(split[1]));
        });
        Map<String, Long> stringLongMap = javaPairRDD.countByKey();
        System.out.println(stringLongMap);
    }
}

```

```
{a=3, b=1, c=1}
```



### foreach()

Run a function *func* on each element of the dataset. This is usually done for side effects such as updating an [Accumulator](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators) or interacting with external storage systems.
**Note**: modifying variables other than Accumulators outside of the `foreach()` may result in undefined behavior. See [Understanding closures ](https://spark.apache.org/docs/latest/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka)for more details.



## （四）总结

Spark的开发可以用Java或者Scala，Spark本身使用Scala编写，具体使用哪种语言进行开发需要根据项目情况考虑时间和学习成本。具体的API都可以在Spark官网查询：https://spark.apache.org/docs/2.3.0/rdd-programming-guide.html

# 问题

spark的集群分区，概念等。