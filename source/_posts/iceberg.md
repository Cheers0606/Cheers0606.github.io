---
title: iceberg
tags: ["iceberg","数据湖","湖仓一体"]
overdue: false
categories: 数据湖
date: 2023-04-29 15:58:48
updated: 2023-04-29 15:58:48
---

> Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time.
>
> Iceberg是用于大型分析表的高性能格式。Iceberg为大数据带来了SQL表的可靠性和简单性，同时使**Spark**，Trino，Flink，Presto，Hive和Impala等引擎能够同时安全地使用相同的表。
>
> - 格式：数据的组织方式
> - SQL表：可以用SQL的方式进行操作
> - 兼容性：各种计算、分析引擎都可以操作iceberg



## 一、 iceberg简介

### 1.0 从hive说起

> https://www.tabular.io/blog/iceberg-fileio-cloud-native-tables/

在hive当中，hive是当前大数据领域处理数据的通用表格式。在hive中，我们怎么定义一张表呢？
1. 存放在 metastore 中的schema（表的格式定义、属性定义）
2. 保存在文件系统上的指定目录下的数据文件（内表 warehouse目录下，外表在表指定location位置）

![image-20240913212215795](/image/image-20240913212215795.png)

存在的问题：

1. 现在的计算引擎(Presto、Spark)都是分布式执行的，以 Spark 为例，假如某个表有 100 个数据文件，执行时一共有 10 个 Executor，在 Exector 执行前，Driver 会对数据文件进行切分，最终每个 Executor 可能分配 10 个数据文件。由于 Hive 表格式只保存了数据文件的目录，所以在文件切分前会使用文件系统的 list 操作，列出所有的数据文件。

list 操作存在以下问题：

- HDFS 文件系统下，如果频繁大量的调用 list 操作会给 NameNode 的 RPC 带来压力；如果文件数过多，list 会比较耗时，曾经就遇到一个这样的例子，由于一个表使用了二级分区，生成了大量的分区和文件，在执行全表扫描的时候，仅仅是文件切分就花了10 分钟。
- 对象存储下 list 操作非常慢，导致数据访问延时太高





![image-20240913212414709](/image/image-20240913212414709.png)

2. 由于 Hive 表格式只保存了数据文件的目录，所以在 Executor 执行时，先把计算结果写入临时目录，等待 Executor 全部执行完成后，Driver 端会把临时文件目录 rename 到正式的文件目录，此操作依赖文件系统的 rename 操作。在对象存储中 rename 操作非常慢，并且该操作在对象存储上不可靠（不同厂商实现不同， 早期 S3 的 rename 是 copy 操作）

![image-20240913213440162](/image/image-20240913213440162.png)

3. Hive 表的 schema 集中存储在 metastore中，metastore 很容易成为性能瓶颈，同时也会带来分库分表等运维成本。
4. Hive 表的统计信息(文件行数，文件大小，文件个数) 不是强制要求写入的，很多情况不存在统计信息或者是过时的，planner 层无法有效的做基于代价（CBO）的优化。另外统计信息的粒度很粗是表级别的。Hive Metastore 在分区管理上也做的不好，分区依赖文件系统的目录树，导致分区的 evolution 成本昂贵
5. 不支持删除，更新表等操作，或者成本非常高，需要重刷数据
6. 如果同时存在 读-写，写-写 任务时，无法保证任务的一致性，会发生莫名其妙的错误，或刚写入的数据被其它任务覆盖了。

### 1.1 iceberg是什么

![image-20240913215048851](/image/image-20240913215048851.png)

[Apache Iceberg](https://iceberg.apache.org/)是一种用于**大规模分析的开源表格格式**。它改进了传统表存储解决方案的局限性，提供了一种高性能、更高效的大规模数据管理方式。Iceberg允许对数据进行细粒度控制，支持模式演化、时间旅行和事务支持等功能，这些功能对现代数据架构至关重要。

> 存储引擎之上，计算引擎之下的中间层，本质是一种表格格式。
>
> 按照一定的方式把数据组织起来。

Netflix最初（18年）推出Iceberg项目是为了应对处理其海量数据仓库的挑战，并提高现有表格格式的性能和可扩展性。随后，它被开源，并被广泛的公司采用，如Airbnb，Adobe，LinkedIn等等。

​	Iceberg 的诞生初衷就是为了优化 Hive 存在的问题，比如不支持行级更新，不支持 data skipping，不支持分区演进等。但是 Iceberg 对 hive metastore 支持较好，可以将 Hive 在不迁移数据文件的情况下直接转换为 Iceberg 表，成本很低。

### 1.2 为什么选iceberg

Apache Iceberg 提供了一种快速、高效的方式来大规模处理大型数据集。它具有以下优势：

1. 开源：Apache Iceberg 是一个开源项目，遵循   Apache License 2.0版本

2. 可扩展性/兼容性：Apache Iceberg 旨在高效处理大型数据集。向下可以兼容hdfs、s3等数据存储组件，向上可以兼容spark、flink、hive、presto、doris的计算引擎。

3. 性能：Apache Iceberg 具有多种优化查询性能的功能，包括

    - 文件分区：数据可以在多个字段上分区，包括复杂的嵌套数据结构。可以帮助减少查询需要扫描的数据量。

    - Predicate Pushdown：通过支持Predicate Pushdown，Iceberg可以在存储级别过滤数据，而无需查询引擎将数据加载到内存中，从而减少数据传输和处理时间。

    - 增量扫描/流式读取：Iceberg启用增量扫描，只读取自上次查询以来更改的文件部分（CDC）。这有助于通过减少要重新处理的数据来提高重复或频繁查询的性能。

    - 压缩：iceberg自动压缩和调整大小的文件，以减少碎片和不必要的访问。

4. 灵活性：Apache Iceberg 允许用户更改数据的组织方式，使其可以随着时间的推移而发展，无需重写查询或重建数据结构。它还支持多种数据格式和数据来源，因此可与现有系统集成。

5. 可靠性：Apache Iceberg 通过对事务处理的支持来确保数据的一致性和可靠性。用户可以跟踪数据随时间推移而发生的变化，并回滚到历史版本以纠正问题。（时间旅行）

### 1.3 关键特性

#### 1.3.1 演化

Iceberg可以通过SQL的方式进行表级别模式演进。iceberg执行这些操作时，不需要复杂的操作和昂贵的代价，如重写表数据或迁移到一个新表。

例如，Hive表分区无法更改，因此从每日分区布局移动到每小时分区布局需要一个新表。由于查询依赖于分区，因此必须为新表重写查询。在某些情况下，甚至像重命名列这样简单的更改也不受支持，或者可能导致[数据正确性](https://iceberg.apache.org/docs/nightly/evolution/#correctness)问题。

- 模式演化（schema evolution）

  > iceberg的模式演进是通过**更改元数据**实现的，因此所有的操作都不需要重写数据文件。

    - ADD：增加新的列
    - DROP：删除指定列，也可以是删除嵌套结构中的某些列
    - RENAME：重命名列
    - UPDATE：将复杂结构(struct, map<key, value>, list)中的基本类型扩展类型长度, 比如tinyint修改成int（需要本身是直接隐式转换的）
    - REORDER：重新排列字段

  Iceberg使用**唯一的ID**（类似于人的身份证，名字可以随便改，但是身份证一定是唯一不变的）来跟踪表中的每一列。当您添加一个列时，它会被分配一个新的ID，这样现有的数据就不会被错误地使用。使用名称或者位置信息来定位列的, 都会存在一些问题, 比如使用名称的话，名称可能会重复，使用位置的话，不能修改顺序并且废弃的字段也不能删除。

- 分区演化

    - Iceberg可以在一个已存在的表上直接修改，因为Iceberg的查询流程并不和分区信息直接关联。

    - 当我们改变一个表的分区策略时，对应修改分区之前的数据不会改变, 依然会采用老的分区策略，新的数据会采用新的分区策略，也就是说同一个表会有两种分区策略，旧数据采用旧分区策略，新数据采用新新分区策略, 在元数据里两个分区策略相互独立，不重合。

    - 在查询数据的时候，如果存在跨分区策略的情况，则会解析成两个不同执行计划，如Iceberg官网提供图所示：

![Partition evolution diagram](https://iceberg.apache.org/docs/nightly/assets/images/partition-spec-evolution.png)

​	图中booking_table表2008年按月分区，进入2009年后改为按天分区，这两中分区策略共存于该表中。

​	借助Iceberg的隐藏分区（Hidden Partition），在写SQL 查询的时候，不需要在SQL中特别指定分区过滤条件，Iceberg会自动分区，过滤掉不需要的数据。

​	Iceberg分区演化操作同样是一个元数据操作, 不会重写数据文件。

#### 1.3.2 隐藏分区

在传统的Hive分区方法中，分区是通过物理文件夹结构来实现的，每个分区对应一个特定的文件夹。这种方法存在一些问题，例如：

- 修改分区方案需要重新组织整个数据集，这可能非常耗时和资源密集。
- 添加或删除分区列通常需要重新写入数据和重新组织文件夹。

Iceberg的隐藏分区解决了这些问题。它将分区信息存储在元数据文件中，这使得分区管理更加灵活和高效。

假设我们有一个日志表，包含一个时间戳列`event_ts`。我们想根据这个时间戳列进行分区。在Hive中，我们需要创建一个新的分区列，例如`event_date`，并将其作为分区列。在Iceberg中，我们可以使用隐藏分区来实现这一点。我们可以在表定义中指定分区表达式，例如`day(event_ts)`，这告诉Iceberg如何根据时间戳列进行分区。

```sql
CREATE TABLE logs (
  level string,
  event_ts timestamptz,
  message string
) PARTITIONED BY (
  day(event_ts)
);

SELECT level, message FROM logs
WHERE event_ts BETWEEN '2018-12-01 10:00:00' AND '2018-12-01 12:00:00';

## 这种情况下，可以命中分区，不会全表扫描
```

#### 1.3.3 分区转换

Iceberg还提供了一组分区转换函数，可以用于分区表达式。这些函数包括：

- `year`、`month`、`day`、`hour`：根据时间戳列进行分区。
- `bucket(N, any_col)`：将数据分成N个桶。
- `truncate(L, string_col)`：截断字符串列到指定长度。
- `truncate(W, numeric_col)`：截断数字列到指定宽度。

#### 1.3.4 时间旅行

Iceberg的时间旅行是基于快照（snapshot）实现的。每个表都由一系列的快照组成，每个快照代表了表在某个时间点的状态。通过解析表的元数据文件，用户可以获取到当前表的快照ID以及所有的快照信息。

Iceberg的时间旅行通过以下步骤实现：

1. **快照ID**：每个快照都有一个唯一的ID，用于标识该快照的时间点。用户可以通过快照ID来查询表在某个时间点的数据。
2. **快照文件**：每个快照都对应一个快照文件，该文件记录了该快照的所有数据文件信息。通过解析快照文件，用户可以获取到该快照的数据文件列表。
3. **数据文件**：数据文件是实际存储数据的文件。在Iceberg中，数据文件被分为多个分区，每个分区对应一个列。这使得Iceberg可以根据查询条件自动进行分区裁剪，提高查询性能。

以下是使用Iceberg的时间旅行功能的示例：

```sql
SELECT * FROM customer_iceberg FOR VERSION AS OF 5043425904354141100;
```

该查询将返回表在指定快照ID的状态。

```sql
SELECT * FROM customer_iceberg FOR TIMESTAMP AS OF TIMESTAMP '2022-09-18 07:18:09.002 America/New_York';
```

Iceberg的时间旅行功能在数据湖架构中具有广泛的应用场景，包括：

- **数据回滚**：当数据出现问题时，管理员可以通过时间旅行功能回滚到之前的快照，恢复数据到某个稳定的状态。
- **数据审计**：时间旅行功能可以用于审计数据的历史变化，帮助管理员了解数据的变化过程。
- **故障恢复**：时间旅行功能可以用于恢复数据到某个稳定的状态，帮助管理员快速恢复数据

注意：

Iceberg的时间旅行功能允许用户查询表在某个特定时间点的状态，但并不是无限选择时间。

1. **历史数据保留策略**：
   Iceberg 的历史快照不会被永久保存，快照的保留时长通常由数据清理策略（如 `expireSnapshots` 命令）决定。如果某个快照或特定时间点的历史数据被清理掉了，那么就无法回溯到这个时间点或快照。
2. **存储空间**：
   每个快照包含表的元数据和数据文件引用，保留过多的快照将增加存储空间的需求。为了减少存储成本，一般会通过策略定期清理老的快照和数据文件。
3. **性能影响**：
   随着表的快照数量增加，表的元数据文件可能变大，进行时间旅行查询时需要额外的元数据扫描。因此，当表有大量历史快照时，查询历史数据的性能可能会受到影响。
4. **快照生成的间隔**：
   虽然可以回到任意时间点，但能否找到具体时间的快照也取决于实际的快照生成频率。如果在某段时间没有生成快照，无法查询到这个时间点的具体数据，Iceberg 只能回溯到最近的快照。

#### 1.3.5 其他

1. 事务性写入：Iceberg支持原子性、一致性、隔离性和持久性（ACID）的事务，确保数据表的一致性和可靠性
2. 多引擎支持：Iceberg支持多种数据处理引擎，包括Apache Spark、Apache Flink、Trino、Presto、Hive和Impala等
3. 高效的存储管理：最大限度地减少元数据大小并提高读/写性能，尤其是对于大型数据集。

## 二、 iceberg的安装部署

iceberg其实不是一个独立的组件，不需要单独的进行部署。只需要在对应的引擎中增加相应的包（runtime包）就可以了。

具体参考 第四章节：iceberg集成



## 三、iceberg底层原理

```sql
CREATE TABLE hadoop_prod.default.events (event_time timestamp, device_id bigint) USING ICEBERG PARTITIONED BY (days(event_time), bucket(64,device_id));


INSERT INTO hadoop_prod.default.events VALUES (current_timestamp(), 1), (current_timestamp(), 2), (current_timestamp(), 3);

INSERT INTO hadoop_prod.default.events VALUES (current_timestamp(), 1), (current_timestamp(), 2), (current_timestamp(), 3);

select * from hadoop_prod.default.events.snapshots;
```

![image-20240922105358644](/image/image-20240922105358644.png)

在 Insert 执行完成后，最终生成的文件结构，如上图所示，主要可以分为三类文件:

1. 数据文件，普通的 Parquet 文件，存放着写入的数据。
2. 元数据文件，主要是 avro 和 json 类型，这正是 Iceberg 表和 Hive 表的本质区别。
3. catalog（version-hint.text 文件，只有使用 Hadoop catalog 才会存在此文件）

### iceberg的存储结构

![image-20240922105534011](/image/image-20240922105534011.png)



- manifest file （清单文件）

Manifest file是一个元数据文件，它列出组成快照（snapshot）的数据文件（data files）的列表信息。每行都是每个数据文件的详细描述，manifest file 文件核心内容如下图所示，还有一些其它信息，包括数据文件的状态、文件路径、分区信息、列级别的统计信息（比如每列的最大最小值、空值数等）、文件的大小以及文件里面数据行数等信息。其中列级别的统计信息可以在扫描表数据时过滤掉不必要的文件。默认一个 manifest file 文件大小是 8M。

图片来源 https://tabular.io/blog/iceberg-metadata-indexing/

![image-20240922105843676](/image/image-20240922105843676.png)

- manifest list（清单列表）

manifest list是一个元数据文件，它列出构建表快照（Snapshot）的清单（Manifest file）。这个元数据文件中存储的是Manifest file列表，每个Manifest file占据一行。每行中存储了Manifest file的路径、其存储的数据文件（data files）的分区范围，增加了几个数文件、删除了几个数据文件等信息，这些信息可以用来在查询时提供过滤，加快速度。

![image-20240922110444634](/image/image-20240922110444634.png)

- snapshot（表快照）

  快照代表一张表在某个时刻的状态。每个快照里面会列出表在某个时刻的所有 data files 列表。data files是存储在不同的manifest files里面，manifest files是存储在一个Manifest list文件里面，而一个Manifest list文件代表一个快照。

- metadata file（元数据文件）

Iceberg 文件组织结构图中 的 json 文件，每次提交表的时候生成一个，上面一共有 3 个 metadata.json 文件，创建表的时候生成一个，每 insert 一次生成一个。包含的信息主要有：

1. 当前表的 schema 信息和历史的 schema 信息。
2. 表的提交历史（快照）和当前快照ID。快照中包含当前表的一些统计信息，例如文件总大小，文件个数，总行数。
3. 文件路径、分区等其它信息。

![image-20240922111511160](/image/image-20240922111511160.png)

- Catalog （目录）

Iceberg 文件组织结构图中 version-hint.txt 文件，使用 Hadoop Catalog 才会生成此文件。由于表的 schema 和统计信息已经保存在 metadata.josn 文件中， catalog 比较轻量，只需要保存 Iceberg 表最新的 metadata.json 文件的存储路径。

当前常用的catalog为：**hive catalog**和hadoop catalog

## 四、 iceberg集成

集成的基本步骤：

1. 添加相关的runtime包
2. 增加相关配置
3. 重启对应的服务

### 4.1 hive集成

1. 上传相关包到hive的lib路径下
    1. Iceberg-hive-rumtime-xxx.jar
    2. libfb303-0.9.3.jar

2. 添加iceberg配置。（hive-site.xml）

   ```xml
   <property>
       <name>iceberg.engine.hive.enabled</name>
       <value>true</value>
   </property>
   ```

3. 启动hive（如果已经启动，那么需要重启）

   ```bash
   nohup hive --service metastore &
   nohup hive --service hiveserver2 &
   
   beeline -u jdbc:hive2://localhost:10000
   ```



hive使用iceberg的一些操作

> 默认使用的是hive catalog，表会存放到hive的warehouse目录

```sql
-- 开启本地模式，加快一些简单操作的速度
set hive.exec.mode.local.auto=true;

CREATE TABLE iceberg_demo1 (id int) 
STORED BY 'org.apache.iceberg.mr.hive.HiveIcebergStorageHandler';
-- 查看HDFS可以发现，表目录在默认的hive仓库路径下。
-- /user/hive/warehouse/iceberg_demo1
INSERT INTO iceberg_demo1 values(1);

show create table iceberg_demo1;
```

- 如果想用hadoop catalog，也可以进行配置

  ```sql
  -- 配置一个叫做iceberg_hadoop的catalog
  set iceberg.catalog.iceberg_hadoop.type=hadoop;
  set iceberg.catalog.iceberg_hadoop.warehouse=hdfs://hadoop:9000/warehouse/iceberg-hadoop;
  
  CREATE TABLE iceberg_demo3 (id int) 
  STORED BY 'org.apache.iceberg.mr.hive.HiveIcebergStorageHandler' 
  LOCATION 'hdfs://hadoop:9000/warehouse/iceberg-hadoop/default/iceberg_demo3'
  TBLPROPERTIES('iceberg.catalog'='iceberg_hadoop');
  
   
  INSERT INTO iceberg_demo3 values(1);
  
  show create table iceberg_demo3;
  ```

- 加载已有的表（外表的方式）

  ```sql
  CREATE EXTERNAL TABLE iceberg_demo4 (id int)
  STORED BY 'org.apache.iceberg.mr.hive.HiveIcebergStorageHandler'
  LOCATION 'hdfs://hadoop:9000/warehouse/iceberg-hadoop/default/iceberg_demo3'
  TBLPROPERTIES ('iceberg.catalog'='location_based_table');
  
  select * from iceberg_demo4;
  ```

- 分区表

  ```sql
  CREATE EXTERNAL TABLE iceberg_demo5 (id int,name string)
  PARTITIONED BY (age int)
  STORED BY 'org.apache.iceberg.mr.hive.HiveIcebergStorageHandler';
  
  describe formatted iceberg_demo5;
  
  show create table iceberg_demo5;
  ```

  hive的语法创建的分区表，并不会在hive的metastore当中创建分区，而是将分区数据转换为iceberg标识分区default-partition-spec。这种情况下**不能使用**iceberg的分区转换，例如 bucket(10,age)，如果需要做分区转换，只能用spark或者flink引擎等。（推荐使用spark）

### 4.2 spark集成（重点）

> spark和iceberg的集成度最高，很多操作都只能通过spark（sql）来进行

iceberg和不同的引擎进行集成时，要注意对应的版本关系。参考官网说明。一般iceberg版本越高，对spark的版本要求也越高。

> https://iceberg.apache.org/releases/#120-release

1. 安装spark

   `tar -zxvf spark-3.3.4-bin-hadoop2.tgz -C ~/app/`

2. 配置环境变量

   `vim ~/.bash_profile `

   修改SPARK_HOME路径为3.3.4的安装路径，并重新连接ssh使配置生效。

3. 配置spark支持iceberg

   ```properties
   ## hive catalog (常用)
   spark.sql.catalog.hive_prod = org.apache.iceberg.spark.SparkCatalog
   spark.sql.catalog.hive_prod.type = hive
   spark.sql.catalog.hive_prod.uri = thrift://hadoop:9083
   
   ## hadoop catalog
   spark.sql.catalog.hadoop_prod = org.apache.iceberg.spark.SparkCatalog
   spark.sql.catalog.hadoop_prod.type = hadoop
   spark.sql.catalog.hadoop_prod.warehouse = hdfs://hadoop:9000/warehouse/iceberg-hadoop
   
   ```

   ```bash
   ## 如果要用hive catalog的话，需要将hive的 hive-site.xml文件也复制到spark的conf目录下，这样子才会读取hive当中的配置项，如hive.metastore.warehouse.dir
   cp ~/app/hive/conf/hive-site.xml ~/app/spark-3.3.4-bin-hadoop2/conf
   # 同时把hive.metastore.warehouse.dir中的地址写全
     <property>
       <name>hive.metastore.warehouse.dir</name>
       <value>hdfs://hadoop:9000/user/hive/warehouse</value>
       <description>location of default database for the warehouse</description>
     </property>
   ```

4. 上传iceberg-spark-runtime-xxx.jar 包到spark的jars目录下，并复制hive的lib下的MySQL驱动包到spark的jars目录下。

5. 启动spark-sql。



#### 4.2.1 基本操作

```sql
# 其中 hive_prod 是上面配置的catalog名字，对应的是hive catalog
show tables in hive_prod.default;

iceberg_demo1
iceberg_demo3
iceberg_demo4
iceberg_demo5

# 可以看到，我们启动spark sql后，可以直接访问到存放在hive metastore中的表。


use hadoop_prod;  # 不是特别推荐，最好是操作的时候都带上catalog名称
create database db1; 

-- 推荐写法：
create database hadoop_prod.db2; 

create table hadoop_prod.default.sample1 (id bigint COMMENT 'unique id', data string ) using iceberg;

```

使用spark建表的话，用的就是spark sql当中的数据类型。spark和iceberg的数据类型对应关系参考：https://iceberg.apache.org/docs/latest/spark-getting-started/#type-compatibility

使用spark还支持以下语法：

- `PARTITIONED BY (partition-expressions)` 配置分区
- `LOCATION '(fully-qualified-uri)'` 设置表位置
- `COMMENT 'table documentation'`设置表格描述
- `TBLPROPERTIES ('key'='value', ...)` 设置[表配置](https://iceberg.apache.org/docs/latest/configuration/)
    - 对Iceberg表的每次更改都会生成一个新的元数据文件（json文件）以提供原子性。默认情况下，旧元数据文件作为历史文件保存不会删除。
    - 如果要自动清除元数据文件，在表属性中设置write.metadata.delete-after-commit.enabled=true。这将保留一些元数据文件（直到write.metadata.previous-versions-max），并在每个新创建的元数据文件之后删除旧的元数据文件。

注意：`CREATE TABLE ... LIKE ...` 这种语法不支持

#### 4.2.2 分区表

```sql
-- 分区表
CREATE TABLE hadoop_prod.default.sample2 (
    id bigint,
    data string,
    category string)
USING iceberg
PARTITIONED BY (category);

-- 创建隐藏分区表
CREATE TABLE hadoop_prod.default.sample3 (
    id bigint,
    data string,
    category string,
    ts timestamp)
USING iceberg
PARTITIONED BY (bucket(16, id), days(ts), category);
```

支持的转换包括：

- `year（ts）`：按年份划分

- `month(ts)`:按月分区
- `day(ts)` or `date(ts)`: 等同于dateint分区
- `hour(ts)` or `date_hour(ts)`: 等同于dateint和hour分区
- `bucket(N, col)`: 按哈希值mod N bucket进行分区
- `truncate（L，col）`：按截断为L的值进行分区
    - Strings are truncated to the given length
      字符串被截断为给定长度
    - 整数和长整型截断到bin：`truncate（10，i）`产生分区0，10，20，30，.

注：为了兼容性，也支持`years(ts)`, `months(ts)`, `days(ts)` and `hours(ts)` 的旧语法。

#### 4.2.3 CTAS

```sql
CREATE TABLE hadoop_prod.default.sample4
USING iceberg
AS SELECT * from hadoop_prod.default.sample3;
-- 新创建的表不会从SELECT中的源表继承分区规范和表属性，可以在CTAS中使用PARTITIONED BY和TBLPROPERTIES为新表声明分区规范和表属性。

CREATE TABLE hadoop_prod.default.sample5
USING iceberg
PARTITIONED BY (bucket(8, id), hours(ts), category)
TBLPROPERTIES ('key'='value')
AS SELECT * from hadoop_prod.default.sample3;

show create table hadoop_prod.default.sample4;
show create table hadoop_prod.default.sample5;
```

#### 4.2.4 删除表

```sql
CREATE EXTERNAL TABLE hadoop_prod.default.sample6 (
    id bigint COMMENT 'unique id',
    data string)
USING iceberg;

INSERT INTO hadoop_prod.default.sample6 values(1,'a');
DROP TABLE hadoop_prod.default.sample6;
-- 对于hadoop catalog来说，drop table会把整个表目录删除（包括data和metadata）

-- 对于hive catalog 而言，从0.14开始，DROP TABLE只会从目录中删除表。为了删除表的内容，应该使用DROP TABLE PURGE。
CREATE TABLE hive_prod.default.sample7 (
    id bigint COMMENT 'unique id',
    data string)
USING iceberg;
INSERT INTO hive_prod.default.sample7 values(1,'a');
-- /user/hive/warehouse/sample7
-- 删除表(实际上数据还在，但是重建表不能读取到数据，为什么？-- 因为metafest file指向变了)
-- 不加purge，相当于只是删除了hive中的元数据
DROP TABLE hive_prod.default.sample7;
-- 删除表和数据(data和matadata目录还在)
DROP TABLE hive_prod.default.sample7 PURGE;


CREATE TABLE hive_prod.default.sample8 (
    id bigint COMMENT 'unique id',
    data string)
USING iceberg;

INSERT INTO hive_prod.default.sample8 values(1,'a');
DROP TABLE hive_prod.default.sample8 PURGE;
```

#### 4.2.5 修改表

Iceberg在Spark 3中提供了完整`的ALTER TABLE`支持，包括：

-  重命名表
-  设置或删除表属性
-  添加、删除和重命名列
-  添加、删除和重命名嵌套字段
-  重新排序顶级列和嵌套结构字段
-  扩展`int`、`float`和`decimal`字段的类型
-  使必需列成为可选列

```sql
ALTER TABLE hadoop_prod.default.sample1
ADD COLUMNS (
    category string comment 'new_column'
  );
```



具体例子参考官网，几个需要注意的点：

1. 修改表明只支持hive catalog，不支持hadoop catalog

2. 分区操作需要使用sql扩展，[SQL扩展还](https://iceberg.apache.org/docs/latest/spark-configuration/#sql-extensions)可用于添加对分区演化和设置表的写入顺序的支持

    1. 配置方式：

       ```bash
       vim spark-default.conf
       
       spark.sql.extensions = org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions
       ```

    2. 保存后需要重启spark-sql

    3. ```sql
      -- 添加分区
      ALTER TABLE hadoop_prod.default.sample1 ADD PARTITION FIELD category;
      ALTER TABLE hadoop_prod.default.sample1 ADD PARTITION FIELD bucket(16, id);
      ALTER TABLE hadoop_prod.default.sample1 ADD PARTITION FIELD truncate(data, 4);
      -- 和bucket(16, id)冲突，需要删了再创建
      ALTER TABLE hadoop_prod.default.sample1 ADD PARTITION FIELD bucket(16, id) AS shard;
      
      -- 删除分区
      ALTER TABLE hadoop_prod.default.sample1 DROP PARTITION FIELD category;
      ALTER TABLE hadoop_prod.default.sample1 DROP PARTITION FIELD bucket(16, id);
      ALTER TABLE hadoop_prod.default.sample1 DROP PARTITION FIELD truncate(data, 4);
      ALTER TABLE hadoop_prod.default.sample1 DROP PARTITION FIELD shard;
      
      -- 修改分区
      ALTER TABLE hadoop_prod.default.sample1 ADD PARTITION FIELD bucket(16, id);
      ALTER TABLE hadoop_prod.default.sample1 REPLACE PARTITION FIELD bucket(16, id) WITH bucket(8, id);
      ```

#### 4.2.6 插入数据

```sql
CREATE TABLE hadoop_prod.default.a (
    id bigint,
    count bigint)
USING iceberg;
INSERT INTO hadoop_prod.default.a VALUES (1, 1), (2, 2), (3, 3);
select * from hadoop_prod.default.a;

CREATE TABLE hadoop_prod.default.b (
    id bigint,
count bigint,
flag string)
USING iceberg;
INSERT INTO hadoop_prod.default.b VALUES (1, 1, 'a'), (2, 2, 'b'), (4, 4, 'd');

select * from hadoop_prod.default.a;
1       1
2       2
3       3

-- merge into 行级更新 (hive中不支持这种语法)
MERGE INTO hadoop_prod.default.a t 
USING (SELECT * FROM hadoop_prod.default.b) u ON t.id = u.id
WHEN MATCHED AND u.flag='b' THEN UPDATE SET t.count = t.count + u.count
WHEN MATCHED AND u.flag='a' THEN DELETE
WHEN NOT MATCHED THEN INSERT (id,count) values (u.id,u.count);
-- id=2 --> count = 2+2
-- id=1 --> delete
-- id=4 not match --> 新增
select * from hadoop_prod.default.a;
4       4
2       4
3       3
```

#### 4.2.7 元数据查询

```sql
-- 查询表快照
SELECT * FROM hadoop_prod.default.a.snapshots;

-- 查询数据文件信息
SELECT * FROM hadoop_prod.default.a.files;

-- 查询表历史
SELECT * FROM hadoop_prod.default.a.history;

-- 查询 manifest
SELECT * FROM hadoop_prod.default.a.manifests;
```

#### 4.2.8 存储过程

Procedures可以通过CALL从任何已配置的Iceberg Catalog中使用。所有Procedures都在namespace中。

```sql
## 语法
## 1. 按照参数名传参
CALL catalog_name.system.procedure_name(arg_name_2 => arg_2, arg_name_1 => arg_1)
## 2. 按位置传递参数时，如果结束参数是可选的，则只有结束参数可以省略。
CALL catalog_name.system.procedure_name(arg_1, arg_2, ... arg_n)

2）快照管理
（1）回滚到指定的快照id
CALL hadoop_prod.system.rollback_to_snapshot('default.a', 5445001398936810299)
（2）回滚到指定时间的快照
CALL hadoop_prod.system.rollback_to_timestamp('db.sample', TIMESTAMP '2021-06-30 00:00:00.000')
（3）设置表的当前快照ID
CALL hadoop_prod.system.set_current_snapshot('db.sample', 1)
（4）从快照变为当前表状态
CALL hadoop_prod.system.cherrypick_snapshot('default.a', 7629160535368763452)

CALL hadoop_prod.system.cherrypick_snapshot(snapshot_id => 7629160535368763452, table => 'default.a' )

3）元数据管理
（1）删除早于指定日期和时间的快照，但保留最近100个快照:
CALL hive_prod.system.expire_snapshots('db.sample', TIMESTAMP '2021-06-30 00:00:00.000', 100)
（2）删除Iceberg表中任何元数据文件中没有引用的文件（孤儿文件）
#列出所有需要删除的候选文件
CALL catalog_name.system.remove_orphan_files(table => 'db.sample', dry_run => true)

#删除指定目录中db.sample表不知道的任何文件
CALL catalog_name.system.remove_orphan_files(table => 'db.sample', location => 'tablelocation/data')

（3）合并数据文件（合并小文件）
CALL catalog_name.system.rewrite_data_files('db.sample')

CALL catalog_name.system.rewrite_data_files(table => 'db.sample', strategy => 'sort', sort_order => 'id DESC NULLS LAST,name ASC NULLS FIRST')

CALL catalog_name.system.rewrite_data_files(table => 'db.sample', strategy => 'sort', sort_order => 'zorder(c1,c2)')

CALL catalog_name.system.rewrite_data_files(table => 'db.sample', options => map('min-input-files','2'))

CALL catalog_name.system.rewrite_data_files(table => 'db.sample', where => 'id = 3 and name = "foo"')

（4）重写表清单来优化执行计划
CALL catalog_name.system.rewrite_manifests('db.sample')

#重写表db中的清单。并禁用Spark缓存的使用。这样做可以避免执行程序上的内存问题。
CALL catalog_name.system.rewrite_manifests('db.sample', false)

4）迁移表
（1）快照
CALL catalog_name.system.snapshot('db.sample', 'db.snap')
CALL catalog_name.system.snapshot('db.sample', 'db.snap', '/tmp/temptable/')
（2）迁移
CALL catalog_name.system.migrate('spark_catalog.db.sample', map('foo', 'bar'))
CALL catalog_name.system.migrate('db.sample')
（3）添加数据文件
CALL spark_catalog.system.add_files(
table => 'db.tbl',
source_table => 'db.src_tbl',
partition_filter => map('part_col_1', 'A')
)

CALL spark_catalog.system.add_files(
  table => 'db.tbl',
  source_table => '`parquet`.`path/to/table`'
)
5）元数据信息
（1）获取指定快照的父快照id
CALL spark_catalog.system.ancestors_of('db.tbl')
（2）获取指定快照的所有祖先快照
CALL spark_catalog.system.ancestors_of('db.tbl', 1)
CALL spark_catalog.system.ancestors_of(snapshot_id => 1, table => 'db.tbl')
```

#### 4.2.9 通过api操作iceberg

参考代码。





### 4.3 flink集成

```bash
# 1. 上传并解压flink安装包
tar -zxvf flink-1.16.3-bin-scala_2.12.tgz -C ~/app/

# 2. 配置环境变量
sudo vim ~/.bash_profile
export HADOOP_CLASSPATH=`hadoop classpath`
source  ~/.bash_profile
# 检查是否生效
echo $HADOOP_CLASSPATH

# 3. 上传iceberg runtime包到flink的lib下
flink-sql-connector-hive-3.1.2_2.12-1.16.3.jar
iceberg-flink-runtime-1.16-1.1.0.jar
# 4. 配置
vim /home/hadoop/app/flink-1.16.3/conf/flink-conf.yaml
#修改为0.0.0.0，允许远程访问
rest.bind-address: 0.0.0.0

classloader.check-leaked-classloader: false
taskmanager.numberOfTaskSlots: 4

state.backend: rocksdb
execution.checkpointing.interval: 30000
state.checkpoints.dir: hdfs://hadoop:9000/flink-ckps
state.backend.incremental: true

# 5. 使用local模式启动
## 5.1 修改wokers（可以配置多个localhost）
vim /home/hadoop/app/flink-1.16.3/conf/workers
localhost

/home/hadoop/app/flink-1.16.3/bin/start-cluster.sh

# 查看webui：http://hadoop:8081

# 6. 启动sql-client
/home/hadoop/app/flink-1.16.3/bin/sql-client.sh embedded
```

#### 4.3.1 语法说明

```sql
CREATE CATALOG <catalog_name> WITH (
 'type'='iceberg',
 `<config_key>`=`<config_value>`
); 
```

- type: 必须是iceberg。（必须）

- catalog-type: 内置了hive和hadoop两种catalog，也可以使用catalog-impl来自定义catalog。（可选）

- catalog-impl: 自定义catalog实现的全限定类名。如果未设置catalog-type，则必须设置。（可选）

- property-version: 描述属性版本的版本号。此属性可用于向后兼容，以防属性格式更改。当前属性版本为1。（可选）

- cache-enabled: 是否启用目录缓存，默认值为true。（可选）

- cache.expiration-interval-ms: 本地缓存catalog条目的时间(以毫秒为单位)；负值，如-1表示没有时间限制，不允许设为0。默认值为-1。（可选）

#### 4.3.2 hive catalog

```sql
CREATE CATALOG hive_catalog WITH (
  'type'='iceberg',
  'catalog-type'='hive',
  'uri'='thrift://hadoop:9083',
  'clients'='5',
  'property-version'='1',
  'warehouse'='hdfs://hadoop:9000/warehouse/iceberg-hive'
);

 use catalog hive_catalog;
```



#### 4.3.3 hadoop catalog

```sql
CREATE CATALOG hadoop_catalog WITH (
  'type'='iceberg',
  'catalog-type'='hadoop',
  'warehouse'='hdfs://hadoop:9000/warehouse/iceberg-hadoop',
  'property-version'='1'
);

use catalog hadoop_catalog;
```



#### 4.3.4 其他

```sql
# upsert模式
CREATE TABLE `hive_catalog`.`test1`.`sample5` (
  `id`  INT UNIQUE COMMENT 'unique id',
  `data` STRING NOT NULL,
 PRIMARY KEY(`id`) NOT ENFORCED
) with (
'format-version'='2', 
'write.upsert.enabled'='true'
);
# 如果想要Iceberg支持基于主键的UPSERT。需要保证format-version为2
# 在UPSERT模式下，如果对表进行分区，则分区字段必须也是主键。
```

flink还有一些限制：

- 不支持创建隐藏分区的Iceberg表。
- 不支持创建带有计算列的Iceberg表。
- 不支持创建带watermark的Iceberg表。
- 不支持添加列，删除列，重命名列，更改列。
- Iceberg目前不支持Flink SQL 查询表的元数据信息，需要使用Java API 实现。



flink读写：https://iceberg.apache.org/docs/nightly/flink-writes/#metrics

通过flink流式写入iceberg的场景不多，且需要注意，flinksink底层是在flink的checkpoint时生成iceberg的数据文件并提交到iceberg存储中，如果checkpoint设置过于频繁，容易产生小文件过多的问题。一般checkpoint间隔设置为分钟级，如10分钟，30分钟。





## 附录

### 数据湖的对比

![image-20240913220447111](/image/image-20240913220447111.png)

![image-20240913220452752](/image/image-20240913220452752.png)

![image-20240913220500764](/image/image-20240913220500764.png)

![image-20240913220504628](/image/image-20240913220504628.png)

| **功能对比**                 | **Iceberg**                               | **Hudi**                                | **Delta Lake**                       | **Paimon**             |
| ---------------------------- | ----------------------------------------- | --------------------------------------- | ------------------------------------ | ---------------------- |
| **ACID**                     | Yes                                       | Yes                                     | Yes                                  | Yes                    |
| **事务管理**                 | 支持原子操作和快照隔离                    | 支持实时更新和事务管理                  | 支持ACID事务和时间旅行               | 支持实时更新和事务管理 |
| **Concurrent Multi-Writers** | Yes                                       | Yes (需要外部锁)                        | Yes                                  | Yes                    |
| **Schema演进**               | 增/删/改列 (Spark 2021.9支持/Flink/Trino) | 增/删/改列 (Spark 2022.9支持)           | 增/删/改列                           | 增/删/改列             |
| **分区演进**                 | Yes                                       | No                                      | Yes                                  | Yes                    |
| **隐藏分区**                 | Yes                                       | No                                      | Yes                                  | Yes                    |
| **行级更新**                 | COW/MOR                                   | COW/MOR                                 | COW                                  | COW                    |
| **引擎支持**                 | Flink, Spark, Hive, Trino, Presto等多种   | Flink, Spark, Hive, Trino, Presto等多种 | Spark, Presto, Hive                  | Flink, Spark           |
| **计算下推**                 | 词语+聚合下推                             | 词语下推                                | 词语+聚合下推                        | 词语+聚合下推          |
| **支持api语言**              | Java/Scala, Python                        | Java/Scala                              | Java/Scala, Python                   | Java/Scala, Python     |
| **支持文件类型**             | parquet, avro, orc                        | parquet, avro                           | parquet                              | parquet, avro          |
| **文件加密**                 | No                                        | Yes                                     | Yes                                  | Yes                    |
| **支持流写（CDC）**          | Yes                                       | Yes (append/update/delete)              | Yes                                  | Yes                    |
| **支持流读（CDC）**          | Yes                                       | Yes                                     | Yes                                  | Yes                    |
| **文件合并/排序/清理**       | 手动                                      | 自动/手动                               | 自动                                 | 自动                   |
| **社区支持**                 | 活跃的开源社区                            | 活跃的开源社区，国内支持较好            | 由Databricks主导，开源版本有一定限制 | 国内支持较好，社区活跃 |



### 周边项目

1. apache kyuubi：Apache Kyuubi, a distributed and multi-tenant gateway to provide serverless SQL on lakehouses.

   可以理解为在spark-sql的基础上进一步完善功能。（多租户管理）

   ![image-20240922171435789](/image/image-20240922171435789.png)

2. apache amoro：Management system for Lakehouse

   基于数据湖的一个管理系统，可以帮我们管理和监控数据湖，尤其是自动化的小文件合并功能。

   ![image-20240922171257049](/image/image-20240922171257049.png)
