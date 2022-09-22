# FLINK官网：https://flink.apache.org/

# Flink简介（官网搬运）

Flink项目的核心目标，是“数据流上的有状态计算”（Stateful  Computations over Data Streams）。

Apache Flink 是一个框架和分布式处理引擎，如下图所示，用于对无界和有界数据流进行有状态计算。Flink 被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

Flink 是一个流式大数据处理引擎。 “内存执行速度”和“任意规模”，突出了 Flink 的两个特点：速度快、可扩展性强

![image-20220920152901972](Flink.assets/image-20220920152901972.png)

## 流式数据处理的发展

### 传统数据处理架构

从字面上来看OLTP是做事务处理，OLAP是做分析处理。从对数据库操作来看，OLTP主要是对数据的增删改，OLAP是对数据的查询。

#### 事务处理（OLTP,on-line transaction processing）

![image-20220920155004246](Flink.assets/image-20220920155004246.png)



#### 分析处理（OLAP,On-Line Analytical Processing）



### 有状态的流处理

不难想到，如果我们对于事件流的处理非常简单，例如收到一条请求就返回一个“收到”， 那就可以省去数据库的查询和更新了。但是这样的处理是没什么实际意义的。在现实的应用中， 往往需要还其他一些额外数据。我们可以把需要的额外数据保存成一个“状态”，然后针对这条数据进行处理，并且更新状态。在传统架构中，这个状态就是保存在数据库里的。这就是所谓的“有状态的流处理”。 

为了加快访问速度，我们可以直接将状态保存在本地内存。当应用收到一 个新事件时，它可以从状态中读取数据，也可以更新状态。而当状态是从内存中读写的时候， 这就和访问本地变量没什么区别了，实时性可以得到极大的提升。 

![image-20220920160630651](Flink.assets/image-20220920160630651.png)

另外，数据规模增大时，我们也不需要做重构，只需要构建分布式集群，各自在本地计算就可以了，可扩展性也变得更好。 

因为采用的是一个分布式系统，所以还需要保护本地状态，防止在故障时数据丢失。我们可以定期地将应用状态的一致性检查点（checkpoint）存盘，写入远程的持久化存储，遇到故障时再去读取进行恢复，这样就保证了更好的容错性。

### Lambda架构

对于有状态的流处理，当数据越来越多时，我们必须用分布式的集群架构来获取更大的吞吐量。但是分布式架构会带来另一个问题：怎样保证数据处理的顺序是正确的呢？ 

对于批处理来说，这并不是一个问题。因为所有数据都已收集完毕，我们可以根据需要选择、排列数据，得到想要的结果。可如果我们采用“来一个处理一个”的流处理，就可能出现 “乱序”的现象：本来先发生的事件，因为分布处理的原因滞后了。怎么解决这个问题呢？ 

以 Storm 为代表的第一代分布式开源流处理器，主要专注于具有毫秒延迟的事件处理，特点就是一个字“快”；而对于准确性和结果的一致性，是不提供内置支持的，因为结果有可能取决于到达事件的时间和顺序。另外，第一代流处理器通过检查点来保证容错性，但是故障恢 复的时候，即使事件不会丢失，也有可能被重复处理——所以无法保证 exactly-once。

 与批处理器相比，可以说第一代流处理器牺牲了结果的准确性，用来换取更低的延迟。而 批处理器恰好反过来，牺牲了实时性，换取了结果的准确。 我们自然想到，如果可以让二者做个结合，不就可以同时提供快速和准确的结果了吗？

正是基于这样的思想，Lambda 架构被设计出来。我们可以认为这是第二代流处理架构，但事实上，它只是第一代流处理器和批处理器的简单合并。

![image-20220920160923195](Flink.assets/image-20220920160923195.png)

Lambda 架构主体是传统批处理架构的增强。它的“批处理层”（Batch Layer）就是由传统的批处理器和存储组成，而“实时层”（Speed Layer）则由低延迟的流处理器实现。数据到达之后，两层处理双管齐下，一方面由流处理器进行实时处理，另一方面写入批处理存储空间， 等待批处理器批量计算。流处理器快速计算出一个近似结果，并将它们写入“流处理表”中。 而批处理器会定期处理存储中的数据，将准确的结果写入批处理表，并从快速表中删除不准确的结果。最终，应用程序会合并快速表和批处理表中的结果，并展示出来。

### 新一代流处理器

之前的分布式流处理架构，都有明显的缺陷，人们也一直没有放弃对流处理器的改进和完善。终于，在原有流处理器的基础上，新一代分布式开源流处理器诞生了。为了与之前的系统区分，我们一般称之为第三代流处理器，代表当然就是 Flink。 

第三代流处理器通过巧妙的设计，完美解决了乱序数据对结果正确性的影响。这一代系统还做到了精确一次（exactly-once）的一致性保障，是第一个具有一致性和准确结果的开源流 处理器。另外，先前的流处理器仅能在高吞吐和低延迟中二选一，而新一代系统能够同时提供这两个特性。所以可以说，这一代流处理器仅凭一套系统就完成了 Lambda 架构两套系统的工作，它的出现使得 Lambda 架构黯然失色。 

除了低延迟、容错和结果准确性之外，新一代流处理器还在不断添加新的功能，例如高可用的设置，以及与资源管理器（如 YARN 或 Kubernetes）的紧密集成等等。

## 架构

Apache Flink 是一个框架和分布式处理引擎，用于在*无边界和有边界*数据流上进行有状态的计算。Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。

接下来，我们来介绍一下 Flink 架构中的重要方面。

### 处理无界和有界数据

任何类型的数据都可以形成一种事件流。信用卡交易、传感器测量、机器日志、网站或移动应用程序上的用户交互记录，所有这些数据都形成一种流。

数据可以被作为 *无界* 或者 *有界* 流来处理。

1. **无界流** 有定义流的开始，但没有定义流的结束。它们会无休止地产生数据。无界流的数据必须持续处理，即数据被摄取后需要立刻处理。我们不能等到所有数据都到达再处理，因为输入是无限的，在任何时候输入都不会完成。处理无界数据通常要求以特定顺序摄取事件，例如事件发生的顺序，以便能够推断结果的完整性。
2. **有界流** 有定义流的开始，也有定义流的结束。有界流可以在摄取所有数据后再进行计算。有界流所有数据可以被排序，所以并不需要有序摄取。有界流处理通常被称为批处理

![img](Flink.assets/bounded-unbounded.png)

**Apache Flink 擅长处理无界和有界数据集** 精确的时间控制和状态化使得 Flink 的运行时(runtime)能够运行任何处理无界流的应用。有界流则由一些专为固定大小数据集特殊设计的算法和数据结构进行内部处理，产生了出色的性能。

通过探索 Flink 之上构建的 [用例](https://flink.apache.org/zh/usecases.html) 来加深理解。

### 部署应用到任意地方

Apache Flink 是一个分布式系统，它需要计算资源来执行应用程序。Flink 集成了所有常见的集群资源管理器，例如 [Hadoop YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)、 [Apache Mesos](https://mesos.apache.org/) 和 [Kubernetes](https://kubernetes.io/)，但同时也可以作为独立集群运行。

Flink 被设计为能够很好地工作在上述每个资源管理器中，这是通过资源管理器特定(resource-manager-specific)的部署模式实现的。Flink 可以采用与当前资源管理器相适应的方式进行交互。

部署 Flink 应用程序时，Flink 会根据应用程序配置的并行性自动标识所需的资源，并从资源管理器请求这些资源。在发生故障的情况下，Flink 通过请求新资源来替换发生故障的容器。提交或控制应用程序的所有通信都是通过 REST 调用进行的，这可以简化 Flink 与各种环境中的集成。

### 运行任意规模应用

Flink 旨在任意规模上运行有状态流式应用。因此，应用程序被并行化为可能数千个任务，这些任务分布在集群中并发执行。所以应用程序能够充分利用无尽的 CPU、内存、磁盘和网络 IO。而且 Flink 很容易维护非常大的应用程序状态。其异步和增量的检查点算法对处理延迟产生最小的影响，同时保证精确一次状态的一致性。

[Flink 用户报告了其生产环境中一些令人印象深刻的扩展性数字](https://flink.apache.org/zh/poweredby.html)

- 处理**每天处理数万亿的事件**,
- 应用维护**几TB大小的状态**, 和
- 应用**在数千个内核上运行**。

### 利用内存性能

有状态的 Flink 程序针对本地状态访问进行了优化。任务的状态始终保留在内存中，如果状态大小超过可用内存，则会保存在能高效访问的磁盘数据结构中。任务通过访问本地（通常在内存中）状态来进行所有的计算，从而产生非常低的处理延迟。Flink 通过定期和异步地对本地状态进行持久化存储来保证故障场景下精确一次的状态一致性。

![img](Flink.assets/local-state.png)

## 应用

Apache Flink 是一个针对无界和有界数据流进行有状态计算的框架。Flink 自底向上在不同的抽象级别提供了多种 API，并且针对常见的使用场景开发了专用的扩展库。

在本章中，我们将介绍 Flink 所提供的这些简单易用、易于表达的 API 和库。

### 流处理应用的基本组件

可以由流处理框架构建和执行的应用程序类型是由框架对 *流*、*状态*、*时间* 的支持程度来决定的。在下文中，我们将对上述这些流处理应用的基本组件逐一进行描述，并对 Flink 处理它们的方法进行细致剖析。

#### 流

显而易见，（数据）流是流处理的基本要素。然而，流也拥有着多种特征。这些特征决定了流如何以及何时被处理。Flink 是一个能够处理任何类型数据流的强大处理框架。

- **有界** 和 **无界** 的数据流：流可以是无界的；也可以是有界的，例如固定大小的数据集。Flink 在无界的数据流处理上拥有诸多功能强大的特性，同时也针对有界的数据流开发了专用的高效算子。
- **实时** 和 **历史记录** 的数据流：所有的数据都是以流的方式产生，但用户通常会使用两种截然不同的方法处理数据。或是在数据生成时进行实时的处理；亦或是先将数据流持久化到存储系统中——例如文件系统或对象存储，然后再进行批处理。Flink 的应用能够同时支持处理实时以及历史记录数据流。

#### 状态

只有在每一个单独的事件上进行转换操作的应用才不需要状态，换言之，每一个具有一定复杂度的流处理应用都是有状态的。任何运行基本业务逻辑的流处理应用都需要在一定时间内存储所接收的事件或中间结果，以供后续的某个时间点（例如收到下一个事件或者经过一段特定时间）进行访问并进行后续处理。

<img src="Flink.assets/function-state.png" alt="img" style="zoom: 50%;" />

应用状态是 Flink 中的一等公民，Flink 提供了许多状态管理相关的特性支持，其中包括：

- **多种状态基础类型**：Flink 为多种不同的数据结构提供了相对应的状态基础类型，例如原子值（value），列表（list）以及映射（map）。开发者可以基于处理函数对状态的访问方式，选择最高效、最适合的状态基础类型。
- **插件化的State Backend**：State Backend 负责管理应用程序状态，并在需要的时候进行 checkpoint。Flink 支持多种 state backend，可以将状态存在内存或者 [RocksDB](https://rocksdb.org/)。RocksDB 是一种高效的嵌入式、持久化键值存储引擎。Flink 也支持插件式的自定义 state backend 进行状态存储。
- **精确一次语义**：Flink 的 checkpoint 和故障恢复算法保证了故障发生后应用状态的一致性。因此，Flink 能够在应用程序发生故障时，对应用程序透明，不造成正确性的影响。
- **超大数据量状态**：Flink 能够利用其异步以及增量式的 checkpoint 算法，存储数 TB 级别的应用状态。
- **可弹性伸缩的应用**：Flink 能够通过在更多或更少的工作节点上对状态进行重新分布，支持有状态应用的分布式的横向伸缩。

#### 时间

时间是流处理应用另一个重要的组成部分。因为事件总是在特定时间点发生，所以大多数的事件流都拥有事件本身所固有的时间语义。进一步而言，许多常见的流计算都基于时间语义，例如窗口聚合、会话计算、模式检测和基于时间的 join。流处理的一个重要方面是应用程序如何衡量时间，即区分事件时间（event-time）和处理时间（processing-time）。

Flink 提供了丰富的时间语义支持。

- **事件时间模式**：使用事件时间语义的流处理应用根据事件本身自带的时间戳进行结果的计算。因此，无论处理的是历史记录的事件还是实时的事件，事件时间模式的处理总能保证结果的准确性和一致性。
- **Watermark 支持**：Flink 引入了 watermark 的概念，用以衡量事件时间进展。Watermark 也是一种平衡处理延时和完整性的灵活机制。
- **迟到数据处理**：当以带有 watermark 的事件时间模式处理数据流时，在计算完成之后仍会有相关数据到达。这样的事件被称为迟到事件。Flink 提供了多种处理迟到数据的选项，例如将这些数据重定向到旁路输出（side output）或者更新之前完成计算的结果。
- **处理时间模式**：除了事件时间模式，Flink 还支持处理时间语义。处理时间模式根据处理引擎的机器时钟触发计算，一般适用于有着严格的低延迟需求，并且能够容忍近似结果的流处理应用。

### 分层 API

Flink 根据抽象程度分层，提供了三种不同的 API。每一种 API 在简洁性和表达力上有着不同的侧重，并且针对不同的应用场景。

<img src="Flink.assets/api-stack.png" alt="img" style="zoom: 33%;" />

#### ProcessFunction

[ProcessFunction](https://nightlies.apache.org/flink/flink-docs-stable/dev/stream/operators/process_function.html) 是 Flink 所提供的最具表达力的接口。ProcessFunction 可以处理一或两条输入数据流中的单个事件或者归入一个特定窗口内的多个事件。它提供了对于时间和状态的细粒度控制。开发者可以在其中任意地修改状态，也能够注册定时器用以在未来的某一时刻触发回调函数。因此，你可以利用 ProcessFunction 实现许多[有状态的事件驱动应用](https://flink.apache.org/zh/usecases.html#eventDrivenApps)所需要的基于单个事件的复杂业务逻辑。

#### DataStream API

[DataStream API](https://nightlies.apache.org/flink/flink-docs-stable/dev/datastream_api.html) 为许多通用的流处理操作提供了处理原语。这些操作包括窗口、逐条记录的转换操作，在处理事件时进行外部数据库查询等。DataStream API 支持 Java 和 Scala 语言，预先定义了例如`map()`、`reduce()`、`aggregate()` 等函数。你可以通过扩展实现预定义接口或使用 Java、Scala 的 lambda 表达式实现自定义的函数。

#### SQL & Table API

Flink 支持两种关系型的 API，[Table API 和 SQL](https://nightlies.apache.org/flink/flink-docs-stable/dev/table/index.html)。这两个 API 都是批处理和流处理统一的 API，这意味着在无边界的实时数据流和有边界的历史记录数据流上，关系型 API 会以相同的语义执行查询，并产生相同的结果。Table API 和 SQL 借助了 [Apache Calcite](https://calcite.apache.org/) 来进行查询的解析，校验以及优化。它们可以与 DataStream 和 DataSet API 无缝集成，并支持用户自定义的标量函数，聚合函数以及表值函数。

Flink 的关系型 API 旨在简化[数据分析](https://flink.apache.org/zh/usecases.html#analytics)、[数据流水线和 ETL 应用](https://flink.apache.org/zh/usecases.html#pipelines)的定义。

### 库

Flink 具有数个适用于常见数据处理应用场景的扩展库。这些库通常嵌入在 API 中，且并不完全独立于其它 API。它们也因此可以受益于 API 的所有特性，并与其他库集成。

- **[复杂事件处理(CEP)](https://nightlies.apache.org/flink/flink-docs-stable/dev/libs/cep.html)**：模式检测是事件流处理中的一个非常常见的用例。Flink 的 CEP 库提供了 API，使用户能够以例如正则表达式或状态机的方式指定事件模式。CEP 库与 Flink 的 DataStream API 集成，以便在 DataStream 上评估模式。CEP 库的应用包括网络入侵检测，业务流程监控和欺诈检测。
- **[DataSet API](https://nightlies.apache.org/flink/flink-docs-stable/dev/batch/index.html)**：DataSet API 是 Flink 用于批处理应用程序的核心 API。DataSet API 所提供的基础算子包括*map*、*reduce*、*(outer) join*、*co-group*、*iterate*等。所有算子都有相应的算法和数据结构支持，对内存中的序列化数据进行操作。如果数据大小超过预留内存，则过量数据将存储到磁盘。Flink 的 DataSet API 的数据处理算法借鉴了传统数据库算法的实现，例如混合散列连接（hybrid hash-join）和外部归并排序（external merge-sort）。
- **[Gelly](https://nightlies.apache.org/flink/flink-docs-stable/dev/libs/gelly/index.html)**: Gelly 是一个可扩展的图形处理和分析库。Gelly 是在 DataSet API 之上实现的，并与 DataSet API 集成。因此，它能够受益于其可扩展且健壮的操作符。Gelly 提供了[内置算法](https://nightlies.apache.org/flink/flink-docs-stable/dev/libs/gelly/library_methods.html)，如 label propagation、triangle enumeration 和 page rank 算法，也提供了一个简化自定义图算法实现的 [Graph API](https://nightlies.apache.org/flink/flink-docs-stable/dev/libs/gelly/graph_api.html)。

## 运维

Apache Flink 是一个针对无界和有界数据流进行有状态计算的框架。由于许多流应用程序旨在以最短的停机时间连续运行，因此流处理器必须提供出色的故障恢复能力，以及在应用程序运行期间进行监控和维护的工具。

Apache Flink 非常注重流数据处理的可运维性。因此在这一小节中，我们将详细介绍 Flink 的故障恢复机制，并介绍其管理和监控应用的功能。

### 7 * 24小时稳定运行

在分布式系统中，服务故障是常有的事，为了保证服务能够7*24小时稳定运行，像Flink这样的流处理器故障恢复机制是必须要有的。显然这就意味着，它(这类流处理器)不仅要能在服务出现故障时候能够重启服务，而且还要当故障发生时，保证能够持久化服务内部各个组件的当前状态，只有这样才能保证在故障恢复时候，服务能够继续正常运行，好像故障就没有发生过一样。

Flink通过几下多种机制维护应用可持续运行及其一致性:

- **检查点的一致性**: Flink的故障恢复机制是通过建立分布式应用服务状态一致性检查点实现的，当有故障产生时，应用服务会重启后，再重新加载上一次成功备份的状态检查点信息。结合可重放的数据源，该特性可保证*精确一次（exactly-once）*的状态一致性。
- **高效的检查点**: 如果一个应用要维护一个TB级的状态信息，对此应用的状态建立检查点服务的资源开销是很高的，为了减小因检查点服务对应用的延迟性（SLAs服务等级协议）的影响，Flink采用异步及增量的方式构建检查点服务。
- **端到端的精确一次**: Flink 为某些特定的存储支持了事务型输出的功能，及时在发生故障的情况下，也能够保证精确一次的输出。
- **集成多种集群管理服务**: Flink已与多种集群管理服务紧密集成，如 [Hadoop YARN](https://hadoop.apache.org/), [Mesos](https://mesos.apache.org/), 以及 [Kubernetes](https://kubernetes.io/)。当集群中某个流程任务失败后，一个新的流程服务会自动启动并替代它继续执行。
- **内置高可用服务**: Flink内置了为解决单点故障问题的高可用性服务模块，此模块是基于[Apache ZooKeeper](https://zookeeper.apache.org/) 技术实现的，[Apache ZooKeeper](https://zookeeper.apache.org/)是一种可靠的、交互式的、分布式协调服务组件。

### Flink能够更方便地升级、迁移、暂停、恢复应用服务

驱动关键业务服务的流应用是经常需要维护的。比如需要修复系统漏洞，改进功能，或开发新功能。然而升级一个有状态的流应用并不是简单的事情，因为在我们为了升级一个改进后版本而简单停止当前流应用并重启时，我们还不能丢失掉当前流应用的所处于的状态信息。

而Flink的 *Savepoint* 服务就是为解决升级服务过程中记录流应用状态信息及其相关难题而产生的一种唯一的、强大的组件。一个 Savepoint，就是一个应用服务状态的一致性快照，因此其与checkpoint组件的很相似，但是与checkpoint相比，Savepoint 需要手动触发启动，而且当流应用服务停止时，它并不会自动删除。Savepoint 常被应用于启动一个已含有状态的流服务，并初始化其（备份时）状态。Savepoint 有以下特点：

- **便于升级应用服务版本**: Savepoint 常在应用版本升级时使用，当前应用的新版本更新升级时，可以根据上一个版本程序记录的 Savepoint 内的服务状态信息来重启服务。它也可能会使用更早的 Savepoint 还原点来重启服务，以便于修复由于有缺陷的程序版本导致的不正确的程序运行结果。
- **方便集群服务移植**: 通过使用 Savepoint，流服务应用可以自由的在不同集群中迁移部署。
- **方便Flink版本升级**: 通过使用 Savepoint，可以使应用服务在升级Flink时，更加安全便捷。
- **增加应用并行服务的扩展性**: Savepoint 也常在增加或减少应用服务集群的并行度时使用。
- **便于A/B测试及假设分析场景对比结果**: 通过把同一应用在使用不同版本的应用程序，基于同一个 Savepoint 还原点启动服务时，可以测试对比2个或多个版本程序的性能及服务质量。
- **暂停和恢复服务**: 一个应用服务可以在新建一个 Savepoint 后再停止服务，以便于后面任何时间点再根据这个实时刷新的 Savepoint 还原点进行恢复服务。
- **归档服务**: Savepoint 还提供还原点的归档服务，以便于用户能够指定时间点的 Savepoint 的服务数据进行重置应用服务的状态，进行恢复服务。

### 监控和控制应用服务

如其它应用服务一样，持续运行的流应用服务也需要监控及集成到一些基础设施资源管理服务中，例如一个组件的监控服务及日志服务等。监控服务有助于预测问题并提前做出反应，日志服务提供日志记录能够帮助追踪、调查、分析故障发生的根本原因。最后，便捷易用的访问控制应用服务运行的接口也是Flink的一个重要的亮点特征。

Flink与许多常见的日志记录和监视服务集成得很好，并提供了一个REST API来控制应用服务和查询应用信息。具体表现如下：

- **Web UI方式**: Flink提供了一个web UI来观察、监视和调试正在运行的应用服务。并且还可以执行或取消组件或任务的执行。
- **日志集成服务**:Flink实现了流行的slf4j日志接口，并与日志框架[log4j](https://logging.apache.org/log4j/2.x/)或[logback](https://logback.qos.ch/)集成。
- **指标服务**: Flink提供了一个复杂的度量系统来收集和报告系统和用户定义的度量指标信息。度量信息可以导出到多个报表组件服务，包括 [JMX](https://en.wikipedia.org/wiki/Java_Management_Extensions), Ganglia, [Graphite](https://graphiteapp.org/), [Prometheus](https://prometheus.io/), [StatsD](https://github.com/etsy/statsd), [Datadog](https://www.datadoghq.com/), 和 [Slf4j](https://www.slf4j.org/).
- **标准的WEB REST API接口服务**: Flink提供多种REST API接口，有提交新应用程序、获取正在运行的应用程序的Savepoint服务信息、取消应用服务等接口。REST API还提供元数据信息和已采集的运行中或完成后的应用服务的指标信息。



## WordCount

word.txt

```
hello world
hello flink
hello java
```

### 批处理

我们进行单词频次统计的基本思路是：先逐行读入文件数据，然后将每一行文字拆分成单词；接着按照单词分组，统计每组数据的个数，就是对应单词的频次。

Flink 同时提供了 Java 和 Scala 两种语言的 API，有些类在两套 API 中名称是一样的。 所以在引入包时，如果有 Java 和 Scala 两种选择，要注意选用 Java 的包。

```java
public class BatchWordCount {
    public static void main(String[] args) throws Exception {
        //创建执行环境
        ExecutionEnvironment executionEnvironment = ExecutionEnvironment.getExecutionEnvironment();
        //从文件读取数据，按行读取(存储的元素就是每行的文本)
        DataSource<String> dataSource = executionEnvironment.readTextFile("Flink/src/main/resources/word.txt");
        //转换数据格式
        FlatMapOperator<String, Tuple2<String, Long>> wordAndOne = dataSource
                .flatMap((String line, Collector<Tuple2<String, Long>> out) -> {
                    String[] words = line.split(" ");
                    for (String word : words) {
                        out.collect(Tuple2.of(word, 1L));
                    }
                })
                //当 Lambda 表达式使用 Java 泛型的时候, 由于泛型擦除的存在, 需要显示的声明类型信息
                .returns(Types.TUPLE(Types.STRING, Types.LONG));
        UnsortedGrouping<Tuple2<String, Long>> wordAndOneUG = wordAndOne.groupBy(0);
        AggregateOperator<Tuple2<String, Long>> sum = wordAndOneUG.sum(1);
        sum.print();
    }
}
```

![image-20220921094742776](Flink.assets/image-20220921094742776.png)



需要注意的是，这种代码的实现方式，是基于 DataSet API 的，也就是我们对数据的处理转换，是看作数据集来进行操作的。事实上 Flink 本身是流批统一的处理架构，批量的数据集本质上也是流，没有必要用两套不同的 API 来实现。所以从 Flink 1.12 开始，官方推荐的做法是直接使用 DataStream API，在提交任务时通过将执行模式设为 BATCH 来进行批处理：

```
bin/flink run -Dexecution.runtime-mode=BATCH BatchWordCount.jar
```

这样，DataSet API 就已经处于“软弃用”（soft deprecated）的状态，在实际应用中只要维护一套 DataStream API 就可以了。这里只是为了方便理解，依然用 DataSet API 做了批处理的实现。

### 流处理

在 Flink 的视角里，一切数据都可以认为是流，流数据是无界流，而批数据则是有界流。所以批处理，其实就可以看作有界流的处理。 

对于流而言，我们会在获取输入数据后立即处理，这个过程是连续不断的。当然，有时我们的输入数据可能会有尽头，这看起来似乎就成了一个有界流；但是它跟批处理是截然不同的 ——在输入结束之前，我们依然会认为数据是无穷无尽的，处理的模式也仍旧是连续逐个处理。 下面我们就针对不同类型的输入数据源，用具体的代码来实现流处理。

#### 读取文件（有界数据流）

```java
public class BoundedStreamWordCount {
    public static void main(String[] args) throws Exception {
        //创建流式执行环境
        StreamExecutionEnvironment streamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
        //读取文件
        DataStream<String> stringDataStream = streamExecutionEnvironment.readTextFile("Flink/src/main/resources/word.txt");
        //转换数据格式
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = stringDataStream
                .flatMap((String line, Collector<String> words) -> {
                    Arrays.stream(line.split(" ")).forEach(words::collect);
                })
                .returns(Types.STRING)
                .map(word -> Tuple2.of(word, 1L))
                .returns(Types.TUPLE(Types.STRING, Types.LONG));
        //分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne.keyBy(t -> t.f0);
        //求和
        SingleOutputStreamOperator<Tuple2<String, Long>> sum = wordAndOneKS.sum(1);
        //打印
        sum.print();
        //执行
        streamExecutionEnvironment.execute();
    }
}
```

主要观察与批处理程序 BatchWordCount 的不同： 

- 创建执行环境的不同，流处理程序使用的是 StreamExecutionEnvironment。
- 每一步处理转换之后，得到的数据对象类型不同。
- 分组操作调用的是 keyBy 方法，可以传入一个匿名函数作为键选择器 （KeySelector），指定当前分组的 key 是什么。
- 代码末尾需要调用 streamExecutionEnvironment 的 execute 方法，开始执行任务。



![image-20220921095006670](Flink.assets/image-20220921095006670.png)

可以看到，这与批处理的结果是完全不同的。批处理针对每个单词，只会输出一个最终的统计个数；而在流处理的打印结果中，“hello”这个单词每出现一次，都会有一个频次统计数据输出。这就是流处理的特点，数据逐个处理，每来一条数据就会处理输出一次。我们通过 打印结果，可以清晰地看到单词“hello”数量增长的过程。

### 读取文本流（无界数据流）

在实际的生产环境中，真正的数据流其实是无界的，有开始却没有结束，这就要求我们需要保持一个监听事件的状态，持续地处理捕获的数据。 

为了模拟这种场景，我们就不再通过读取文件来获取数据了，而是监听数据发送端主机的指定端口，统计发送来的文本数据中出现过的单词的个数。具体实现上，我们只要对 BoundedStreamWordCount 代码中读取数据的步骤稍做修改，就可以实现对真正无界流的处理。

```java
public class StreamWordCount {
    public static void main(String[] args) throws Exception{
        //创建流式执行环境
        StreamExecutionEnvironment streamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
        //读取文本流
        DataStream<String> dataStream = streamExecutionEnvironment.socketTextStream("10.10.11.146", 1234);
        //转换数据格式
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = dataStream
                .flatMap((String line, Collector<String> words) -> {
                    Arrays.stream(line.split(" ")).forEach(words::collect);
                })
                .returns(Types.STRING)
                .map(word -> Tuple2.of(word, 1L))
                .returns(Types.TUPLE(Types.STRING, Types.LONG));
        //分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne.keyBy(t -> t.f0);
        //求和
        SingleOutputStreamOperator<Tuple2<String, Long>> sum = wordAndOneKS.sum(1);
        //打印
        sum.print();
        //执行
        streamExecutionEnvironment.execute();
    }
}
```

注意事项： 

- socket 文本流的读取需要配置两个参数：发送端主机名和端口号。这里代码中指定了主机“10.10.11.146”的 1234 端口作为发送数据的 socket 端口，需要根据测试环境自行配置。
- 在实际项目应用中，主机名和端口号这类信息往往可以通过配置文件，或者传入程序运行参数的方式来指定。
- socket文本流数据的发送，可以通过Linux系统自带的netcat工具进行模拟。

在 Linux 环境的主机上，执行下列命令，发送数据进行测试：

```
nc -lk 1234
```

![image-20220921095713686](Flink.assets/image-20220921095713686.png)

可以发现，输出的结果与之前读取文件的流处理非常相似。



# 状态编程

Flink 处理机制的核心，就是“有状态的流式计算”。

在 Flink 这样的分布式系统中，我们不仅需要定义出状态在任务并行时的处理方式，还需要考虑如何持久化保存、以便发生故障时正确地恢复。这就需要一套完整的管理机制来处理所有的状态。

## Flink中的状态

在流处理中，数据是连续不断到来和处理的。每个任务进行计算处理时，可以基于当前数据直接转换得到输出结果；也可以依赖一些其他数据。这些由一个任务维护，并且用来计算输出结果的所有数据，就叫作这个任务的状态。

> 状态在Flink中叫做State，用来保存中间计算结果或者缓存数据。**State是实现有状态的计算下Exactly-Once的基础**。

### 有状态算子

在 Flink 中，算子任务可以分为无状态和有状态两种情况。

无状态的算子任务只需要观察每个独立事件，根据当前输入的数据直接转换输出结果，如下图所示。例如，可以将一个字符串类型的数据拆分开作为元组输出；也可以对数据做一些计算，比如每个代表数量的字段加 1。如 map、filter、flatMap， 计算时不依赖其他数据，就都属于无状态的算子。

![image-20220922132512219](Flink.assets/image-20220922132512219.png)

有状态的算子任务，则除当前数据之外，还需要一些其他数据来得到计算结果。这里的 “其他数据”，就是所谓的状态（state），最常见的就是之前到达的数据，或者由之前数据计算出的某个结果。比如，做求和（sum）计算时，需要保存之前所有数据的和，这就是状态；窗口算子中会保存已经到达的所有数据，这些也都是它的状态。

如果希望检索到某种 “事件模式”（event pattern），比如“先有下单行为，后有支付行为”，那么也应该把之前的行为保存下来，这同样属于状态。容易发现，之前讲过的聚合算子、窗口算子都属于有状态的算子。

![image-20220922132914907](Flink.assets/image-20220922132914907.png)

上图所示为有状态算子的一般处理流程，具体步骤如下。 

1. 算子任务接收到上游发来的数据；
2. 获取当前状态； 
3. 根据业务逻辑进行计算，更新状态；
4. 得到计算结果，输出发送到下游任务。

### 状态的管理

在传统的事务型处理架构中，这种额外的状态数据是保存在数据库中的。而对于实时流处理来说，这样做需要频繁读写外部数据库，如果数据规模非常大肯定就达不到性能要求了。所以 Flink 的解决方案是，将状态直接保存在内存中来保证性能，并通过分布式扩展来提高吞吐量。

在 Flink 中，每一个算子任务都可以设置并行度，从而可以在不同的 slot 上并行运行多个实例，我们把它们叫作“并行子任务”。而状态既然在内存中，那么就可以认为是子任务实例上的一个本地变量，能够被任务的业务逻辑访问和修改。 

这样看来状态的管理似乎非常简单，我们直接把它作为一个对象交给 JVM 就可以了。然而大数据的场景下，我们必须使用分布式架构来做扩展，在低延迟、高吞吐的基础上还要保证容错性，一系列复杂的问题就会随之而来了。

- 状态的访问权限。我们知道 Flink 上的聚合和窗口操作，一般都是基于 KeyedStream 的，数据会按照 key 的哈希值进行分区，聚合处理的结果也应该是只对当前 key 有效。 然而同一个分区（也就是 slot）上执行的任务实例，可能会包含多个 key 的数据，它们同时访问和更改本地变量，就会导致计算结果错误。所以这时状态并不是单纯的本地变量。

> KeyedStream继承了DataStream，是由datastream的keyBy()，产生的。表示按key的value分区过的流。在datastream的功能基础上，由添加了一些max,min等聚合的功能。

- 容错性，也就是故障后的恢复。状态只保存在内存中显然是不够稳定的，我们需要将它持久化保存，做一个备份；在发生故障后可以从这个备份中恢复状态。
- 我们还应该考虑到分布式应用的横向扩展性。比如处理的数据量增大时，我们应该相应地对计算资源扩容，调大并行度。这时就涉及到了状态的重组调整。

Flink 有一套完整的状态管理机制，将底层一些核心功能全部封装起来，包括状态的高效存储和访问、持久化保存和故障恢复，以及资源扩展时的调整。这样，我们只需要调用相应的 API 就可以很方便地使用状态，或对应用的容错机制进行配置，从而将更多的精力放在业务逻辑的开发上。

### 状态的分类

#### 托管状态（Managed State）和原始状态（Raw State）

Flink 的状态有两种：托管状态（Managed State）和原始状态（Raw State）。托管状态就是由 Flink 统一管理的，状态的存储访问、故障恢复和重组等一系列问题都由 Flink 实现，我们只要调接口就可以；而原始状态则是自定义的，相当于就是开辟了一块内存，需要我们自己管理，实现状态的序列化和故障恢复。 

具体来讲，托管状态是由 Flink 的运行时（Runtime）来托管的；在配置容错机制后，状态会自动持久化保存，并在发生故障时自动恢复。当应用发生横向扩展时，状态也会自动地重组分配到所有的子任务实例上。对于具体的状态内容，Flink 也提供了值状态（ValueState）、 列表状态（ListState）、映射状态（MapState）、聚合状态（AggregateState）等多种结构，内部支持各种数据类型。聚合、窗口等算子中内置的状态，就都是托管状态；我们也可以在富函数类（RichFunction）中通过上下文来自定义状态，这些也都是托管状态。 

而对比之下，原始状态就全部需要自定义了。Flink 不会对状态进行任何自动操作，也不知道状态的具体数据类型，只会把它当作最原始的字节（Byte）数组来存储。我们需要花费大量的精力来处理状态的管理和维护。 

所以只有在遇到托管状态无法实现的特殊需求时，我们才会考虑使用原始状态；一般情况下不推荐使用。绝大多数应用场景，我们都可以用 Flink 提供的算子或者自定义托管状态来实现需求。

#### 算子状态（Operator State）和按键分区状态（Keyed State）

在 Flink 中，一个算子任务会按照并行度分为多个并行子任务执行，而不同的子任务会占据不同的任务槽（task slot）。由于不同的 slot 在计算资源上是物理隔离的，所以 Flink 能管理的状态在并行任务间是无法共享的，每个状态只能针对当前子任务的实例有效。 

而很多有状态的操作（比如聚合、窗口）都是要先做 keyBy 进行按键分区的。按键分区之后，任务所进行的所有计算都应该只针对当前 key 有效，所以状态也应该按照 key 彼此隔离。 在这种情况下，状态的访问方式又会有所不同。 

基于这样的想法，又可以将托管状态分为两类：算子状态和按键分区状态。

##### 算子状态（Operator State）

> 跟一个特定算子的实例绑定，整个算子只对应一个State对象(相同的并行算子都能访问到状态)。只支持ListState

状态作用范围限定为当前的算子任务实例，也就是只对当前并行子任务实例有效。这就意味着对于一个并行子任务，占据了一个“分区”，它所处理的所有数据都会访问到相同的状态， 状态对于同一任务而言是共享的

![image-20220922135140444](Flink.assets/image-20220922135140444.png)

算子状态可以用在所有算子上，使用的时候其实就跟一个本地变量没什么区别——因为本地变量的作用域也是当前任务实例。在使用时，我们还需进一步实现 CheckpointedFunction 接口。

##### 按键分区状态（Keyed State）

> 状态跟特定的key绑定，即每个KeyedStream对应一个State对象。支持多种State(ValueState、MapState、ListState等)

状态是根据输入流中定义的键（key）来维护和访问的，所以只能定义在按键分区流 （KeyedStream）中，也就 keyBy 之后才可以使用

![image-20220922135515415](Flink.assets/image-20220922135515415.png)

按键分区状态应用非常广泛。之前讲到的聚合算子必须在 keyBy 之后才能使用，就是因为聚合的结果是以 Keyed State 的形式保存的。另外，也可以通过富函数类（Rich Function） 来自定义 Keyed State，所以只要提供了富函数类接口的算子，也都可以使用 Keyed State。 

所以即使是 map、filter 这样无状态的基本转换算子，我们也可以通过富函数类给它们“追加”Keyed State，或者实现CheckpointedFunction 接口来定义 Operator State；从这个角度讲， Flink 中所有的算子都可以是有状态的。

无论是 Keyed State 还是 Operator State，它们都是在本地实例上维护的，也就是说每个并行子任务维护着对应的状态，算子的子任务之间状态不共享。

## 按键分区状态（Keyed State）

在实际应用中，我们一般都需要将数据按照某个 key 进行分区，然后再进行计算处理；所以最为常见的状态类型就是 Keyed State。

keyBy 之后的聚合、窗口计算，算子所持有的状态，都是 Keyed State。 另外，我们还可以通过富函数类（Rich Function）对转换算子进行扩展、实现自定义功能， 比如 RichMapFunction、RichFilterFunction。在富函数中，我们可以调用.getRuntimeContext() 获取当前的运行时上下文（RuntimeContext），进而获取到访问状态的句柄；这种富函数中自定义的状态也是 Keyed State。

### 简介

按键分区状态（Keyed State）顾名思义，是任务按照键（key）来访问和维护的状态。它的特点非常鲜明，就是以 key 为作用范围进行隔离。 

在进行按键分区（keyBy）之后，具有相同键的所有数据，都会分配到同一个并行子任务中；所以如果当前任务定义了状态，Flink 就会在当前并行子任务实例中，为每个键值维护一个状态的实例。于是当前任务就会为分配来的所有数据，按照 key 维护和处理对应的状态。 

因为一个并行子任务可能会处理多个 key 的数据，所以 Flink 需要对 Keyed State 进行一些特殊优化。在底层，Keyed State 类似于一个分布式的映射（map）数据结构，所有的状态会根据 key 保存成键值对（key-value）的形式。这样当一条数据到来时，任务就会自动将状态的访问范围限定为当前数据的 key，从 map 存储中读取出对应的状态值。所以具有相同 key 的所有数据都会到访问相同的状态，而不同 key 的状态之间是彼此隔离的。 这种将状态绑定到 key 上的方式，相当于使得状态和流的逻辑分区一一对应了：不会有别的 key 的数据来访问当前状态；而当前状态对应 key 的数据也只会访问这一个状态，不会分发到其他分区去。这就保证了对状态的操作都是本地进行的，对数据流和状态的处理做到了分区一致性。

另外，在应用的并行度改变时，状态也需要随之进行重组。不同 key 对应的 Keyed State 可以进一步组成所谓的键组（key groups），每一组都对应着一个并行子任务。键组是 Flink 重新分配 Keyed State 的单元，键组的数量就等于定义的最大并行度。当算子并行度发生改变时， Keyed State 就会按照当前的并行度重新平均分配，保证运行时各个子任务的负载相同。 

需要注意，使用 Keyed State 必须基于 KeyedStream。没有进行 keyBy 分区的 DataStream， 即使转换算子实现了对应的富函数类，也不能通过运行时上下文访问 Keyed State。

###  支持的结构类型

实际应用中，需要保存为状态的数据会有各种各样的类型，有时还需要复杂的集合类型， 比如列表（List）和映射（Map）。对于这些常见的用法，Flink 的按键分区状态（Keyed State） 提供了足够的支持。

#### 值状态（Value State）

顾名思义，状态中只保存一个“值”（value）。ValueState本身是一个接口，源码中定义如下：

```java
@PublicEvolving
public interface ValueState<T> extends State {
    T value() throws IOException;
    void update(T value) throws IOException;
}
```

这里的 T 是泛型，表示状态的数据内容可以是任何具体的数据类型。如果想要保存一个长整型值作为状态，那么类型就是 ValueState。 我们可以在代码中读写值状态，实现对于状态的访问和更新。 

- T value()：获取当前状态的值；
- update(T value)：对状态进行更新，传入的参数 value 就是要覆写的状态值。 

在具体使用时，为了让运行时上下文清楚到底是哪个状态，我们还需要创建一个“状态描述器”（StateDescriptor）来提供状态的基本信息。例如源码中，ValueState 的状态描述器一个构造方法如下：

```java
public ValueStateDescriptor(String name, Class<T> typeClass) {
    super(name, typeClass, null);
}
```

这里需要传入状态的名称和类型——这跟我们声明一个变量时做的事情完全一样。有了这个描述器，运行时环境就可以获取到状态的控制句柄（handler）了。

#### 列表状态（ListState）

将需要保存的数据，以列表（List）的形式组织起来。

在 ListState接口中同样有一个类型参数 T，表示列表中数据的类型。ListState 也提供了一系列的方法来操作状态，使用方式与一般的 List 非常相似。 

```java
@PublicEvolving
public interface ListState<T> extends MergingState<T, Iterable<T>> {
    //传入一个列表 values，直接对状态进行覆盖；
    void update(List<T> values) throws Exception;
    //向列表中添加多个元素，以列表 values 形式传入。 
    void addAll(List<T> values) throws Exception;
}
```

类似地，ListState 的状态描述器就叫作 ListStateDescriptor，用法跟 ValueStateDescriptor 完全一致。

#### 映射状态（MapState）

 把一些键值对（key-value）作为状态整体保存起来，可以认为就是一组 key-value 映射的列表。对应的 MapState<UK,UV>接口中，就会有 UK、UV 两个泛型，分别表示保存的 key 和 value 的类型。同样，MapState 提供了操作映射状态的方法，与 Map 的使用非常类似。 

```java
@PublicEvolving
public interface MapState<UK, UV> extends State {
    //传入一个 key 作为参数，查询对应的 value 值；
    UV get(UK key) throws Exception;
    //传入一个键值对，更新 key 对应的 value 值；
    void put(UK key, UV value) throws Exception;
    //将传入的映射 map 中所有的键值对，全部添加到映射状态中；
    void putAll(Map<UK, UV> map) throws Exception;
    //将指定 key 对应的键值对删除；
    void remove(UK key) throws Exception;
    //判断是否存在指定的 key，返回一个 boolean 值。
    boolean contains(UK key) throws Exception;
    //获取映射状态中所有的键值对；
    Iterable<Map.Entry<UK, UV>> entries() throws Exception;
    //获取映射状态中所有的键（key），返回一个可迭代 Iterable 类型；
    Iterable<UK> keys() throws Exception;
    //获取映射状态中所有的值（value），返回一个可迭代 Iterable 类型；
    Iterable<UV> values() throws Exception;
    //获取映射状态中所有的映射（mapping），返回一个可迭代 Iterable 类型
    Iterator<Map.Entry<UK, UV>> iterator() throws Exception;
    //判断映射是否为空，返回一个 boolean 值。
    boolean isEmpty() throws Exception;
}
```

#### 归约状态（ReducingState）

类似于值状态（Value），不过需要对添加进来的所有数据进行归约，将归约聚合之后的值作为状态保存下来。

ReducintState这个接口调用的方法类似于 ListState，只不过它保存的只是一个聚合值，所以调用.add()方法时，不是在状态列表里添加元素，而是直接把新数据和之前的状态进行归约，并用得到的结果更新状态。 

归约逻辑的定义，是在归约状态描述器（ReducingStateDescriptor）中，通过传入一个归约函数（ReduceFunction）来实现的。这里的归约函数，就是ReduceFunction，所以状态类型跟输入的数据类型是一样的。

```java
@PublicEvolving
public class ReducingStateDescriptor<T> extends StateDescriptor<ReducingState<T>, T> {

    private static final long serialVersionUID = 1L;

    private final ReduceFunction<T> reduceFunction;

    public ReducingStateDescriptor(
            String name, ReduceFunction<T> reduceFunction, Class<T> typeClass) {
        super(name, typeClass, null);
        this.reduceFunction = checkNotNull(reduceFunction);

        if (reduceFunction instanceof RichFunction) {
            throw new UnsupportedOperationException(
                    "ReduceFunction of ReducingState can not be a RichFunction.");
        }
    }

    public ReducingStateDescriptor(
            String name, ReduceFunction<T> reduceFunction, TypeInformation<T> typeInfo) {
        super(name, typeInfo, null);
        this.reduceFunction = checkNotNull(reduceFunction);
    }

    public ReducingStateDescriptor(
            String name, ReduceFunction<T> reduceFunction, TypeSerializer<T> typeSerializer) {
        super(name, typeSerializer, null);
        this.reduceFunction = checkNotNull(reduceFunction);
    }

    /** Returns the reduce function to be used for the reducing state. */
    public ReduceFunction<T> getReduceFunction() {
        return reduceFunction;
    }

    @Override
    public Type getType() {
        return Type.REDUCING;
    }
}
```

这里的描述器有三个参数，其中第二个参数就是定义了归约聚合逻辑的 ReduceFunction， 另外两个参数则是状态的名称和类型。

#### 聚合状态（AggregatingState）

与归约状态非常类似，聚合状态也是一个值，用来保存添加进来的所有数据的聚合结果。 与 ReducingState 不同的是，它的聚合逻辑是由在描述器中传入一个更加一般化的聚合函数（AggregateFunction）来定义的；这也就是之前我们讲过的 AggregateFunction，里面通过一个 累加器（Accumulator）来表示状态，所以聚合的状态类型可以跟添加进来的数据类型完全不同，使用更加灵活。

同样地，AggregatingState 接口调用方法也与 ReducingState 相同，调用.add()方法添加元素时，会直接使用指定的 AggregateFunction 进行聚合并更新状态。

### 代码实现

了解了按键分区状态（Keyed State）的基本概念和类型，接下来我们就可以尝试在代码中使用状态了。

#### 整体介绍

在 Flink 中，状态始终是与特定算子相关联的；算子在使用状态前首先需要“注册”，其实就是告诉 Flink 当前上下文中定义状态的信息，这样运行时的 Flink 才能知道算子有哪些状态。 

状态的注册，主要是通过“状态描述器”（StateDescriptor）来实现的。状态描述器中最重要的内容，就是状态的名称（name）和类型（type）。Flink 中的状态，可以认为是加了一些复杂操作的内存中的变量；而当我们在代码中声明一个局部变量时，都需要指定变量类型和名称，名称就代表了变量在内存中的地址，类型则指定了占据内存空间的大小。同样地， 我们一旦指定了名称和类型，Flink 就可以在运行时准确地在内存中找到对应的状态，进而返回状态对象供我们使用了。所以在一个算子中，我们也可以定义多个状态，只要它们的名称不同就可以了。 

另外，状态描述器中还可能需要传入一个用户自定义函数（user-defined-function，UDF）， 用来说明处理逻辑，比如前面提到的 ReduceFunction 和 AggregateFunction。

以 ValueState 为例，可以定义值状态描述器如下：

```java
ValueStateDescriptor<Long> descriptor = new ValueStateDescriptor<>(
	"my state", // 状态名称
	Types.LONG // 状态类型
);
```

这里我们定义了一个叫作“my state”的长整型 ValueState 的描述器。

代码中完整的操作是，首先定义出状态描述器；然后调用.getRuntimeContext()方法获取运行时上下文；继而调用 RuntimeContext 的获取状态的方法，将状态描述器传入，就可以得到对应的状态了。 

因为状态的访问需要获取运行时上下文，这只能在富函数类（Rich Function）中获取到， 所以自定义的 Keyed State 只能在富函数中使用。当然，底层的处理函数（Process Function） 本身继承了 AbstractRichFunction 抽象类，所以也可以使用。 

在富函数中，调用.getRuntimeContext()方法获取到运行时上下文之后，RuntimeContext 有以下几个获取状态的方法：

```java
alueState<T> getState(ValueStateDescriptor<T>)
MapState<UK, UV> getMapState(MapStateDescriptor<UK, UV>)
ListState<T> getListState(ListStateDescriptor<T>)
ReducingState<T> getReducingState(ReducingStateDescriptor<T>)
AggregatingState<IN, OUT> getAggregatingState(AggregatingStateDescriptor<IN,ACC,OUT>)
```

对于不同结构类型的状态，只要传入对应的描述器、调用对应的方法就可以了。 获取到状态对象之后，就可以调用它们各自的方法进行读写操作了。另外，所有类型的状 态都有一个方法.clear()，用于清除当前状态。

 代码中使用状态的整体结构如下：

```java
public static class MyFlatMapFunction extends RichFlatMapFunction<Long, String>
{
    // 声明状态
    private transient ValueState<Long> state;
    @Override
    public void open(Configuration config) {
        // 在 open 生命周期方法中获取状态
        ValueStateDescriptor<Long> descriptor = new ValueStateDescriptor<>(
                "my state", // 状态名称
                Types.LONG // 状态类型
        );
        state = getRuntimeContext().getState(descriptor);
    }
    @Override
    public void flatMap(Long input, Collector<String> out) throws Exception {
        // 访问状态
        Long currentState = state.value();
        currentState += 1; // 状态数值加 1
        // 更新状态
        state.update(currentState);
        if (currentState >= 100) {
            out.collect(“state: ” + currentState);
            state.clear(); // 清空状态
        }
    }
}
```

因为 RichFlatmapFunction 中的.flatmap()是每来一条数据都会调用一次的，所以我们不应该在这里调用运行时上下文的.getState()方法，而是在生命周期方法.open()中获取状态对象。

另外还有一个问题，我们获取到的状态对象也需要有一个变量名称 state（注意这里跟状态的名称 my state 不同），但这个变量不应该在 open 中声明——否则在.flatmap()里就访问不到了。所以我们还需要在外面直接把它定义为类的属性，这样就可以在不同的方法中通用了。而在外部又不能直接获取状态，因为编译时是无法拿到运行时上下文的。所以最终的解决方案就变成了： 在外部声明状态对象，在 open 生命周期方法中通过运行时上下文获取状态。 

这里需要注意，这种方式定义的都是 Keyed State，它对于每个 key 都会保存一份状态实例。所以对状态进行读写操作时，获取到的状态跟当前输入数据的 key 有关；只有相同 key 的数据，才会操作同一个状态，不同 key 的数据访问到的状态值是不同的。而且上面提到 的.clear()方法，也只会清除当前 key 对应的状态。 

另外，状态不一定都存储在内存中，也可以放在磁盘或其他地方，具体的位置是由一个可 配置的组件来管理的，这个组件叫作“状态后端”（State Backend）。

#### 值状态（ValueState）

我们这里会使用用户 id 来进行分流，然后分别统计每个用户的 pv 数据，由于我们并不想每次 pv 加一，就将统计结果发送到下游去，所以这里我们注册了一个定时器，用来隔一段时间发送 pv 的统计结果，这样对下游算子的压力不至于太大。具体实现方式是定义一个用来保 存定时器时间戳的值状态变量。当定时器触发并向下游发送数据以后，便清空储存定时器时间 戳的状态变量，这样当新的数据到来时，发现并没有定时器存在，就可以注册新的定时器了， 注册完定时器之后将定时器的时间戳继续保存在状态变量中。

```java
public class PeriodicPVExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );

        stream.print("input");

        // 统计每个用户的pv，隔一段时间（10s）输出一次结果
        stream.keyBy(data -> data.user)
                .process(new PeriodicPvResult())
                .print();

        env.execute();
    }

    // 注册定时器，周期性输出pv
    public static class PeriodicPvResult extends KeyedProcessFunction<String, Event, String> {
        // 定义两个状态，保存当前pv值，以及定时器时间戳
        ValueState<Long> countState;
        ValueState<Long> timerTsState;

        @Override
        public void open(Configuration parameters) throws Exception {
            countState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("count", Long.class));
            timerTsState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("timerTs", Long.class));
        }

        @Override
        public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
            // 更新count值
            Long count = countState.value();
            if (count == null) {
                countState.update(1L);
            } else {
                countState.update(count + 1);
            }
            // 注册定时器
            if (timerTsState.value() == null) {
                ctx.timerService().registerEventTimeTimer(value.timestamp + 10 * 1000L);
                timerTsState.update(value.timestamp + 10 * 1000L);
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            out.collect(ctx.getCurrentKey() + " pv: " + countState.value());
            // 清空状态
            timerTsState.clear();
        }
    }
}
```

#### 列表状态（ListState）

在 Flink SQL 中，支持两条流的全量 Join，语法如下：

```sql
SELECT * FROM A INNER JOIN B WHERE A.id = B.id；
```

 这样一条 SQL 语句要慎用，因为 Flink 会将 A 流和 B 流的所有数据都保存下来，然后进行 Join。

不过在这里我们可以用列表状态变量来实现一下这个 SQL 语句的功能。代码如下：

```java
public class TwoStreamFullJoinExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple3<String, String, Long>> stream1 = env
                .fromElements(
                        Tuple3.of("a", "stream-1", 1000L),
                        Tuple3.of("b", "stream-1", 2000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                                    @Override
                                    public long extractTimestamp(Tuple3<String, String, Long> t, long l) {
                                        return t.f2;
                                    }
                                })
                );

        SingleOutputStreamOperator<Tuple3<String, String, Long>> stream2 = env
                .fromElements(
                        Tuple3.of("a", "stream-2", 3000L),
                        Tuple3.of("b", "stream-2", 4000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                                    @Override
                                    public long extractTimestamp(Tuple3<String, String, Long> t, long l) {
                                        return t.f2;
                                    }
                                })
                );

        stream1.keyBy(r -> r.f0)
                .connect(stream2.keyBy(r -> r.f0))
                .process(new CoProcessFunction<Tuple3<String, String, Long>, Tuple3<String, String, Long>, String>() {
                    private ListState<Tuple3<String, String, Long>> stream1ListState;
                    private ListState<Tuple3<String, String, Long>> stream2ListState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        super.open(parameters);
                        stream1ListState = getRuntimeContext().getListState(
                                new ListStateDescriptor<Tuple3<String, String, Long>>("stream1-list", Types.TUPLE(Types.STRING, Types.STRING))
                        );
                        stream2ListState = getRuntimeContext().getListState(
                                new ListStateDescriptor<Tuple3<String, String, Long>>("stream2-list", Types.TUPLE(Types.STRING, Types.STRING))
                        );
                    }

                    @Override
                    public void processElement1(Tuple3<String, String, Long> left, Context context, Collector<String> collector) throws Exception {
                        stream1ListState.add(left);
                        for (Tuple3<String, String, Long> right : stream2ListState.get()) {
                            collector.collect(left + " => " + right);
                        }
                    }

                    @Override
                    public void processElement2(Tuple3<String, String, Long> right, Context context, Collector<String> collector) throws Exception {
                        stream2ListState.add(right);
                        for (Tuple3<String, String, Long> left : stream1ListState.get()) {
                            collector.collect(left + " => " + right);
                        }
                    }
                })
                .print();

        env.execute();
    }
}
```

![image-20220922165732608](Flink.assets/image-20220922165732608.png)

#### 映射状态（MapState）

映射状态的用法和 Java 中的 HashMap 很相似。在这里我们可以通过 MapState 的使用来探索一下窗口的底层实现，也就是我们要用映射状态来完整模拟窗口的功能。这里我们模拟一个滚动窗口。我们要计算的是每一个 url 在每一个窗口中的 pv 数据。

```java
// 使用KeyedProcessFunction模拟滚动窗口
public class FakeWindowExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );

        // 统计每10s窗口内，每个url的pv
        stream.keyBy(data -> data.url)
                .process(new FakeWindowResult(10000L))
                .print();

        env.execute();
    }

    public static class FakeWindowResult extends KeyedProcessFunction<String, Event, String> {
        // 定义属性，窗口长度
        private Long windowSize;

        public FakeWindowResult(Long windowSize) {
            this.windowSize = windowSize;
        }

        // 声明状态，用map保存pv值（窗口start，count）
        MapState<Long, Long> windowPvMapState;

        @Override
        public void open(Configuration parameters) throws Exception {
            windowPvMapState = getRuntimeContext().getMapState(new MapStateDescriptor<Long, Long>("window-pv", Long.class, Long.class));
        }

        @Override
        public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
            // 每来一条数据，就根据时间戳判断属于哪个窗口
            Long windowStart = value.timestamp / windowSize * windowSize;
            Long windowEnd = windowStart + windowSize;

            // 注册 end -1 的定时器，窗口触发计算
            ctx.timerService().registerEventTimeTimer(windowEnd - 1);

            // 更新状态中的pv值
            if (windowPvMapState.contains(windowStart)) {
                Long pv = windowPvMapState.get(windowStart);
                windowPvMapState.put(windowStart, pv + 1);
            } else {
                windowPvMapState.put(windowStart, 1L);
            }
        }

        // 定时器触发，直接输出统计的pv结果
        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            Long windowEnd = timestamp + 1;
            Long windowStart = windowEnd - windowSize;
            Long pv = windowPvMapState.get(windowStart);
            out.collect("url: " + ctx.getCurrentKey()
                    + " 访问量: " + pv
                    + " 窗口：" + new Timestamp(windowStart) + " ~ " + new Timestamp(windowEnd));

            // 模拟窗口的销毁，清除map中的key
            windowPvMapState.remove(windowStart);
        }
    }
}
```

#### 聚合状态（AggregatingState）

我们举一个简单的例子，对用户点击事件流每 5 个数据统计一次平均时间戳。这是一个类似计数窗口（CountWindow）求平均值的计算，这里我们可以使用一个有聚合状态的 RichFlatMapFunction 来实现。

```java
public class AverageTimestampExample {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );


        // 统计每个用户的点击频次，到达5次就输出统计结果
        stream.keyBy(data -> data.user)
                .flatMap(new AvgTsResult())
                .print();

        env.execute();
    }

    public static class AvgTsResult extends RichFlatMapFunction<Event, String> {
        // 定义聚合状态，用来计算平均时间戳
        AggregatingState<Event, Long> avgTsAggState;

        // 定义一个值状态，用来保存当前用户访问频次
        ValueState<Long> countState;

        @Override
        public void open(Configuration parameters) throws Exception {
            avgTsAggState = getRuntimeContext().getAggregatingState(new AggregatingStateDescriptor<Event, Tuple2<Long, Long>, Long>(
                    "avg-ts",
                    new AggregateFunction<Event, Tuple2<Long, Long>, Long>() {
                        @Override
                        public Tuple2<Long, Long> createAccumulator() {
                            return Tuple2.of(0L, 0L);
                        }

                        @Override
                        public Tuple2<Long, Long> add(Event value, Tuple2<Long, Long> accumulator) {
                            return Tuple2.of(accumulator.f0 + value.timestamp, accumulator.f1 + 1);
                        }

                        @Override
                        public Long getResult(Tuple2<Long, Long> accumulator) {
                            return accumulator.f0 / accumulator.f1;
                        }

                        @Override
                        public Tuple2<Long, Long> merge(Tuple2<Long, Long> a, Tuple2<Long, Long> b) {
                            return null;
                        }
                    },
                    Types.TUPLE(Types.LONG, Types.LONG)
            ));

            countState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("count", Long.class));
        }

        @Override
        public void flatMap(Event value, Collector<String> out) throws Exception {
            Long count = countState.value();
            if (count == null){
                count = 1L;
            } else {
                count ++;
            }

            countState.update(count);
            avgTsAggState.add(value);

            // 达到5次就输出结果，并清空状态
            if (count == 5){
                out.collect(value.user + " 平均时间戳：" + new Timestamp(avgTsAggState.get()));
                countState.clear();
            }
        }
    }
}
```

### 状态生存时间（TTL）

在实际应用中，很多状态会随着时间的推移逐渐增长，如果不加以限制，最终就会导致存储空间的耗尽。一个优化的思路是直接在代码中调用.clear()方法去清除状态，但是有时候我们的逻辑要求不能直接清除。这时就需要配置一个状态的“生存时间”（time-to-live，TTL），当状态在内存中存在的时间超出这个值时，就将它清除。

具体实现上，如果用一个进程不停地扫描所有状态看是否过期，显然会占用大量资源做无用功。状态的失效其实不需要立即删除，所以我们可以给状态附加一个属性，也就是状态的“失效时间”。状态创建的时候，设置失效时间 = 当前时间 + TTL；之后如果有对状态的访问和修改，我们可以再对失效时间进行更新；当设置的清除条件被触发时（比如，状态被访问的时候，或者每隔一段时间扫描一次失效状态），就可以判断状态是否失效、从而进行清除了。 

配置状态的 TTL 时，需要创建一个 StateTtlConfig 配置对象，然后调用状态描述器的.enableTimeToLive()方法启动 TTL 功能。

```java
ValueStateDescriptor<Event> valueStateDescriptor = new ValueStateDescriptor<>("my-state", Event.class);
// 配置状态的TTL
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.hours(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnReadAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build();

valueStateDescriptor.enableTimeToLive(ttlConfig);
```

这里用到了几个配置项： 

- .newBuilder() 状态 TTL 配置的构造器方法，必须调用，返回一个 Builder 之后再调用.build()方法就可以得到 StateTtlConfig 了。方法需要传入一个 Time 作为参数，这就是设定的状态生存时间。 
- .setUpdateType() 设置更新类型。更新类型指定了什么时候更新状态失效时间，这里的 OnCreateAndWrite 表示只有创建状态和更改状态（写操作）时更新失效时间。另一种类型 OnReadAndWrite 则表示无论读写操作都会更新失效时间，也就是只要对状态进行了访问，就表明它是活跃的，从而延长生存时间。这个配置默认为 OnCreateAndWrite。 
- .setStateVisibility() 设置状态的可见性。所谓的“状态可见性”，是指因为清除操作并不是实时的，所以当状态过期之后还有可能基于存在，这时如果对它进行访问，能否正常读取到就是一个问题了。这里设置的 NeverReturnExpired 是默认行为，表示从不返回过期值，也就是只要过期就认为它已经被清除了，应用不能继续读取；这在处理会话或者隐私数据时比较重要。对应的另一种配置是 ReturnExpireDefNotCleanedUp，就是如果过期状态还存在，就返回它的值。 

除此之外，TTL 配置还可以设置在保存检查点（checkpoint）时触发清除操作，或者配置增量的清理（incremental cleanup），还可以针对 RocksDB 状态后端使用压缩过滤器（compaction  filter）进行后台清理。 

这里需要注意，目前的 TTL 设置只支持处理时间。另外，所有集合类型的状态（例如 ListState、MapState）在设置 TTL 时，都是针对每一项（per-entry）元素的。也就是说，一个列表状态中的每一个元素，都会以自己的失效时间来进行清理，而不是整个列表一起清理。
