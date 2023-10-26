---
layout: default
title: 数据湖06：Delta Lake原理和功能概述
parent: DataLake
nav_order: 3.6
---

### 1. DeltaLake是什么

Delta Lake 是 DataBricks 公司开源的、用于构建湖仓架构的存储框架。能够支持 Spark，Flink，Hive，PrestoDB，Trino 等查询/计算引擎。作为一个开放格式的存储层，它在提供了批流一体的同时，为湖仓架构提供可靠的，安全的，高性能的保证。

![](../../assets/images/DataLake/attachments/数据湖06：Delta%20Lake原理和功能概述_image_0.png)

### 2. 关键特性

Delta Lake 关键特性：

1. ACID事务：通过不同等级的隔离策略，Delta Lake 支持多个 pipeline 的并发读写；

1. 数据版本管理：Delta Lake 通过 Snapshot 等来管理、审计数据及元数据的版本，并进而支持 time-travel 的方式查询历史版本数据或回溯到历史版本；

1. 开源文件格式：Delta Lake 通过 parquet 格式来存储数据，以此来实现高性能的压缩等特性；

1. 批流一体：Delta Lake 支持数据的批量和流式读写；

1. 元数据演化：Delta Lake 允许用户合并 schema 或重写 schema，以适应不同时期数据结构的变更；

1. 丰富的DML：Delta Lake 支持 Upsert，Delete 及 Merge 来适应不同场景下用户的使用需求，比如 CDC 场景；

1. 文件结构：湖表较于普通 Hive 表一个很大的不同点在于：湖表的元数据是自管理的，存储于文件系统。

### 3. 文件组织结构

下图为 Delta Lake 表的文件结构：

![](../../assets/images/DataLake/attachments/数据湖06：Delta%20Lake原理和功能概述_image_1.png)

Delta Lake 的文件结构主要有两部分组成：

1. _delta_log目录：存储 deltalake 表的所有元数据信息，其中：

- 每次对表的操作称一次 commit，包括数据操作（Insert/Update/Delete/Merge）和元数据操作（添加新列/修改表配置），每次 commit 都会生成一个新的 json 格式的 log 文件，记录本次 commit 对表产生的行为（action），如新增文件，删除文件，更新后的元数据信息等；

- 默认情况下，每10次 commit 会自动合并成一个 parquet 格式的 checkpoint 文件，用于加速元数据的解析，及支持定期清理历史的元数据文件；

1. 数据目录/文件：除 _delta_log 目录之外的即为实际存储表数据的文件；需要注意：

- DeltaLake 对分区表的数据组织形式同普通的 Hive 表，分区字段及其对应值作为实际数据路径的一部分；

- 并非所有可见的数据文件均为有效的；DeltaLake 是以 snapshot 的形式组织表，最新 snopshot 所对应的有效数据文件在 _delta_log 元数据中管理；

### 4. 元数据机制

Delta Lake 通过 snapshot 来管理表的多个版本，并且支持对历史版本的 Time-Travel 查询。不管是查询当前最新的 snapshot 还是历史某版本的 snapshot 信息，都需要先解析得到对应 snapshot 的元数据信息，主要涉及到：

- 当前 DeltaLake 的读写版本协议（Protocol）；

- 表的字段信息和配置信息（Metadata）;

- 有效的数据文件列表；这一点通过一组新增文件（AddFile）和删除文件（RemoveFile）来描述；

那在加载具体 snopshot 时，为了加速加载流程，先尝试找到小于或等于该版本的 checkpoint 文件，然后结合其后直到当前版本的 log 文件，共同解析得到元数据信息。