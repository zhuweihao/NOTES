计算向数据移动？

数据向计算移动？



# 相关问题

## Spark on Yarn

- yarn-client：Driver 程序运行在客户端，适用于交互、调试，希望立即看到 app 的输出
- yarn-cluster：Driver 程序运行在由 RM（ResourceManager）启动的 AP（ApplicationMaster） 适用于生产环境。

<img src="https://s2.loli.net/2022/07/21/Vvm6BuRJeogKdfq.png" alt="image-20220721235349304" style="zoom: 67%;" />

yarn运行流程

1. 用户向 YARN 中提交应用程序，其中包括 MRAppMaster 程序，启动 MRAppMaster 的命令，用户程序等。
2. ResourceManager 为该程序分配第一个 Container，并与对应的 NodeManager 通讯，要求它在这个 Container 中启动应用程序 ApplicationMaster。
3. ApplicationMaster 首先向 ResourceManager 注册，这样用户可以直接通过 ResourceManager查看应用程序的运行状态，然后将为各个任务申请资源，并监控它的运行状态，直到运行结束，重复 4 到 7 的步骤。
4. ApplicationMaster 采用轮询的方式通过 RPC 协议向 ResourceManager 申请和领取资源。
5. 一旦 ApplicationMaster 申请到资源后，便与对应的 NodeManager 通讯，要求它启动任务。
6. NodeManager 为任务设置好运行环境（包括环境变量、JAR 包、二进制程序等）后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。
7. 各个任务通过某个 RPC 协议向 ApplicationMaster 汇报自己的状态和进度，以让 ApplicationMaster随时掌握各个任务的运行状态，从而可以在任务败的时候重新启动任务。
8. 应用程序运行完成后，ApplicationMaster 向 ResourceManager 注销并关闭自己。



首先需要了解ApplicationMaster，ApplicationMaster实际上是特定计算框架的一个实例，每种计算框架都有自己独特的ApplicationMaster，负责与ResourceManager协商资源，并和NodeManager协同来执行和监控Container。

ApplicationMaster有如下功能：

- 初始化向ResourceManager报告自己的活跃信息的进程
- 计算应用程序的的资源需求。
-  将需求转换为YARN调度器可以理解的ResourceRequest。
-  与调度器协商申请资源
- 与NodeManager协同合作使用分配的Container。
- 跟踪正在运行的Container状态，监控它的运行。
- 对Container或者节点失败的情况进行处理，在必要的情况下重新申请资源。
  

任何一个yarn上运行的任务都必须有一个ApplicationMaster，而任何一个Spark任务都会有一个Driver，Driver就是运行SparkContext的进程，它会构建TaskScheduler和DAGScheduler，当然在Driver上你也可以做很多非Spark的事情，这些事情只会在Driver上面执行，而由SparkContext上牵引出来的代码则会由DAGScheduler分析，并形成Job和Stage交由TaskScheduler，再由TaskScheduler交由各Executor分布式执行。

yarn-client和yarn-cluster的主要区别在于，yarn-client的Driver运行在本地，而ApplicationMaster运行在yarn的一个节点上，他们之间进行远程通信，ApplicationMaster只负责资源申请和释放(当然还有DelegationToken的刷新)，然后等待Driver的完成；而yarn-cluster的Driver则运行在ApplicationMaster所在的container里，Driver和ApplicationMaster是同一个进程的两个不同线程，它们之间也会进行通信，ApplicationMaster同样等待Driver的完成，从而释放资源。

### yarn-client

优先运行的是Driver(我们写的应用代码就是入口)，然后在初始化SparkContext的时候，会作为client端向yarn申请ApplicationMaster资源，当ApplicationMaster运行后，它会向yarn注册自己并申请Executor资源，之后由本地Driver与其通信控制任务运行，而ApplicationMaster则时刻监控Driver的运行情况，如果Driver完成或意外退出，ApplicationMaster会释放资源并注销自己。所以在该模式下，如果运行spark-submit的程序退出了，整个任务也就退出了。

![img](https://s2.loli.net/2022/07/22/Nv7EsjBVHpKfZ5F.png)

### yarn-cluster

本地进程则仅仅只是一个client，它会优先向yarn申请ApplicationMaster资源运行ApplicationMaster，在运行ApplicationMaster的时候通过反射启动Driver(我们的应用代码)，在SparkContext初始化成功后，再向yarn注册自己并申请Executor资源，此时Driver与ApplicationMaster运行在同一个container里，是两个不同的线程，当Driver运行完毕，ApplicationMaster会释放资源并注销自己。所以在该模式下，本地进程仅仅是一个client，如果结束了该进程，整个Spark任务也不会退出，因为Driver是在远程运行的

![img](https://s2.loli.net/2022/07/22/uT9SnNlKzgkGswZ.png)

### 读取外部配置

参考：

[(22条消息) Spark 读取外部配置文件（各种提交模式、使用 typesafe.config）_空藍性忘的博客-CSDN博客_spark 读取外部配置文件](https://blog.csdn.net/weixin_40458255/article/details/110210614)

[提交带有Spark typesafe配置的应用程序属性文件 - IT屋-程序员软件开发技术分享社区 (it1352.com)](https://www.it1352.com/1848294.html)

---

yarn-cluster模式（未验证）

在提交任务时通过--files 指定配置文件

----

--files不适用于local模式和yarn-client，这两种模式配置方法如下

- --files配置项不变，将作业的根目录添加到classpath中

  ```
  spark2-submit ... \
      --conf spark.driver.extraClassPath=./  \
      --conf spark.executor.extraClassPath=./  \    // if you need to load config at executors
  ```

  

- 使用`config.file`系统属性来影响程序在哪里寻找配置文件

  ```
  spark2-submit ... \
      --conf spark.driver.extraJavaOptions=-Dconfig.file=./application.conf \
      --conf spark.executor.extraJavaOptions=-Dconfig.file=./application.conf \
  ```

-----

示例：

```
spark-submit --master local --conf spark.driver.extraJavaOptions=-Dconfig.file=./properties.properties --conf spark.executor.extraJavaOptions=-Dconfig.file=./properties.properties Spark-1.0-SNAPSHOT.jar

```



### 端口占用问题及解决办法

spark配置：[Configuration - Spark 3.3.0 Documentation (apache.org)](https://spark.apache.org/docs/latest/configuration.html)

Spark on Yarn：[Running Spark on YARN - Spark 3.3.0 Documentation (apache.org)](https://spark.apache.org/docs/latest/running-on-yarn.html#spark-properties)

---

本地模式下，每一个Spark任务都会占用一个SparkUI端口，默认为4040，如果被占用则依次递增端口重试。但是有个默认重试次数，为16次。16次重试都失败后，会放弃该任务的运行。

解决办法：

1. 配置spark.port.maxRetries。默认为16，可适当增大。

   ```bash
   --config spark.port.maxRetries=100
   ```

2. 配置spark.ui.port。通过该配置指定一个没有被占用的端口。

   ```bash
   --config spark.ui.port=12345
   ```

![image-20220731175751963](https://s2.loli.net/2022/07/31/s92XrI8uHgcbhdL.png)

![image-20220731175716798](https://s2.loli.net/2022/07/31/7ljOck8NGU4BqfZ.png)

----

相关问题：on yarn模式下，spark程序如果因为资源不足或其他原因在第一次提交时候失败，然后又会不断提交，导致过多的系统资源被无效占用。

可以使用如下配置限制重试次数

```bash
--conf spark.yarn.maxAppAttempts=1 
```

![image-20220731175908086](https://s2.loli.net/2022/07/31/Gb85FPlwY7rXOfS.png)

spark on yarn模式，虽然由Yarn负责启动和管理AM以及分配资源，但是Spark有自己的AM实现，当Executor运行起来后，任务的控制是由Driver负责的。而重试上，Yarn只负责AM的重试。

另外，在Spark对ApplicationMaster的实现里，Spark提供了参数 spark.yarn.max.executor.failures 来控制Executor的失败次数，当Executor的失败次数达到这个值的时候，整个Spark应该程序就失败了

![image-20220731180830284](https://s2.loli.net/2022/07/31/XKOksLGVD9alRbN.png)

在YARN配置中，有如下配置

![image-20220731180547592](https://s2.loli.net/2022/07/31/hu7aIqld5ig4yYS.png)

所以默认情况下，spark.yarn.maxAppAttempts的值为2，如果想不进行第二次重试，可以将改值设为1

### 查看日志

driver 、executor日志

1. 进入hadoop yarn控制台
2. 找到对应任务的id，进入任务详情页
3. 详情页中查看logs

![image-20220731183428308](https://s2.loli.net/2022/07/31/U9X3K7mJgPxHdbL.png)

### Spark中pipline计算模式

相关论文：Resilient Distributed Datasets: A Fault-tolerant Abstraction for In-memory Cluster Computing

论文链接：https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf

相关论文：Spark: Cluster Computing with Working Sets

论文链接：https://www.usenix.org/legacy/event/hotcloud10/tech/full_papers/Zaharia.pdf

----

参考链接：https://blog.csdn.net/Shockang/article/details/121984986

----

（有点抽象，但是看完下面的内容应该可以理解。）

#### 宽窄依赖划分

论文中提出，设计如何表达 RDDs 之间依赖的接口是一个非常有意思的问题。作者发现将依赖定义成两种类型就足够了：

- 窄依赖，表示父亲 RDDs 的一个分区最多被子 RDDs 一个分区所依赖。
- 宽依赖，表示父亲 RDDs 的一个分区可以被子 RDDs 的多个子分区所依赖。

比如，map 操作是一个窄依赖，join 操作是一个宽依赖操作（除非父亲 RDDs 已经被 hash 分区过），下图显示了其他的例子

![image-20220731215517902](https://s2.loli.net/2022/07/31/APgrJELMTi5xDov.png)

以下两个原因使的这种区别很有用

- 第一，窄依赖可以使得在集群中一个机器节点上面流水线（pipline）式执行所有父亲的分区数据，比如，我们可以将每一个元素应用了 map 操作后紧接着应用 filter 操作，与此相反，宽依赖需要父亲 RDDs 的所有分区数据准备好并且利用类似于 MapReduce 的操作将数据在不同的节点之间进行Shuffle。


- 第二， 窄依赖从一个失败节点中恢复是非常高效的，因为只需要重新计算相对应的父亲的分区数据就可以，而且这个重新计算是在不同的节点进行并行重计算的，与此相反，在一个含有宽依赖的血缘关系 RDDs 图中，一个节点的失败可能导致一些分区数据的丢失，但是我们需要重新计算父 RDD 的所有分区的数据。

  

#### job调度器

当一个用户对某个 RDD 调用了 action 操作（比如 count 或者 save）的时候调度器会检查这个 RDD 的血缘关系图，然后根据这个血缘关系图构建一个含有 stages 的有向无环图（DAG），最后按照步骤执行这个 DAG 中的 stages，如下图。每一个 stage 包含了尽可能多的带有窄依赖的 transformations 操作。

![image-20220731215534221](https://s2.loli.net/2022/07/31/2o5KbaXqtDusHUp.png)

上面的例子中，带有颜色的方形表示分区，黑色的是表示这个分区的数据存储在内存中，对 RDD G 调用 action 操作，我们根据宽依赖生成很多 stages，且将窄依赖的 transformations 操作放在 stage 中。在这个场景中，stage 1 的输出结果已经在内存中，所以我们开始运行 stage 2，然后是 stage 3。

这个 stage 的划分是根据需要 shuffle 操作的宽依赖或者任何可以切断对父亲 RDD 计算的某个操作（因为这些父亲 RDD 的分区已经计算过了）。然后调度器可以调度启动 tasks 来执行没有父亲 stage 的 stage（或者父亲 stage 已经计算好了的 stage），一直到计算完我们的最后的目标 RDD 。

调度器在分配 tasks 的时候是采用延迟调度来达到数据本地性的目的（说白了，就是数据在哪里，计算就在哪里）。如果某个分区的数据在某个节点上的内存中，那么将这个分区的计算发送到这个机器节点中。如果某个 RDD 为它的某个分区提供了这个数据存储的位置节点，则将这个分区的计算发送到这个节点上。

对于宽依赖（比如 shuffle 依赖），我们将中间数据写入到节点的磁盘中以利于从错误中恢复，这个和 MapReduce 将 map 后的结果写入到磁盘中是很相似的。

只要一个任务所在的 stage 的父亲 stage 还是有效的话，那么当这个 task 失败的时候，我们就可以在其他的机器节点中重新跑这个任务。如果一些 stages 变的无效的话（比如因为一个 shuffle 过程中 map 端的一个输出结果丢失了），我们需要重新并行提交没有父亲 stage 的 stage（或者父亲 stage 已经计算好了的 stage）的计算任务。虽然备份 RDD 的血缘关系图比较容易，但是不能容忍调度器调度失败的场景。

虽然目前 Spark 中所有的计算都是响应 driver 程序中调用的 action 操作，但是我们也是需要尝试在集群中调用 lookup 操作，这种操作是根据 key 来随机访问已经 hash 分区过的 RDD 所有元素以获取相应的 value。

在这种场景中，如果一个分区没有计算的话，那么 task 需要将这个信息告诉调度器。

#### pipline计算模式

![img](https://s2.loli.net/2022/07/31/nN3aAklJIpr4hcY.png)

- Spark的pipeLine的计算模式，就是来一条数据然后计算一条数据，把所有的逻辑走完，然后落地。而MapReduce是计算完落地，然后在计算，然后再落地到磁盘或内存，最后数据是落在计算节点上，这也是spark比Mapreduce快的原因。
- 管道中的数据在对RDD进行持久化的时候会进行落地，即shuffle时会进行数据落地



验证代码：

```scala
object pipline {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local").setAppName("SparkPipeline")
    val sc = new SparkContext(conf)
    val rdd = sc.makeRDD(Array(1, 2, 3, 4))
    //执行filter
    val filterRdd = rdd.filter {
      x => {
        println("fliter********" + x)
        true
      }
    }
    //执行map
    filterRdd.map {
      x => {
        println("map--------" + x)
        x
      }
    }.count() //触发执行
    sc.stop()
  }
}
```

输出结果：从结果中可以看出，每读一条数据，都会经过filter和map这个过程，读完数据后都是一条条地并行计算，不用等，比如说RDD中的某一条数据执行ilter后，马上执行map，不用等rdd中其它数据都执行完fliter.

![image-20220731223124479](https://s2.loli.net/2022/07/31/KXUMruPaZRzBtgn.png)





## Kafka偏移量维护

kafka生产者每次生产消息时，都是往对应partition的文件中追加写入，而消息的被读取状态是由consumer来维护的，所以每个partition中offset一般都是连续递增的（如果开启了压缩，因为对旧数据的merge会导致不连续）被读取的消息并不会删除，所以每次都是追加写入顺序读写，具备很好的吞吐量。
consumer在消费消息后，向broker中有个专门维护每个consumer的offset的topic生产一条消息，记录自己当前已读的消息的offset+1的值作为新的offset的消息。当然在旧版本的实现是在zookeeper上有个节点存放这个offset，当时后面考虑性能问题，kafka改到了topic里，同时可以自由配置使用zookeeper还是使用topic

------

kafka的broker是无状态的，整个过程中伴随由zookeeper的协调参与，一般是不同broker存储了不同partition或副本数据，当存在多个副本时，从哪个broker读取数据时由zookeeper决定的，一般会由一台kafka作为leader(被读取)，如果该kafka不可用时，zookeeper切换到别的broker，因为broker在zookeeper上维护一个 /broker/ids/{id}的临时节点，如果kafka不可用，该节点也会被删除，kafka集群会根据该节点的信息，切换被读取的kafka

-----

每次consumer消费消息时，kafka都会根据该topic下对于该consumer记录的offset，传递对应消息，当offset不存在时，根据auto.offset.reset配置的值，会有几种不同策略

- earliest：无指定的offset时，从头开始消费
- latest：无提交的offset时，消费该分区下最新产生的消息
- none：topic不存在指定的offset，则抛出异常

------

对kafka整个集群设置偏移量，适合测试环境，丢弃整个消息队列中的数据：

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group testSpark --reset-offsets --all-topics --to-latest --execute
```

当一个topic存在多个分区，其中单个分区产生堆积时，生产环境中显然不适合丢弃整个topic的数据，所以，为了尽量减少对生产数据的影响，可以对指定的分区重设偏移量

查看消费者组

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

查看指定消费者组消费情况

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group testSpark
```

![image-20220722164149135](https://s2.loli.net/2022/07/22/Y9QdirAMeH6FW3g.png)

根据具体积压的分区情况，设置指定分区的偏移量

```bash
#--topic testSpark:0表示testSpark 0号分区
#--to-offset 123表示偏移量设置为123
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092  --group testSpark --reset-offsets --topic testSpark:0 --to-offset 123  --execute
```

-----------

kafka之所以使用了文件存储还可以有这么高的吞吐量得益于他利用了操作系统的零拷贝技术，主要是利用Channel的transferTo方法
普通IO模式下(read,write)，实际经过四次的文件内容复制

1. 将磁盘文件数据，copy到操作系统内核缓冲区；

2. 将内核缓冲区的数据，copy到应用程序的buffer；
3. 将应用程序buffer中的数据，copy到socket发送缓冲区(属于操作系统内核)；
4. 将socket buffer的数据，copy到网卡，由网卡进行网络传输。

而在零拷贝技术下，只有两步

1. 将磁盘文件数据读取到操作系统内核缓冲区
2. 将内核缓冲区数据copy到网卡，由网卡进行网络传输

### 通过broker id查看其对应的服务器

想法：通过broker id查看服务器的ip地址

未找到实现方法

----

Kafka集群中,每个broker都有一个唯一 的id值用来区分彼此。 Kafka在启动时会在zookeeper中/brokers/ids路径 下创建一个与当前broker的id为名称的虚节点，Kafka的健康状态检查就依赖于此节点。当broker下线时，该虚节点会自动删除，其他broker或者 客户端通过判断/brokers/ids路径下是否有此broker的id来确定该broker的健康状态。可以通过配置文件config/server.properties里的broker.id参数来配置broker的id值，默认情况下broker.id值为-1，Kafka broker的id值必须大于等于0时才有可能正常启动。

kafka在进行安装配置的时候，可以在server.properties文件中进行配置broker id

也可以对kafka访问设置进行配置

示例：

```
#常规配置
listeners=PLAINTEXT://localhost:9092
#内外网访问配置
listener.security.protocol.map=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
listeners=INTERNAL://192.168.0.1:9092,EXTERNAL://100.0.0.9:19092
advertised.listeners=INTERNAL://192.168.0.1:9092,EXTERNAL://100.0.0.9:19092
inter.broker.listener.name=INTERNAL
```

### kafka分区和副本的分配原则

参考链接：[分区副本的分配规则 | 石臻臻的杂货铺 (szzdzhp.com)](https://www.szzdzhp.com/kafka/Source_code/source-fenpei-rule.html)

-----

#### 副本类型

- 首领副本

  每个分区都有一个首领副本。为了保证一致性，所有生产者和消费者请求都会经过这个首领副本。

- 跟随者副本：首领以外的副本都是跟随者副本。跟随者副本不处理来自客户端的请求，它们唯一的任务就是从首领那里复制消息，保持与首领一致的状态。如果首领发生崩溃，其中一个跟随者会被提升为新首领，选举新首领是由控制器broker来完成的。

首领的另一个任务是搞清楚哪个跟随者的状态和自己是一致的。为了与首领保持一致，跟随者向首领发送获取数据的请求，这种请求和消费者为了读取消息而发送的请求是一样的。首领将响应消息发送给跟随者。请求消息里包含了跟随者想要获取消息的偏移量，而且这些偏移量总是有序的。例如，一个跟随者副本先请求消息1，再请求消息2，然后请求消息3，在收到这三个请求的响应前，它是不会发送第4个请求消息的。如果跟随者发送了第4个消息，首领就知道它已经收到了前三个响应。通过查看每个跟随者的请求的最新偏移量，首领就会知道每个跟随者的复制进度。

#### 自动分配

在创建topic或者是新增分区时，如果不指定分区副本的分配方式，Kafka会自动帮我们分配。

Kafka分区副本自动分配在三个地方用到，它们分别是：

- 创建topic时
- topic新增分区时
- 使用脚本自动重分配副本时

我们可以通过配置启动类参数生成分配策略

```
--zookeeper xxxx:2181 --topics-to-move-json-file config/move-json-file.json --broker-list "0,1,2,3" --generate  
move-json-file.json文件中内容为：
{
  "topics":[
    {"topic":"topicTest"}
  ],
 "version":1
}
以上参数表示为对topicTest重分配，希望分配到0，1，2，3这几个broker上
```

分区副本具体的分配算法有两种：无机架方式和有机架方式，
这两个算法的分配原则为：

- 尽量将副本平均分配在所有的broker上
- 每个broker上分配到的leader副本尽可能一样多
- 分区的多个副本被分配在不同的broker上
- 有机架分配方式要求集群中每个broker都要有机架信息否则抛出异常

##### 无机架方式

```scala
/**
   * 副本分配时,有三个原则:
   * 1. 将副本平均分布在所有的 Broker 上;
   * 2. partition 的多个副本应该分配在不同的 Broker 上;
   * 3. 如果所有的 Broker 有机架信息的话, partition 的副本应该分配到不同的机架上。
   *
   * 为实现上面的目标,在没有机架感知的情况下，应该按照下面两个原则分配 replica:
   * 1. 从 broker.list 随机选择一个 Broker,使用 round-robin 算法分配每个 partition 的第一个副本;
   * 2. 对于这个 partition 的其他副本,逐渐增加 Broker.id 来选择 replica 的分配。
   * 3. 对于副本分配来说,每经历一次Broker的遍历,则第一个副本跟后面的副本直接的间隔+1;
   */

  private def assignReplicasToBrokersRackUnaware(nPartitions: Int,
                                                 replicationFactor: Int,
                                                 brokerList: Seq[Int],
                                                 fixedStartIndex: Int,
                                                 startPartitionId: Int): Map[Int, Seq[Int]] = {
    val ret = mutable.Map[Int, Seq[Int]]()
    // 这里是上一层传递过了的所有 存活的Broker列表的ID
    val brokerArray = brokerList.toArray
    //默认随机选一个index开始
    val startIndex = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
    //默认从0这个分区号开始
    var currentPartitionId = math.max(0, startPartitionId)
    var nextReplicaShift = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
    for (_ <- 0 until nPartitions) {
      if (currentPartitionId > 0 && (currentPartitionId % brokerArray.length == 0))
        nextReplicaShift += 1
      val firstReplicaIndex = (currentPartitionId + startIndex) % brokerArray.length
      val replicaBuffer = mutable.ArrayBuffer(brokerArray(firstReplicaIndex))
      for (j <- 0 until replicationFactor - 1)
        replicaBuffer += brokerArray(replicaIndex(firstReplicaIndex, nextReplicaShift, j, brokerArray.length))
      ret.put(currentPartitionId, replicaBuffer)
      currentPartitionId += 1
    }
    ret
  }

  //主要的计算间隔数的方法
  private def replicaIndex(firstReplicaIndex: Int, secondReplicaShift: Int, replicaIndex: Int, nBrokers: Int): Int = {
    val shift = 1 + (secondReplicaShift + replicaIndex) % (nBrokers - 1)
    (firstReplicaIndex + shift) % nBrokers
  }
  
```

实例说明：

![在这里插入图片描述](https://s2.loli.net/2022/08/01/5RzcaxnBhCqFZv1.png)

上面是分配的情况,我们每一行每一行看, 每次都是先把每个分区的副本分配好的;

- 最开始的时候,随机一个Broker作为第一个来接受P0;；这里我们假设随机到了 broker-0； 所以第一个P0在broker-0上；那么第二个p0-2的位置跟nextReplicaShit有关，这个值也是随机的，这里假设随机的起始值也是0;；这个值意思可以简单的理解为，第一个副本和第二个副本的间隔

- 因为nextReplicaShit=0；所以p0的分配分别再 {0,1,2}
- 然后再分配后面的分区,分区的第一个副本位置都是按照broker顺序遍历的
- 直到这一次的broker遍历完了，那么就要重头再进行遍历了，同时nextReplicaShit=nextReplicaShit+1=1
- P5-1 再broker-0上，然后p5-2要跟p5-1间隔nextReplicaShit=1个位置，所以p5-2这时候在broker-2上，P5-3则在P5-2基础上顺推一位就行了，如果顺推的位置上已经有了副本,则继续顺推到没有当前分区副本的Broker

上面预设的nextReplicaShift=0，并且BrokerList顺序也是 {0,1,2,3}，这样的情况理解起来稍微容易一点，但是再实际的分配过程中，这个BrokerList并不是总是按照顺序来的，很可能都是乱的。




##### 有机架方式

机架（rack）相当于一个组，该组里面可能有多个broker，单个broker只属于一个组，通过修改server.properties来指定broker属于哪个特定的组（机架)

```
broker.rack=rackName
```

这相当于broker的分组能力，可以将不同组的brokers分配到不同的区域中，以提高单个区域发生故障时整个集群的可用性，即容灾

类似无机架分配方式，算法尽量将单个分区的每个副本分配到不同的组（机架）内，最多会分配到min(rackSize, replication-factor) 个不同的组中

```scala
private def assignReplicasToBrokersRackAware(nPartitions: Int,
                                             replicationFactor: Int,
                                             brokerMetadatas: Seq[BrokerMetadata],
                                             fixedStartIndex: Int,
                                             startPartitionId: Int): Map[Int, Seq[Int]] = {
  val brokerRackMap = brokerMetadatas.collect { case BrokerMetadata(id, Some(rack)) =>
    id -> rack
  }.toMap
  //统计机架个数
  val numRacks = brokerRackMap.values.toSet.size
  //基于机架信息生成一个Broker列表，不同机架上的Broker交替出现
  val arrangedBrokerList = getRackAlternatedBrokerList(brokerRackMap)
  val numBrokers = arrangedBrokerList.size
  val ret = mutable.Map[Int, Seq[Int]]()
  val startIndex = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(arrangedBrokerList.size)
  var currentPartitionId = math.max(0, startPartitionId)
  var nextReplicaShift = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(arrangedBrokerList.size)
  for (_ <- 0 until nPartitions) {
    if (currentPartitionId > 0 && (currentPartitionId % arrangedBrokerList.size == 0))
      nextReplicaShift += 1
    val firstReplicaIndex = (currentPartitionId + startIndex) % arrangedBrokerList.size
    val leader = arrangedBrokerList(firstReplicaIndex)
    //每个分区的副本分配列表
    val replicaBuffer = mutable.ArrayBuffer(leader)
    //每个分区中所分配的机架的列表集
    val racksWithReplicas = mutable.Set(brokerRackMap(leader))
    //每个分区所分配的brokerId的列表集，和racksWithReplicas一起用来做一层筛选处理
    val brokersWithReplicas = mutable.Set(leader)
    var k = 0
    for (_ <- 0 until replicationFactor - 1) {
      var done = false
      while (!done) {
        val broker = arrangedBrokerList(replicaIndex(firstReplicaIndex, nextReplicaShift * numRacks, k, arrangedBrokerList.size))
        val rack = brokerRackMap(broker)
        // Skip this broker if
        // 1. there is already a broker in the same rack that has assigned a replica AND there is one or more racks
        //    that do not have any replica, or
        // 2. the broker has already assigned a replica AND there is one or more brokers that do not have replica assigned
        if ((!racksWithReplicas.contains(rack) || racksWithReplicas.size == numRacks)
            && (!brokersWithReplicas.contains(broker) || brokersWithReplicas.size == numBrokers)) {
          replicaBuffer += broker
          racksWithReplicas += rack
          brokersWithReplicas += broker
          done = true
        }
        k += 1
      }
    }
    ret.put(currentPartitionId, replicaBuffer)
    currentPartitionId += 1
  }
  ret
}
```

#### 指定分配规则分配

参考链接：[(22条消息) Kafka分区副本重分配源码分析_罗纳尔光的博客-CSDN博客](https://blog.csdn.net/qq_37913997/article/details/122944747)

---



### __consumer_offsets

_consumer_offsets 在 Kafka 源码中有个更为正式的名字，叫位移主题。将 Consumer 的位移数据作为一条条普通的 Kafka 消息，提交到 _consumer_offsets 中。可以这么说，__consumer_offsets 的主要作用是保存 Kafka 消费者的位移信息。

位移主题的 Key 中应该保存 3 部分内容：<Group ID，主题名，分区号 >

kafka如何清除位移主题中的过期消息？

Kafka 使用 Compact 策略来删除位移主题中的过期消息，避免该主题无限期膨胀。那么应该如何定义 Compact 策略中的过期呢？对于同一个 Key 的两条消息 M1 和 M2，如果 M1 的发送时间早于 M2，那么 M1 就是过期消息。Compact 的过程就是扫描日志的所有消息，剔除那些过期的消息，然后把剩下的消息整理在一起。

![img](https://s2.loli.net/2022/08/01/szWfPHOdxQcFthm.png)

图中位移为 0、2 和 3 的消息的 Key 都是 K1。Compact 之后，分区只需要保存位移为 3 的消息，因为它是最新发送的。Kafka 提供了专门的后台线程定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据。这个后台线程叫 Log Cleaner。很多实际生产环境中都出现过位移主题无限膨胀占用过多磁盘空间的问题，如果你的环境中也有这个问题，去检查一下 Log Cleaner 线程的状态，通常都是这个线程挂掉了导致的。

## StreamingContext第二个参数

相关参考：

https://blog.csdn.net/qq_22473611/article/details/88060377

https://blog.csdn.net/super_wj0820/article/details/101775937

https://developer.aliyun.com/article/73004

https://blog.csdn.net/super_wj0820/article/details/100987037

https://blog.csdn.net/super_wj0820/article/details/100533335

https://qa.1r1g.com/sf/ask/2946429041/

-----

### Spark相关概念

Application：表示你的应用程序

Driver：表示 main() 函数，创建 SparkContext。由 SparkContext 负责与 ClusterManager 通信，进行资源的申请，任务的分配和监控等。程序执行完毕后关闭SparkContext

Executor：某个 Application 运行在 Worker 节点上的一个进程，该进程负责运行某些 Task ，并且负责将数据存在内存或者磁盘上。在 Spark on Yarn 模式下，其进程名称为 CoarseGrainedExecutor Backend，一个 CoarseGrainedExecutor Backend 进程有且仅有一个 executor 对象，它负责将 Task 包装成 TaskRunner，并从线程池中抽取出一个空闲线程运行 Task，这样，每个CoarseGrainedExecutorBackend 能并行运行Task的数据就取决于分配给它的CPU的个数。

Worker：集群中可以运行 Application 代码的节点。在 Standalone 模式中指的是通过 slave 文件配置的 worker 节点，在Spark on Yarn 模式中指的就是 NodeManager 节点。

Task：在 Executor 进程中执行任务的工作单元，多个 Task 组成一个 Stage

Job：包含多个 Task 组成的并行计算，是由 Action 行为触发的

Stage：⼀个Job会被拆分很多组任务，每组任务被称为Stage。Stage概念是spark中独有的，各个stage之间按照顺序执行。至于stage是怎么切分的，首选得知道spark论文中提到的narrow dependency（窄依赖）和wide dependency（ 宽依赖）的概念。其实很好区分，看一下父RDD中的数据是否进入不同的子RDD，如果只进入到一个子RDD则是窄依赖，否则就是宽依赖。宽依赖和窄依赖的边界就是stage的划分点，从后往前，遇到宽依赖就切割stage。

DAGScheduler：根据 Job 构建基于 Stage 的 DAG，并提交 Stage 给 TaskScheduler，其划分 Stage 的依据是 RDD 之间的依赖关系

TaskScheduler：将 TaskSet 提交给 Worker（集群）运行，每个 Executor 运行什么 Task 就是在此处分配的。

-------

### Spark Streaming 基础概念

#### DStream

Discretized Stream 是 SS 的基础抽象，代表持续性的数据流和经过各种 Spark 原语操作后的结果数据流。DStream 本质上是一个以时间为键，RDD 为值的哈希表，保存了按时间顺序产生的 RDD，而每个 RDD 封装了批处理时间间隔内获取到的数据。Spark Streaming每次将新产生的 RDD 添加到哈希表中，而对于已经不再需要的 RDD 则会从这个哈希表中删除，所以 DStream 也可以简单地理解为以时间为键的 RDD 的动态序列。
![img](https://s2.loli.net/2022/07/28/eBvQCschHb47lLz.png)

#### 窗口时间间隔（窗口长度）：windowDuration

 窗口时间间隔又称为窗口长度，它是一个抽象的时间概念，决定了Spark Streaming 对 RDD 序列进行处理的范围与粒度，即用户可以通过设置窗口长度来对一定时间范围内的数据进行统计和分析。假如设置批处理时间间隔为 1s，窗口时间间隔为 3s。如下图，DStream 每 1s 会产生一个 RDD，红色边框的矩形框就表示窗口时间间隔，一个窗口时间间隔内最多有 3 个 RDD，Spark Streaming 在一个窗口时间间隔内最多会对 3 个 RDD 中的数据进行统计和分析。
![img](https://s2.loli.net/2022/07/28/bmITCfw3e6GkJNZ.jpg)

#### 滑动时间间隔：slideDuration

滑动时间间隔决定了 SS 程序对数据进行统计和分析的频率。它指的是经过多长时间窗口滑动一次形成新的窗口，滑动时间间隔默认情况下和批处理时间间隔相同，而窗口时间间隔一般设置的要比它们两个大。在这里必须注意的一点是滑动时间间隔和窗口时间间隔的大小一定得设置为批处理时间间隔的整数倍。

如下图，批处理时间间隔是 1 个时间单位，窗口时间间隔是 3 个时间单位，滑动时间间隔是 2 个时间单位。对于初始的窗口 time 1-time 3，只有窗口时间间隔满足了才触发数据的处理。这里需要注意的一点是，初始的窗口有可能覆盖的数据没有 3 个时间单位，但是随着时间的推进，窗口最终会覆盖到 3 个时间单位的数据。当每个 2 个时间单位，窗口滑动一次后，会有新的数据流入窗口，这时窗口会移去最早的两个时间单位的数据，而与最新的两个时间单位的数据进行汇总形成新的窗口（time3-time5）。
![img](https://s2.loli.net/2022/07/28/PVK5JAp6fCMjyuL.jpg)

---

<img src="https://s2.loli.net/2022/07/28/Bu8fT9Ulo2V7JX3.png" alt="img" style="zoom:67%;" />

### 明确Spark中Job 与 Streaming中 Job 的区别

Spark Core：

一个 RDD DAG Graph 可以生成一个或多个 Job（Action操作）

一个Job可以认为就是会最终输出一个结果RDD的一条由RDD组织而成的计算

Job在spark里应用里是一个被调度的单位

Streaming：

一个 batch 的数据对应一个 DStreamGraph

而一个 DStreamGraph 包含一或多个关于 DStream 的输出操作

每一个输出对应于一个Job，一个 DStreamGraph 对应一个JobSet，里面包含一个或多个Job

----

###  Streaming Job的并行度

Streaming Job的并行度复杂些，由两个配置决定：

1. spark.scheduler.mode(FIFO/FAIR)
2. spark.streaming.concurrentJobs

我们知道一个Batch可能会有多个Action执行，比如你注册了多个Kafka数据流，每个Action都会产生一个Job,所以一个Batch有可能是一批Job,也就是JobSet的概念，这些Job由jobExecutor依次提交执行,而JobExecutor是一个默认池子大小为1的线程池，所以只能执行完一个Job再执行另外一个Job。这里说的池子，他的大小就是由**spark.streaming.concurrentJobs** 控制的。

concurrentJobs 其实决定了向Spark Core提交Job的并行度。提交一个Job，必须等这个执行完了，才会提交第二个。假设我们把它设置为2，则会并发的把Job提交给Spark Core，Spark 有自己的机制决定如何运行这两个Job,这个机制其实就是FIFO或者FAIR（决定了资源的分配规则）。默认是FIFO,也就是先进先出，你把concurrentJobs设置为2，但是如果底层是FIFO,那么会优先执行先提交的Job，虽然如此，如果资源够两个job运行，还是会并行运行两个Job。

----

***问题***：如果我们的处理时间超过了批处理间隔，会发生什么？

如果您的工作人员尚未完成处理上一个作业,则他们将不会开始处理下一个作业，除非您明确设置`spark.streaming.concurrentJobs`大于1，这意味着将读取偏移量，但实际上不会发送给负责读取数据的执行程序,因此不会丢失任何数据.

但这意味着你的工作将无限延迟并导致大量处理延迟，根据经验，任何Spark作业处理时间应小于为该作业设置的间隔。

----







## Scala集合

### 基本介绍

 Scala 同时支持不可变集合和可变集合，不可变集合可以安全的并发访问 

两个主要的包： 

不可变集合：scala.collection.immutable 

可变集合： scala.collection.mutable

Scala 默认采用不可变集合，对于几乎所有的集合类，Scala 都同时提供了可变(mutable)和不可变 (immutable)的版本

Scala 的集合有三大类：序列 Seq(有序的,Linear Seq)、集 Set、映射 Map【key->value】，所有的 集合都扩展自 Iterable 特质，在 Scala 中集合有可变（mutable）和不可变（immutable）两种类型。

- 不可变集合：scala 不可变集合，就是这个集合本身不能动态变化。类似 java 的数组，是不可以 动态增长的) 
- 可变集合：可变集合，就是这个集合本身可以动态变化的。比如:ArrayList , 是可以动态增长的

### 不可变集合

<img src="https://img-blog.csdnimg.cn/284c83ed1bc3475292815cb8385a0873.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ3VuX0s=,size_20,color_FFFFFF,t_70,g_se,x_16" alt="img"  />



### 可变集合

![img](https://img-blog.csdnimg.cn/827f92fc773149969cd20ebb8fd47147.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ3VuX0s=,size_20,color_FFFFFF,t_70,g_se,x_16)







# 基础功能

实现spark从kafka读取数据，然后上传到ftp服务器

```scala
object kafka {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("SparkStreaming").setMaster("local")
      //问题：第一批次轮询还未结束，3s间隔后，下一轮查询是否会开始
    val streamingContext = new StreamingContext(sparkConf, Seconds(3))

    val kafkaParams: Map[String, Object] = Map[String, Object](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "localhost:9092",
      ConsumerConfig.GROUP_ID_CONFIG -> "testSpark",

      "key.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer",
      "value.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer"
    )

    val kafkaData: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
      streamingContext,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Subscribe[String, String](Set("testSpark"), kafkaParams)
    )

    val result: DStream[String] = kafkaData.map(_.value())

    result.foreachRDD(
      rdd => rdd.foreachPartition(
        iter => {
            //foreachPartition可避免多次建立链接
          val ftpClient = new Ftp("192.168.0.105", 21, "ZWH", "nishengri")
          ftpClient.setMode(FtpMode.Passive)
          val strings: util.ArrayList[String] = new util.ArrayList[String]()
          while (iter.hasNext){
            val str: String = iter.next()
            strings.add(str)
            if(strings.size()>1){
              FileUtils.writeLines(new File("Spark/src/main/resources/kafkaData.txt"), "utf-8", strings, true)
            }
          }
          if(strings.size()>0){
            FileUtils.writeLines(new File("Spark/src/main/resources/kafkaData.txt"), "utf-8", strings, true)
          }
          ftpClient.cd(".")
          ftpClient.upload(".", new File("Spark/src/main/resources/kafkaData.txt"))
          ftpClient.close()
        }
      )
    )

    streamingContext.start()
    streamingContext.awaitTermination()

  }
}
```

问题：

- 在写文件时使用PrintWriter和Writer无法将内容写入文件，且无报错产生，奇奇怪怪
- 不能自动创建文件

```scala
cn.hutool.core.io.IORuntimeException: FileNotFoundException: /home/zhuweihao/opt/datakafkaData-0-2022-07-15 00:43:59.txt (No such file or directory)

val file: File = new File(pathname + "kafkaData-" + partitionId + "-" + time + ".txt")
if (!file.exists()) {
  file.createNewFile()
}
```



# 后续要求

    * 1. 不同分区 写不同的文件，文件命名问题 ？
    * 2. 防止某个文件过大 控制写文件的大小 ？
    * 3. 文件清理工作，上传成功，进行清理
    * 4. kafka 偏移量维护

#### 文件命名问题

文件名后加上分区号以及时间

```scala
val partitionId: Int = TaskContext.getPartitionId()
val date: Any = new Date()
val time: String = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(date)
val ftpClient = new Ftp(host, port.toInt, user, passerword)
```



#### 通过配置文件进行配置

```scala
//读取配置文件
val properties: Properties = new Properties()
properties.load(this.getClass.getClassLoader.getResourceAsStream("properties.properties"))
properties.getProperty("BOOTSTRAP_SERVERS")
val BOOTSTRAP_SERVERS = properties.getProperty("BOOTSTRAP_SERVERS")
val GROUP_ID = properties.getProperty("GROUP_ID")
val key_deserializer = properties.getProperty("key.deserializer")
val value_deserializer = properties.getProperty("value.deserializer")
val topic = properties.getProperty("topic")
val host = properties.getProperty("host")
val port = properties.getProperty("port")
val user = properties.getProperty("user")
val passerword = properties.getProperty("passerword")
val pathname = properties.getProperty("pathname")
```



#### 打包

参考：[(22条消息) maven scala java 混合项目编译、打包（jar包）、运行_枪枪枪的博客-CSDN博客_scala打包jar包](https://blog.csdn.net/az9996/article/details/123109835)



```xml
<!-- Scala Compiler -->
<plugin>
    <groupId>org.scala-tools</groupId>
    <artifactId>maven-scala-plugin</artifactId>
    <version>2.15.2</version>
    <executions>
        <execution>
            <id>scala-compile</id>
            <goals>
                <goal>compile</goal>
            </goals>
            <configuration>
                <!--includes是一个数组，包含要编译的code-->
                <includes>
                    <include>**/*.scala</include>
                </includes>
            </configuration>
        </execution>
        <execution>
            <id>scala-test-compile</id>
            <goals>
                <goal>testCompile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

上传打包运行

```
//windows上传文件到linux
scp E:\BigData\Spark\target\Spark-1.0-SNAPSHOT.jar zhuweihao@192.168.0.107:/home/zhuweihao/opt/data
//spark提交运行
spark-submit --master local[2] /home/zhuweihao/opt/data/Spark-1.0-SNAPSHOT.jar
```



错误：



```
cn.hutool.extra.ftp.FtpException: ConnectException: Connection refused (Connection refused)
```

经排查：配置文件ip写错了doge



文件upload失败

原因：ftp服务器搭建在windows，程序在linux系统执行，windows与linux之间进行ftp传输时对文件名有所限制

亲测文件名中不能使用冒号





# 最终代码

```scala
package com.zhuweihao.SparkStreaming

import cn.hutool.extra.ftp.{Ftp, FtpMode}
import org.apache.commons.io.FileUtils
import org.apache.kafka.clients.consumer.{ConsumerConfig, ConsumerRecord}
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.{ConsumerStrategies, KafkaUtils, LocationStrategies}
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext, TaskContext}
import org.json4s.DateFormat

import java.io.{File, FileInputStream, FileWriter, InputStream, PrintWriter}
import java.text.SimpleDateFormat
import java.util
import java.util.{Date, Properties}

/**
 * @Author zhuweihao
 * @Date 2022/7/8 23:44
 * @Description com.zhuweihao.scala.SparkStreaming
 */
object kafka {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("SparkStreaming").setMaster("local")
    val streamingContext = new StreamingContext(sparkConf, Seconds(15))

    //读取配置文件
    val properties: Properties = new Properties()
    properties.load(this.getClass.getClassLoader.getResourceAsStream("properties.properties"))
    properties.getProperty("BOOTSTRAP_SERVERS")
    val BOOTSTRAP_SERVERS = properties.getProperty("BOOTSTRAP_SERVERS")
    val GROUP_ID = properties.getProperty("GROUP_ID")
    val key_deserializer = properties.getProperty("key.deserializer")
    val value_deserializer = properties.getProperty("value.deserializer")
    val topic = properties.getProperty("topic")
    val host = properties.getProperty("host")
    val port = properties.getProperty("port")
    val user = properties.getProperty("user")
    val passerword = properties.getProperty("passerword")
    val pathname = properties.getProperty("pathname")
    //kafka相关配置
    val kafkaParams: Map[String, Object] = Map[String, Object](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> BOOTSTRAP_SERVERS,
      ConsumerConfig.GROUP_ID_CONFIG -> GROUP_ID,

      "key.deserializer" -> key_deserializer,
      "value.deserializer" -> value_deserializer
    )
    val kafkaData: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
      streamingContext,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Subscribe[String, String](Set(topic), kafkaParams)
    )
    val result: DStream[String] = kafkaData.map(_.value())
    result.foreachRDD(
      rdd => rdd.foreachPartition(
        iter => {
          val partitionId: Int = TaskContext.getPartitionId()
          val date: Any = new Date()
          val time: String = new SimpleDateFormat("yyyy-MM-dd HH:mm").format(date)
          val ftpClient = new Ftp(host, port.toInt, user, passerword)
          val file: File = new File(pathname + "kafkaData-" + partitionId + "-" + time + ".txt")
          if (!file.exists()) {
            file.createNewFile()
          }
          ftpClient.setMode(FtpMode.Passive)
          val strings: util.ArrayList[String] = new util.ArrayList[String]()
          while (iter.hasNext) {
            val str: String = iter.next()
            strings.add(str)
            if (strings.size() > 10) {
              FileUtils.writeLines(file, "utf-8", strings, true)
              strings.clear()
            }
          }
          if (strings.size() > 0) {
            FileUtils.writeLines(file, "utf-8", strings, true)
          }
          ftpClient.cd(".")
          val bool: Boolean = ftpClient.upload(".", file)
          println("upload:" + bool)
          ftpClient.close()
        }
      )
    )


    streamingContext.start()
    streamingContext.awaitTermination()

  }
}
```