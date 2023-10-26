---
layout: default
title: 数据湖07：Apache Hudi原理和功能概述
parent: DataLake
nav_order: 3.7
---

Hudi是Uber公司开源的数据湖架构，数据湖架构是近些年出现的一种新的技术架构，主要是解决目前大数据中Hive储存的一些痛点。HUDI的名字来自四个英文单词的缩写（Hadoop Upsert Delete and Incremental），顾名思义HUDI就是为大数据增加了修改、删除的特性。

当前大数据生态中数据大多存储在Hive中，但是Hive的数据是基于分区存储的，也就最小的单位是分区，如果数据改变数据的话我们不得不对分区进行重建。针对Hive的这些问题，在Hive3.0中也对架构进行了升级和改进，支持了ACID、行级的更新和删除等功能。然而随着各种计算引擎、储存引擎的发展，为了适应不同的业务应用场景，大数据架构设计都偏向于对计算和存储进行解耦，存储结构不依赖计算引擎，然而Hive3.0中的改进并不支持Spark、Flink等引擎。

### 1. Hudi是什么？

Hudi 有时被定位为“表格格式” 或“事务层”。虽然这没有错，但并不能完全体现 Hudi 所提供的一切。

设计者将 Apache Hudi 描述为围绕数据库内核构建的流式数据湖平台（Streaming Data Lake Platform）。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_0.png)

上图从下到上，由左向右看

- hudi 底层的数据可以存储到hdfs、s3、azure、alluxio 等存储

- hudi 可以使用spark/flink 计算引擎来消费 kafka、pulsar 等消息队列的数据，而这些数据可能来源于 app 或者微服务的业务数据、日志数据，也可以是 mysql 等数据库的 binlog 日志数据

- spark/hudi 首先将这些数据处理为 hudi 格式的 row tables （原始表），然后这张原始表可以被 Incremental ETL (增量处理)生成一张 hudi 格式的 derived tables 派生表

- hudi 支持的查询引擎有：trino、hive、impala、spark、presto 等

- 支持 spark、flink、map-reduce 等计算引擎继续对 hudi 的数据进行再次加工处理

Hudi作为一个数据湖方案，他自己本身不产生任何业务数据，可以通过Spark、Flink等工具，接入关系型数据库、日志、消息队列的数据，进行大容量的存储，最终提供给上层查询引擎比如Presto、Hive等进行查询。

相比于传统计算存储架构，HUDI提供了更细粒度的数据处理方式：

- 效率的提升：只更新被修改、删除的数据，而不是更新整个表分区甚至整张表，通过这样的操作，效率提升了一个量级。

- 索引：upsert支持可插拔索引

- ACID语义：增加了ACID语义支持，出现错误可以回滚数据

- 快照：支持读写快照的隔离，读写不相互影响

- 数据恢复：支持保存点数据恢复

- 小文件处理：为行列存储异步合并小文件

- 元数据：支持数据结构变更，并且支持变更的历史跟踪

### 2. Hudi的基础架构

1. 通过DeltaStreammer、Flink、Spark等工具，将数据摄取到数据湖存储，可使用HDFS作为数据湖的数据存储；

1. 基于HDFS可以构建Hudi的数据湖；

1. Hudi提供统一的访问Spark数据源和Flink数据源；

1. 外部通过不同引擎，如：Spark、Flink、Presto、Hive、Impala、Aliyun DLA、AWS Redshit访问接口；

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_1.png)

核心一：Time 有三个数据处理的时间，用于解决因为延迟造成的数据时序问题。

1. 到达的时间

1. 提交的时间

1. 记录的时间

核心二：Timeline，来解决因为延迟造成的数据时序问题。

### 3. Hudi的表格式

Hudi提供两类型表：写时复制（Copy on Write，COW）表和读时合并（Merge On Read，MOR）表。

- 对于 Copy-On-Write Table，用户的 update 会重写数据所在的文件，所以是一个写放大很高，但是读放大为 0，适合写少读多的场景。

- 对于 Merge-On-Read Table，整体的结构有点像 LSM-Tree，用户的写入先写入到 delta data 中，这部分数据使用行存，这部分 delta data 可以手动 merge 到存量文件中，整理为 parquet 的列存结构。

#### 3.1 Copy on Write

简称COW，顾名思义，它是在数据写入的时候，复制一份原来的拷贝，在其基础上添加新数据。        

正在读数据的请求，读取的是最近的完整副本，这类似Mysql 的MVCC的思想。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_2.png)

- 优点：读取时，只读取对应分区的一个数据文件即可，较为高效；

- 缺点：数据写入的时候，需要复制一个先前的副本再在其基础上生成新的数据文件，这个过程比较耗时。 

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_3.png)

COW表主要使用列式文件格式（Parquet）存储数据，在写入数据过程中，执行同步合并，更新数据版本并重写数据文件，类似RDBMS中的B-Tree更新。

更新update：在更新记录时，Hudi会先找到包含更新数据的文件，然后再使用更新值（最新的数据）重写该文件，包含其他记录的文件保持不变。当突然有大量写操作时会导致重写大量文件，从而导致极大的I/O开销。

读取read：在读取数据时，通过读取最新的数据文件来获取最新的更新，此存储类型适用于少量写入和大量读取的场景

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_4.png)

#### 3.2 Merge On Read

简称MOR，新插入的数据存储在delta log 中，定期再将delta log合并进行parquet数据文件。        

读取数据时，会将delta log跟老的数据文件做merge，得到完整的数据返回。下图演示了MOR的两种数据读写方式。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_5.png)

- 优点：由于写入数据先写delta log，且delta log较小，所以写入成本较低；

- 缺点：需要定期合并整理compact，否则碎片文件较多。读取性能较差，因为需要将delta log和老数据文件合并

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_6.png)

MOR表是COW表的升级版，它使用列式（parquet）与行式（avro）文件混合的方式存储数据。在更新记录时，类似NoSQL中的LSM-Tree更新。

- 更新：在更新记录时，仅更新到增量文件（Avro）中，然后进行异步（或同步）的compaction，最后创建列式文件（parquet）的新版本。此存储类型适合频繁写的工作负载，因为新记录是以追加的模式写入增量文件中。

- 读取：在读取数据集时，需要先将增量文件与旧文件进行合并，然后生成列式文件成功后，再进行查询。

#### 3.3 COW vs MOR

对于写时复制（COW）和读时合并（MOR）writer来说，Hudi的WriteClient是相同的。

- COW表，用户在 snapshot 读取的时候会扫描所有最新的 FileSlice 下的 base file。

- MOR表，在 READ OPTIMIZED 模式下，只会读最近的经过 compaction 的 commit。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_7.png)

### 4. 数据组织结构

#### 4.1 表数据结果

Hudi表的数据文件，可以使用操作系统的文件系统存储，也可以使用HDFS这种分布式的文件系统存储。为了后续分析性能和数据的可靠性，一般使用HDFS进行存储。以HDFS存储来看，一个Hudi表的存储文件分为两类。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_8.png)

.hoodie 文件：由于CRUD的零散性，每一次的操作都会生成一个文件，这些小文件越来越多后，会严重影响HDFS的性能，Hudi设计了一套文件合并机制。 .hoodie文件夹中存放了对应的文件合并操作相关的日志文件。

amricas和asia相关的路径是实际的数据文件，按分区存储，分区的路径key是可以指定的。

#### 4.2 .hoodie文件

Hudi把随着时间流逝，对表的一系列CRUD操作叫做Timeline，Timeline中某一次的操作，叫做Instant。

- Instant Action，记录本次操作是一次数据提交（COMMITS），还是文件合并（COMPACTION），或者是文件清理（CLEANS）；

- Instant Time，本次操作发生的时间；

- State，操作的状态，发起(REQUESTED)，进行中(INFLIGHT)，还是已完成(COMPLETED)；

- .hoodie文件夹中存放对应操作的状态记录：

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_9.png)

#### 4.3 数据文件

Hudi真实的数据文件使用Parquet文件格式存储

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_10.png)

其中包含一个metadata元数据文件和数据文件parquet列式存储。

Hudi为了实现数据的CRUD，需要能够唯一标识一条记录，Hudi将把数据集中的唯一字段(record key ) + 数据所在分区(partitionPath) 联合起来当做数据的唯一键。

#### 4.4 数据存储概述

Hudi数据集的组织目录结构与Hive表示非常相似，一份数据集对应这一个根目录。数据集被打散为多个分区，分区字段以文件夹形式存在，该文件夹包含该分区的所有文件。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_11.png)

在根目录下，每个分区都有唯一的分区路径，每个分区数据存储在多个文件中。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_12.png)

每个文件都有惟一的fileId和生成文件的commit所标识。如果发生更新操作时，多个文件共享相同的fileId，但会有不同的commit。 

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_13.png)

#### 4.5 Metadata 元数据

以时间轴（timeline）的形式将数据集上的各项操作元数据维护起来，以支持数据集的瞬态视图，这部分元数据存储于根目录下的元数据目录。一共有三种类型的元数据：

- Commits：一个单独的commit包含对数据集之上一批数据的一次原子写入操作的相关信息。我们用单调递增的时间戳来标识commits，标定的是一次写入操作的开始。

- Cleans：用于清除数据集中不再被查询所用到的旧版本文件的后台活动。

- Compactions：用于协调Hudi内部的数据结构差异的后台活动。例如，将更新操作由基于行存的日志文件归集到列存数据上

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_14.png)

#### 4.6 Index 索引

Hudi维护着一个索引，以支持在记录key存在情况下，将新记录的key快速映射到对应的fileId。

- Bloom filter：存储于数据文件页脚。默认选项，不依赖外部系统实现。数据和索引始终保持一致。

- Apache HBase ：可高效查找一小批key。在索引标记期间，此选项可能快几秒钟。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_15.png)

#### 4.7 Data 数据

Hudi以两种不同的存储格式存储所有摄取的数据，用户可选择满足下列条件的任意数据格式：

- 读优化的列存格式（ROFormat）：缺省值为Apache Parquet；

- 写优化的行存格式（WOFormat）：缺省值为Apache Avro；

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_16.png)

### 5. 查询

Hudi支持三种不同的查询表的方式：Snapshot Queries(快照查询)、Incremental Queries(增量查询)和Read Optimized Queries(读优化查询).

1. Snapshot Queries(快照查询)

查询某个增量提交操作中数据集的最新快照，先进行动态合并最新的基本文件（parquet）和增量文件（Avro）来提供近实时数据集（通常会存在几分钟的延迟）

读取所有partition下每个FileGroup最新的FileSlice中的文件，Copy On Write表读parquet文件，Merge On Read表读parquet + log文件。

1. Incremental Queries(增量查询)

仅查询新写入数据集的文件，需要指定一个Commit/Compaction的即时时间（位于Timeline上的某个instant）作为条件，来查询此条件之后的新数据

可查看自给定commit/delta commit即时操作依赖新写入的数据，有效地提供变更流来启用增量数据管道。

1. Read Optimized Queries(读优化查询)

直接查询基本文件（数据集的最新快照），其实就是列式文件（Parquet）。并保证与非hudi列式数据集相比，具有相同的列式查询性能。

可查看给定的commit/compact即时操作的表的最新快照；

读优化查询和快照查询相同仅访问基本文件，提供给定文件片自上次执行压缩操作以来的数据。通常查询数据的最新程度的保证取决于压缩策略。

### 6. 计算模型

hudi是Uber主导开发的开源数据湖框架，所以大部分的出发点都来源于Uber自身场景，比如司机数据和乘客数据通过订单ID来做join等。

在hudi过去的使用场景里，和大部分公司的架构类似，采用批式和流式共存的Lambda架构，后来Uber提出增量Incremental模型，相对批式来讲，更加实时，相对流式而言，更加经济。

1. 批式模型(Batch)

批式模型就是使用MapReduce、Hive、Spark等典型的批计算引擎，以小时任务或者天任务的形式来做数据计算。

特性如下：

- A.延迟：小时级延迟或者天级别延迟。这里的延迟不单单指的是定时任务的时间，在数据架构里，这里的延迟时间通常是定时任务间隔时间+一系列依赖任务的计算时间+数据平台最终可以展示结果的时间。数据量大、逻辑复杂的情况下，小时任务计算的数据通常真正延迟的时间是2-3小时。

- B.数据完整度：数据较完整。以处理时间为例，小时级别的任务，通常计算的原始数据已经包含了小时内的所有数据，所以得到的数据相对较完整。但如果业务需求是事件时间，这里涉及到终端的一些延迟上报机制，在这里，批式计算任务就很难派上用场。

- C.成本：成本很低。只有在做任务计算时，才会占用资源，如果不做任务计算，可以将这部分批式计算资源出让给在线业务使用。从另一个角度来说成本是挺高的，如原始数据做了一些增删改查，数据晚到的情况，那么批式任务是要全量重新计算。

1. 流式模型(Stream)

流式模型，典型的就是使用Flink来进行实时的数据计算，特性：

- A.延迟：很短，甚至是实时。

- B.数据完整度：较差。因为流式引擎不会等到所有数据到齐之后再开始计算，所以有一个watermark的概念，当数据的时间小于watermark时，就会被丢弃，这样是无法对数据完整度有一个绝对的保障。在互联网场景中，流式模型主要用于活动时的数据大盘展示，对数据的完整度要求并不算很高。在大部分场景中，用户需要开发两个程序，一是流式数据生产流式结果，而是批式计算人物，用于次日修复实时结果。

- C.成本：很高。因为流式任务时常驻的，并且对于多流join的场景，通常要借助内存或者数据库来做state的存储，不管是序列化开销，还是和外部组件交互产生的额外IO，在大数据量下都是不容忽视的。

1. 增量模型(Incremental)

针对批式和流式的优缺点，Uber提出了增量模型(Incremental Mode)，相对批式来讲，更加实时；相对流式而言，更加经济。

增量模型，简单来讲，就是一mini batch的形式来跑准实时任务。hudi在增量模型中支持了两个最重要的特性：

- A.Upsert：这个主要是解决批式模型中，数据不能插入、更新的问题，有了这个特性，可以往Hive中写入增量数据，而不是每次进行完全的覆盖。（hudi自身维护了key-file的映射，所以当upsert时很容易找到key对应的文件）

- B.Incremental Query：增量查询，减少计算的原始数据量。以uber中司机和乘客的数据流join为例，每次抓取两条数据流中的增量数据进行批式的join即可，相比流式数据而言，成本要降低几个数量级。

### 7. 应用

国内很多大公司，都在使用Hudi，构建数据湖，并且与大数据仓库整合，搭建湖仓一体化平台。

![](../../assets/images/DataLake/attachments/数据湖07：Apache%20Hudi原理和功能概述_image_17.png)