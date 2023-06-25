

# Spark篇



- by:  *有梦想的猫小蛋儿*

- 版本:  *v 2021.6*

- *<font color=red>spark版本 : 3.0.0</font>*



## 1 spark 概述

![image-20210603103034344](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603103034344.png)

​	<font color=red>**Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。**</font>

### spark核心模块

![image-20210603105328924](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603105328924.png)

- Spark 是一种由 Scala 语言开发的快速、通用、可扩展的大数据分析引擎
- Spark Core 中提供了 Spark 最基础与最核心的功能，Spark 其他的功能如：Spark SQL，Spark Streaming，GraphX, MLlib 都是在 Spark Core 的基础上进行扩展的
- Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL，户可以使用SQL 或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。
- Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的 API。
- Spark MLlib     MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等
  额外的功能，还提供了一些更底层的机器学习原语。<font color=red>**（略，暂不学习）**</font>
- Spark GraphX  是 Spark 面向图计算提供的框架与算法库。<font color=red>**（略，暂不学习）**</font>

## 1.1 安装部署

运行模式：local、standalone、Yarn 、K8S & Mesos

###  Yarn模式

独立部署（Standalone）模式由 Spark 自身提供计算资源，无需其他框架提供资源。这
种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark 主
要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是
和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的 Yarn 环境
下 Spark 是如何工作的

#### 部署（后边补充）

### 1.2 运行架构

![image-20210603112341556](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603112341556.png)

Spark 框架的核心是一个计算引擎，整体来说，它采用了标准 master-slave 的结构。
如上图所示，它展示了一个 Spark 执行时的基本结构。图形中的 Driver 表示 master，
负责管理整个集群中的作业任务调度。图形中的 Executor 则是 slave，负责实际执行任务。

- 核心组件   Driver和Executor

- **Driver **

  Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作。

  Driver 在 Spark 作业执行时主要负责：

  - 将用户程序转化为作业（job）

  - 在 Executor 之间调度任务(task)

  - 跟踪 Executor 的执行情况

  - 通过 UI 展示查询运行情况

    实际上，我们无法准确地描述 Driver 的定义，因为在整个的编程过程中没有看到任何有关
    Driver 的字眼。所以简单理解，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为
    Driver 类。

- **Executor**

  Spark Executor 是集群中工作节点（Worker）中的一个 JVM 进程，负责在 Spark 作业中运行具体任务（Task），任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点上继续运行。

  Executor 有两个核心功能：

  - 负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程
  - 它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算。

- **Master & Worker**

  Spark 集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master 和 Worker，这里的 Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM, 而Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对数据进行并行的处理和计算，类似于 Yarn 环境中 NM。

- **ApplicationMaster**

  Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含 ApplicationMaster，用于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。说的简单点就是，ResourceManager（资源）和 Driver（计算）之间的解耦合靠的就是ApplicationMaster。

**核心概念**

- **Executor 与 Core**

  Spark Executor 是集群中运行在工作节点（Worker）中的一个 JVM 进程，是整个集群中的专门用于计算的节点。在提交应用中，可以提供参数指定计算节点的个数，以及对应的资源。这里的资源一般指的是工作节点 Executor 的内存大小和使用的虚拟 CPU 核（Core）数量。

  应用程序相关启动参数如下：

  | 名称              | 说明                                   |
  | :---------------- | :------------------------------------- |
  | --num-executors   | 配置 Executor 的数量                   |
  | --executor-memory | 配置每个 Executor 的内存大小           |
  | --executor-cores  | 配置每个 Executor 的虚拟 CPU core 数量 |

- **并行度（Parallelism）**

  在分布式计算框架中一般都是多个任务同时执行，由于任务分布在不同的计算节点进行计算，所以能够真正地实现多任务并行执行，记住，这里是并行，而不是并发。这里我们将整个集群并行执行任务的数量称之为并行度。那么一个作业到底并行度是多少呢？这个取决于框架的默认配置。应用程序也可以在运行过程中动态修改。

- **有向无环图（DAG）**

  ![image-20210603114516269](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603114516269.png)

## 2 Spark Core

Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集
- 累加器：分布式共享只写变量
- 广播变量：分布式共享只读变量

### 2.1 RDD

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

- 弹性
  - 存储的弹性：内存与磁盘的自动切换
  - 容错的弹性：数据丢失可以自动恢复
  - 计算的弹性：计算出错重试机制
  - 分片的弹性：可根据需要重新分片
- 分布式：数据存储在大数据集群不同节点上
- 数据集：RDD 封装了计算逻辑，并不保存数据
- 数据抽象：RDD 是一个抽象类，需要子类具体实现
- 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在新的 RDD 里面封装计算逻辑
- 可分区、并行计算

#### 基础编程

##### RDD 创建

在 Spark 中创建 RDD 的创建方式可以分为四种：

- 从集合（内存）中创建 RDD

  从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

  ```scala
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark")
  val sparkContext = new SparkContext(sparkConf)
  val rdd1 = sparkContext.parallelize(List(1,2,3,4))
  val rdd2 = sparkContext.makeRDD( List(1,2,3,4))
  rdd1.collect().foreach(println)
  rdd2.collect().foreach(println)
  sparkContext.stop()
  ```

  从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法.

- 从外部存储（文件）创建 RDD

  由外部存储系统的数据集创建 RDD 包括：本地的文件系统，所有 Hadoop 支持的数据集，
  比如 HDFS、HBase 等。

  ```scala
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark")
  val sparkContext = new SparkContext(sparkConf)
  val fileRDD: RDD[String] = sparkContext.textFile("input")
  fileRDD.collect().foreach(println)
  sparkContext.stop()
  ```

##### RDD 转换算子

##### RDD 行动算子

##### RDD 依赖关系

- RDD 血缘关系
- RDD 依赖关系
  - 窄依赖
  - 宽依赖

##### RDD 阶段划分

##### RDD 任务划分

RDD 任务切分中间分为：Application、Job、Stage 和 Task

- Application：初始化一个 SparkContext 即生成一个 Application；
- Job：一个 Action 算子就会生成一个 Job；
- Stage：Stage 等于宽依赖(ShuffleDependency)的个数加 1；
- Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。

<font color = red>注意：Application->Job->Stage->Task 每一层都是 1 对 n 的关系。</font>

##### RDD 持久化

- **RDD Cache 缓存**

  RDD 通过 Cache 或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存
  在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算
  子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。

  存储级别

  ```scala
  object StorageLevel {
   val NONE = new StorageLevel(false, false, false, false)
   val DISK_ONLY = new StorageLevel(true, false, false, false)
   val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
   val MEMORY_ONLY = new StorageLevel(false, true, false, true)
   val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
   val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
   val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
   val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
   val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
   val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
   val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
   val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
  
  ```

  ![image-20210603150815199](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603150815199.png)

缓存有可能丢失，或者存储于内存的数据由于内存不足而被删除，RDD 的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于 RDD 的一系列转换，丢失的数据会被重算，由于 RDD 的各个 Partition 是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部 Partition。Spark 会自动对一些 Shuffle 操作的中间数据做持久化操作(比如：reduceByKey)。这样做的目的是为了当一个节点 Shuffle 失败了避免重新计算整个输入。但是，在实际使用的时候，如果想重用数据，仍然建议调用 persist 或 cache。

- **RDD CheckPoint 检查点**

  所谓的检查点其实就是通过将 RDD 中间结果写入磁盘由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发。

- **缓存和检查点区别**

  - Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。
  - Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存储在 HDFS 等容错、高可用的文件系统，可靠性高。
  - 建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存中读取数据即可，否则需要再从头计算一次 RDD。

##### RDD 分区器

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认分区。分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了 Reduce 的个数。

只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None

每个 RDD 的分区 ID 范围：0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。

- **Hash 分区**

  对于给定的 key，计算其 hashCode,并除以分区个数取余

  ```scala
  class HashPartitioner(partitions: Int) extends Partitioner {
   require(partitions >= 0, s"Number of partitions ($partitions) cannot be 
  negative.")
   def numPartitions: Int = partitions
   def getPartition(key: Any): Int = key match {
   case null => 0
   case _ => Utils.nonNegativeMod(key.hashCode, numPartitions)
   }
   override def equals(other: Any): Boolean = other match {
   case h: HashPartitioner =>
   h.numPartitions == numPartitions
   case _ =>
   false
   }
   override def hashCode: Int = numPartitions
  }
  
  ```

- **Range 分区**

  ```scala
  class RangePartitioner[K : Ordering : ClassTag, V](
   partitions: Int,
   rdd: RDD[_ <: Product2[K, V]],
   private var ascending: Boolean = true)
   extends Partitioner {
   // We allow partitions = 0, which happens when sorting an empty RDD under the 
  default settings.
   require(partitions >= 0, s"Number of partitions cannot be negative but found 
  $partitions.")
   private var ordering = implicitly[Ordering[K]]
   // An array of upper bounds for the first (partitions - 1) partitions
   private var rangeBounds: Array[K] = {
   ...
   }
   def numPartitions: Int = rangeBounds.length + 1
   private var binarySearch: ((Array[K], K) => Int) = 
  CollectionsUtils.makeBinarySearch[K]
   def getPartition(key: Any): Int = {
   val k = key.asInstanceOf[K]
   var partition = 0
   if (rangeBounds.length <= 128) {
   // If we have less than 128 partitions naive search
   while (partition < rangeBounds.length && ordering.gt(k, 
  rangeBounds(partition))) {
   partition += 1
   }
   } else {
   // Determine which binary search method to use only once.
   partition = binarySearch(rangeBounds, k)
   // binarySearch either returns the match location or -[insertion point]-1
   if (partition < 0) {
   partition = -partition-1
   }
   if (partition > rangeBounds.length) {
   partition = rangeBounds.length
   }
   }
   if (ascending) {
   partition
   } else {
   rangeBounds.length - partition
   }
   }
   override def equals(other: Any): Boolean = other match {
   ...
   }
   override def hashCode(): Int = {
   ...
   }
   @throws(classOf[IOException])
   private def writeObject(out: ObjectOutputStream): Unit = 
  Utils.tryOrIOException {
   ...
   }
   @throws(classOf[IOException])
   private def readObject(in: ObjectInputStream): Unit = Utils.tryOrIOException 
  {
   ...
   }
  }    
  ```

##### RDD 文件读取与保存

Spark 的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。
文件格式分为：text 文件、csv 文件、sequence 文件以及 Object 文件；
文件系统分为：本地文件系统、HDFS、HBASE 以及数据库。

- **text 文件**

  ```scala
  // 读取输入文件
  val inputRDD: RDD[String] = sc.textFile("input/1.txt")
  // 保存数据
  inputRDD.saveAsTextFile("output")
  ```

- **sequence 文件**

  SequenceFile 文件是 Hadoop 用来存储二进制形式的 key-value 对而设计的一种平面文件(Flat File)。在 SparkContext 中，可以调用 sequenceFile[keyClass, valueClass](path)。
  
  ```scala
  // 保存数据为 SequenceFile
  dataRDD.saveAsSequenceFile("output")
  // 读取 SequenceFile 文件
  sc.sequenceFile[Int,Int]("output").collect().foreach(println)
  ```
  
- **object 对象文件**

  对象文件是将对象序列化后保存的文件，采用 Java 的序列化机制。可以通过 objectFile[T: ClassTag](path)函数接收一个路径，读取对象文件，返回对应的 RDD，也可以通过调用saveAsObjectFile()实现对对象文件的输出。因为是序列化所以要指定类型。

  ```scala
  // 保存数据
  dataRDD.saveAsObjectFile("output")
  // 读取数据
  sc.objectFile[Int]("output").collect().foreach(println)
  ```

### 2.2 累加器

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行 merge。

##### 系统累加器

```scala
val rdd = sc.makeRDD(List(1,2,3,4,5))
// 声明累加器
var sum = sc.longAccumulator("sum");
rdd.foreach(
 num => {
 // 使用累加器
 sum.add(num)
 }
)
// 获取累加器的值
println("sum = " + sum.value)

```

##### 自定义累加器

### 2.3 广播变量

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送。

```scala
val rdd1 = sc.makeRDD(List( ("a",1), ("b", 2), ("c", 3), ("d", 4) ),4)
val list = List( ("a",4), ("b", 5), ("c", 6), ("d", 7) )
// 声明广播变量
val broadcast: Broadcast[List[(String, Int)]] = sc.broadcast(list)
val resultRDD: RDD[(String, (Int, Int))] = rdd1.map {
 case (key, num) => {
 var num2 = 0
 // 使用广播变量
 for ((k, v) <- broadcast.value) {
 if (k == key) {
 num2 = v
 }
 }
 (key, (num, num2))
 }
}
```



## 3 Spark SQL

Spark SQL 是 Spark 用于结构化数据(structured data)处理的 Spark 模块。

**SparkSQL 特点**

- **易整合**

  无缝的整合了 SQL 查询和 Spark 编程

  ![image-20210603160217382](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603160217382.png)

- **统一的数据访问**

  使用相同的方式连接不同的数据源

  ![image-20210603160146885](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603160146885.png)

- **兼容 Hive**

  在已有的仓库上直接运行 SQL 或者 HiveQL 	

  ![image-20210603160445272](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603160445272.png)

- **标准数据连接**

  通过 JDBC 或者 ODBC 来连接

  ![image-20210603160512163](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603160512163.png)

- **DataFrame**

在 Spark 中，DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库中的二维表格。DataFrame 与 RDD 的主要区别在于，前者带有 schema 元信息，即 DataFrame所表示的二维表数据集的每一列都带有名称和类型。这使得 Spark SQL 得以洞察更多的结构信息，从而对藏于 DataFrame 背后的数据源以及作用于 DataFrame 之上的变换进行了针对性的优化，最终达到大幅提升运行时效率的目标。反观 RDD，由于无从得知所存数据元素的具体内部结构，Spark Core 只能在 stage 层面进行简单、通用的流水线优化。同时，与 Hive 类似，DataFrame 也支持嵌套数据类型（struct、array 和 map）。从 API 易用性的角度上看，DataFrame API 提供的是一套高层的关系操作，比函数式的 RDD API 要更加友好，门槛更低。

![image-20210603161217059](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603161217059.png)

上图直观地体现了 DataFrame 和 RDD 的区别.

左侧的 RDD[Person]虽然以 Person 为类型参数，但 Spark 框架本身不了解 Person 类的内部结构。而右侧的 DataFrame 却提供了详细的结构信息，使得 Spark SQL 可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。

DataFrame 是为数据提供了 Schema 的视图。可以把它当做数据库中的一张表来对待DataFrame 也是懒执行的，但性能上比 RDD 要高，主要原因：优化的执行计划，即查询计划通过 Spark catalyst optimiser 进行优化。

- **DataSet **

DataSet 是分布式数据集合。DataSet 是 Spark 1.6 中添加的一个新抽象，是 DataFrame的一个扩展。它提供了 RDD 的优势（强类型，使用强大的 lambda 函数的能力）以及 Spark SQL 优化执行引擎的优点。DataSet 也可以使用功能性的转换（操作 map，flatMap，filter等等）。

- DataSet 是 DataFrame API 的一个扩展，是 SparkSQL 最新的数据抽象
- 用户友好的 API 风格，既具有类型安全检查也具有 DataFrame 的查询优化特性
- 用样例类来对 DataSet 中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet 中的字段名称
- DataSet 是强类型的。比如可以有 DataSet[Car]，DataSet[Person]
- DataFrame 是 DataSet 的特列，DataFrame=DataSet[Row] ，所以可以通过 as 方法将DataFrame 转换为 DataSet。Row 是一个类型，跟 Car、Person 这些的类型一样，所有的表结构信息都用 Row 来表示。获取数据时需要指定顺序

####  核心编程

Spark Core 中，如果想要执行应用程序，需要首先构建上下文环境对象 SparkContext，Spark SQL 其实可以理解为对 Spark Core 的一种封装，不仅仅在模型上进行了封装，上下文环境对象也进行了封装。

SparkSession 是 Spark 最新的 SQL 查询起始点，内部封装了 SparkContext，所以计算实际上是由sparkContext 完成的。当我们使用 spark-shell 的时候, spark 框架会自动的创建一个名称叫做 spark 的 SparkSession 对象, 就像我们以前可以自动获取到一个 sc 来表示 SparkContext 对象一样。

##### DataFrame

###### 创建 DataFrame

在 Spark SQL 中 SparkSession 是创建 DataFrame 和执行 SQL 的入口，创建 DataFrame有三种方式：通过 Spark 的数据源进行创建；从一个存在的 RDD 进行转换；还可以从 Hive Table 进行查询返回。

- **从 Spark 数据源进行创建**

  - 查看 Spark 支持创建文件的数据源格式

    ```scala
    scala> spark.read.
    csv format jdbc json load option options orc parquet schema 
    table text textFile
    ```

  - 在 spark 的 bin/data 目录中创建 user.json 文件

    ```scala
    {"username":"zhangsan","age":20}
    ```

  - 读取 json 文件创建 DataFrame

    ```scala
    val df = spark.read.jsson("data/user.json")
    df: org.apache.spark.sql.DataFrame = [age: bigint， username: string]
    
    ```

    <font color = red>注意：如果从内存中获取数据，spark 可以知道数据类型具体是什么。如果是数字，默认作
    为 Int 处理；但是从文件中读取的数字，不能确定是什么类型，所以用 bigint 接收，可以和
    Long 类型转换，但是和 Int 不能进行转换</font>

  - 展示结果

    ```sql
    +---+--------+
    |age|username|
    +---+--------+
    | 20|zhangsan|
    +---+--------+
    ```

- **RDD 转换为 DataFrame**

  实际开发中，一般通过样例类将RDD转换为DataFrame

  先导入隐式转换包，通过rdd.toDF()方法转换

  在 IDEA 中开发程序时，如果需要 RDD 与 DF 或者 DS 之间互相操作，那么需要引入<font color = red>import spark.implicits._</font>

  这里的 spark 不是 Scala 中的包名，而是创建的 sparkSession 对象的变量名称，所以必须先创建 SparkSession 对象再导入。这里的 spark 对象不能使用 var 声明，因为 Scala 只支持val 修饰的对象的引入。

  - DataFrame 转换为 RDD
    DataFrame 其实就是对 RDD 的封装，所以可以直接获取内部的 RDD

- **从 Hive Table 进行查询返回**

##### DataSet

DataSet 是具有强类型的数据集合，需要提供对应的类型信息。

###### 创建 DataSet

- RDD 转换为 DataSet

  SparkSQL 能够自动将包含有 case 类的 RDD 转换成 DataSet，case 类定义了 table 的结构，case 类属性通过反射变成了表的列名。Case 类可以包含诸如 Seq 或者 Array 等复杂的结构。

```scala
 /**
  * Person样例类
  */
 case class Person(name: String, age: Int)
 
 
  /**
   * 通过RDD创建DataFrame
   */
  @Test
  def creatDFByRDD {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("MyApp")
    val session: SparkSession = SparkSession.builder().config(conf).getOrCreate()
    //根据样例类创建RDD
    val rdd: RDD[(String, Int)] = session.sparkContext.makeRDD(List(("zhangsan", 12), ("lisi", 45), ("wangwu", 23)))
    val person_RDD: RDD[Person] = rdd.map {
      case (name, age) => Person(name, age)
    }
    //导入隐式包，session是上文创建的SparkSession对象
    import session.implicits._
    val df: Dataset[Person] = person_RDD.toDS()
    //查看DF
    df.show()
    session.stop()
  }
```

-  使用基本类型的序列创建 DataSet

  ```scala
  val list: Seq[Int] = List(1, 2, 3, 4)
  import session.implicits._
  val df1 = list.toDS()
  ```

  <font color = red>注意：在实际使用的时候，很少用到把序列转换成DataSet，更多的是通过RDD来得到DataSet</font>

  - DataSet 转换为 RDD
    DataSet 其实也是对 RDD 的封装，所以可以直接获取内部的 RDD

##### DataFrame 和 DataSet 转换

##### RDD、DataFrame、DataSet 三者的关系

###### 三者的共性

- RDD、DataFrame、DataSet 全都是 spark 平台下的分布式弹性数据集，为处理超大型数据提供便利
- 三者都有惰性机制，在进行创建、转换，如 map 方法时，不会立即执行，只有在遇到Action 如 foreach 时，三者才会开始遍历运算;
- 三者有许多共同的函数，如 filter，排序等;
- 在对 DataFrame 和 Dataset 进行操作许多操作都需要这个包:import spark.implicits._（在创建好 SparkSession 对象后尽量直接导入）
- 三者都会根据 Spark 的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出
- 三者都有 partition 的概念
- DataFrame 和 DataSet 均可使用模式匹配获取各个字段的值和类型

###### 三者的区别

- RDD

  - RDD 一般和 spark mllib 同时使用
  - RDD 不支持 sparksql 操作

- DataFrame

  - 与 RDD 和 Dataset 不同，DataFrame 每一行的类型固定为 Row，每一列的值没法直接访问，只有通过解析才能获取各个字段的值
  - DataFrame 与 DataSet 一般不与 spark mllib 同时使用
  - DataFrame 与 DataSet 均支持 SparkSQL 的操作，比如 select，groupby 之类，还能注册临时表/视窗，进行 sql 语句操作
  - DataFrame 与 DataSet 支持一些特别方便的保存方式，比如保存成 csv，可以带上表头，这样每一列的字段名一目了然(后面专门讲解)

- DataSet

  - Dataset 和 DataFrame 拥有完全相同的成员函数，区别只是每一行的数据类型不同。DataFrame 其实就是 DataSet 的一个特例 type DataFrame = Dataset[Row]
  - DataFrame 也可以叫 Dataset[Row],每一行的类型是 Row，不解析，每一行究竟有哪些字段，各个字段又是什么类型都无从得知，只能用上面提到的 getAS 方法或者共性中的第七条提到的模式匹配拿出特定字段。而 Dataset 中，每一行是什么类型是不一定的，在自定义了 case class 之后可以很自由的获得每一行的信息

  ![image-20210603174731857](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210603174731857.png)

## 4 Spark Streaming

#### 概述

![image-20210604175107762](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604175107762.png)

**Spark 流使得构建可扩展的容错流应用程序变得更加容易**

Spark Streaming 用于流式数据的处理。Spark Streaming 支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。

![image-20210604175330375](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604175330375.png)

和 Spark 基于 RDD 的概念很相似，Spark Streaming 使用离散化流(discretized stream)作为抽象表示，叫作 DStream。DStream 是随时间推移而收到的数据的序列。在内部，每个时间区间收到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列(因此得名“离散化”)。所以简单来将，DStream 就是对 RDD 在实时数据处理场景的一种封装。

**Spark Streaming 的特点**

- **易用**

![image-20210604175525889](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604175525889.png)

- **容错**

  ![image-20210604175620402](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604175620402.png)

- **易整合到 Spark 体系**

  ![image-20210604175654553](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604175654553.png)

**Spark Streaming 架构**

![image-20210604180203164](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20210604180203164.png)

#### 数据源

##### kafka

DirectAPI：是由计算的 Executor 来主动消费 Kafka 的数据，速度由自身控制.



##### 优雅关闭

流式任务需要 7*24 小时执行，但是有时涉及到升级代码需要主动停止程序，但是分
布式程序，没办法做到一个个进程去杀死，所有配置优雅的关闭就显得至关重要了。
一般有三种思路：

- **人工手动停止**

- **获取第三方系统做消息通知** (https://www.it610.com/article/1283415083795365888.htm)

  定时的获取第三方系统（hdfs,reids、zk、hbase、db等，我这里用的是 hdfs）的标识，如果需要停止，调用StreamContext对象stop（true,true）方法，自己优雅的终止自己.唯一的问题就是依赖了外部的一个存储系统来达到消息通知的目的。

  MonitorStop

  ```scala
  import java.net.URI
  import org.apache.hadoop.conf.Configuration
  import org.apache.hadoop.fs.{FileSystem, Path}
  import org.apache.spark.streaming.{StreamingContext, StreamingContextState}
  class MonitorStop(ssc: StreamingContext) extends Runnable {
   override def run(): Unit = {
   val fs: FileSystem = FileSystem.get(new URI("hdfs://linux1:9000"), new 
  Configuration(), "atguigu")
   while (true) {
   try
   Thread.sleep(5000)
   catch {
   case e: InterruptedException =>
   e.printStackTrace()
   }
   val state: StreamingContextState = ssc.getState
   val bool: Boolean = fs.exists(new Path("hdfs://linux1:9000/stopSpark"))
   if (bool) {
   if (state == StreamingContextState.ACTIVE) {
   ssc.stop(stopSparkContext = true, stopGracefully = true)
   System.exit(0)
   }
   }
   }
   }
  }
  ```
  
  SparkTest
  
  ```scala
  import org.apache.spark.SparkConf
  import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
  import org.apache.spark.streaming.{Seconds, StreamingContext}
  object SparkTest {
   def createSSC(): _root_.org.apache.spark.streaming.StreamingContext = {
   val update: (Seq[Int], Option[Int]) => Some[Int] = (values: Seq[Int], status: 
  Option[Int]) => {
   //当前批次内容的计算
   val sum: Int = values.sum
   //取出状态信息中上一次状态
   val lastStatu: Int = status.getOrElse(0)
   Some(sum + lastStatu)
   }
   val sparkConf: SparkConf = new 
  SparkConf().setMaster("local[4]").setAppName("SparkTest")
   //设置优雅的关闭
   sparkConf.set("spark.streaming.stopGracefullyOnShutdown", "true")
   val ssc = new StreamingContext(sparkConf, Seconds(5))
   ssc.checkpoint("./ck")
   val line: ReceiverInputDStream[String] = ssc.socketTextStream("linux1", 9999)
   val word: DStream[String] = line.flatMap(_.split(" "))
   val wordAndOne: DStream[(String, Int)] = word.map((_, 1))
   val wordAndCount: DStream[(String, Int)] = wordAndOne.updateStateByKey(update)
   wordAndCount.print()
   ssc
   }
   def main(args: Array[String]): Unit = {
   val ssc: StreamingContext = StreamingContext.getActiveOrCreate("./ck", () => 
  createSSC())
   new Thread(new MonitorStop(ssc)).start()
   ssc.start()
   ssc.awaitTermination()
   }
  }
  ```
  
  
  
- **内部暴露一个socket或者http端口用来接收请求，等待除法关闭流程序**

## 5 Spark调优、性能优化

### 5.1 Explain 查看执行计划

```scala
.explain(mode="xxx")
```

从 3.0 开始，explain 方法有一个新的参数 mode，该参数可以指定执行计划展示格式：
explain(mode="simple")：只展示物理执行计划。
explain(mode="extended")：展示物理执行计划和逻辑执行计划。
explain(mode="codegen") ：展示要 Codegen 生成的可执行 Java 代码。
explain(mode="cost")：展示优化后的逻辑执行计划以及相关的统计。
explain(mode="formatted")：以分隔的方式输出，它会输出更易读的物理执行计划，并展示每个节点的详细信息。

核心的执行过程一共有 5 个步骤：

![image-20211125113913355](https://gitee.com/zhengqianhua0314/image-store/raw/master/image-20211125113913355.png)

### 5.2 资源调优

