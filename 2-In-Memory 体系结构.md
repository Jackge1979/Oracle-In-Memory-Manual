# 第二章 Oracle Database In-Memory 列存储体系结构
  In-Memory 列存储 （IM列存储）在内存中使用为快速扫描优化的列格式存储表和分区。 Oracle数据库使用复杂的架构同时以列和行格式管理数据。

## 2.1 两种格式(dual-format)：列和行

  启用IM列存储时，SGA在单独的位置管理数据：In-Memory区域和数据库数据库缓冲区高速缓存（Buffer Cache）。

### 2.1.1 In-Memory区域

  IM列存储以列格式对数据进行编码：每个列是单独的结构。 这些列是连续存储的，它们对分析查询进行优化。 数据库缓冲区高速缓存（buffer cache ）可以修改对象，也可以在IM列存储中填充的对象。 但是，缓冲区高速缓存（buffer cache ）以传统的行格式存储数据。 数据块连续存储行，优化它们的事务。

  下图说明了基于行的存储和列式存储之间的区别。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr9978JKo74haAfsLpEEFPHvn9YNuicPtIjdicUxeyF7uDqbwtlEzZ64dzfDQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-1 列和基于行的存储

  本节创建以下主题：

  * In-Memory  Area 中的列数据
  * In-Memory  Area 包含IM列存储的可选SGA组件。

  数据库缓冲区高速缓存（Buffer Cache）中的行数据
无论IM列存储是启用还是禁用，数据库缓冲区高速缓存（buffer cache）都以相同的方式存储和处理数据块。 缓冲区I / O和缓冲池功能完全相同。

#### 2.1.1.1 In-Memory Area 中的列数据

  In-Memory Area 是包含IM列存储的可选SGA组件。

  此部分包含以下主题：

   * In-Memory Area 的大小

   In-Memory Area 由 INMEMORY_SIZE 初始化参数控制。 默认情况下，In-Memory Area 的大小为0，这意味着IM列存储被禁用。

   * In-Memory Area 中的内存池（Memory Pools）

   In-Memory Area 分为列数据和元数据的子池。
  
#### 2.1.1.1.1 In-Memory 区域的大小

  内存区域由 INMEMORY_SIZE 初始化参数控制。 默认情况下，内存区域的大小为0，这意味着IM列存储被禁用。

  要启用IM列存储，请将 In-Memory Area 设置为至少100 MB。 大小显示在 V$SGA 中。

  从 SGA_TARGET 初始化参数设置中减去 In-Memory Area 的值。 例如，如果将 SGA_TARGET 设置为10 GB，并且如果将 INMEMORY_SIZE 设置为4 GB，则 SGA_TARGET 设置的40％将分配给 In-Memory 区域。 下图说明了这种关系。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr9975IMicocCoqJpic4dkjRj0ds14bxnia7uZgicHmGibCvLYibq7Cd6hlKQls0A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-2 INMEMORY_SIZE和SGA_TARGET 参数

  与SGA的其他组件（包括buffer cache和shared pool）不同，In-Memory Area 大小不受自动内存管理控制。 当 buffer cache 或 shared pool 需要更多内存时，数据库不会自动缩小 In-Memory Area ，或者当内存空间不足时，增加 In-Memory Area 。 您只能通过手动调整 INMEMORY_SIZE 初始化参数来增加 In-Memory Area 的大小。

  从 Oracle Database 12c Release 2（12.2）开始，可以使用 ALTER SYSTEM 语句动态增加 INMEMORY_SIZE 。 满足以下条件时，数据库分配增加的内存：

  * SGA中有可用的空闲内存。

  * INMEMORY_SIZE 的新大小比当前设置大至少128 MB。

  '注:您不能使用 ALTER SYSTEM 来减少 INMEMORY_SIZE。'

  V$INMEMORY_AREA 和 V$SGA 视图立即反映了更改。
  

  **In-Memory Area 中的内存池（Memory Pools）**
  
  In-Memory Area 为列数据和元数据的子池。

  In-Memory area 被细分为以下子池：

  * 列数据池

  此子池存储IMCU，其中包含列数据。 V$INMEMORY_AREA.POOL 列将此子池标识为1MB POOL，如示例2-1所示。

  * 元数据池

  此子池存储有关驻留在IM列存储中的对象的元数据。 V$INMEMORY_AREA.POOL 列将此子池标识为 64KB POOL，如示例2-1所示。

  ![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997XvCb0r3MpvK4jFJvwht24V5icJhyc0SFfXDezAEYep4fhFYU2LTPuQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  图2-3内存区域中的子池

  数据库使用内部启发式算法确定两个子池的相对大小。 数据库将 In-Memory Area 中的大部分空间分配给列式数据池（1 MB pool）。


  '注:Oracle数据库自动确定子池大小。 您不能更改空间分配。'

  示例 2-1 V$INMEMORY_AREA 视图

  此示例查询 V$INMEMORY_AREA 视图以确定每个子池（包括示例输出）中的可用内存量：

```
COL POOL FORMAT a9
COL POPULATE_STATUS FORMAT a15
SSELECT POOL, TRUNC(ALLOC_BYTES/(1024*1024*1024),2) "ALLOC_GB",
        TRUNC(USED_BYTES/(1024*1024*1024),2) "USED_GB",
        POPULATE_STATUS
FROM    V$INMEMORY_AREA;

POOL      ALLOC_GB   USED_GB    POPULATE_STATUS
--------- ---------- ---------- ---------------
1MB POOL  7.99       0          DONE
64KB POOL 1.98       0          DONE
```

  In-Memory area 的当前大小在V$SGA 视图中看到：

```
SELECT NAME, VALUE/(1024*1024*1024) "SIZE_IN_GB"
FROM   V$SGA 
WHERE  NAME LIKE '%Mem%';

NAME                 SIZE_IN_GB
-------------------- ----------
In-Memory Area       10
```

在此示例中，分配给子池的内存为9.97 GB，而 In-Memory Area 的大小为10 GB。 数据库使用小百分比的内存用于内部管理结构。

### 2.1.2 数据库缓冲区高速缓存（Buffer Cache）中的行数据

  无论IM列存储是启用还是禁用，数据库缓冲区高速缓存（buffer cache）都以相同的方式存储和处理数据块。 缓冲区I / O和缓冲池功能完全相同。

  IM列存储允许在SGA中以传统行格式（缓冲区高速缓存）和列格式同时填充数据。 数据库透明地将OLTP查询（例如主键查找）发送到缓冲区高速缓存，以及分析和报告查询到IM列存储。 在提取数据时，Oracle数据库还可以从同一查询中的两个内存区域读取数据。

  '注:在执行计划中，TABLE ACCESS IN MEMORY FULL 操作表示在IM列存储中访问一些或所有数据。'

  双格式（dual-format）架构不会增加内存需求。 缓冲区高速缓存（buffer cache）被优化为以比数据库的大小小得多的大小运行。

下图显示了示例IM列存储。 数据库以传统行格式将 sh.sales 表存储在磁盘上。 SGA将数据以列格式存储在IM列存储中，并以行格式存储在数据库缓冲区高速缓存（buffer cache）中。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997ZkCeQ0pVEW6jdAVahZdwHEyPpE06tTOkZHhcYe69TQSLMHdlaGDmoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2-4 IM列存储

IM列存储支持永久性，堆组织表（heap-organized tables）的每种磁盘数据格式。 列格式不会影响存储在数据文件或缓冲区高速缓存中的数据的格式，也不会影响 undo 数据和联机 redo 日志记录。

数据库以相同的方式处理DML修改，无论是否启用IM列存储，通过更新缓冲区高速缓存（buffer cache）、联机 redo 日志和 undo 表空间。 但是，数据库使用内部机制来跟踪更改，并确保IM列存储与数据库的其余部分一致。 例如，如果 sales 表填充在IM列存储中，并且如果应用程序更新 sales 中的行，则数据库自动使IM列存储中的 sales 表副本保持事务一致。 访问IM列存储的查询始终对访问缓冲区高速缓存（buffer cache）的查询返回相同的结果。

## 2.2 In-Memory 存储单元

IM列存储管理优化存储单元中的数据和元数据，而不是传统的Oracle数据块。

Oracle数据库在 In-Memory Area 中维护存储单元。 下图显示了In-Memory Area和与其交互的数据库进程的概述。 其余章节描述各种存储器组件。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997kuGpyrfF8KJmpmqiarlrdDwgfHWt5ATFgyKcp7k2xiaqUF0k2PfbHmnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-5 IM列存储：内存和进程体系结构

此部分包含以下主题：

* In-Memory 压缩单元（IMCU）

In-Memory 压缩单元（IMCU）是包含用于一个或多个列的数据的压缩的只读存储单元。

* 快照元数据单元（SMU）

快照元数据单元（SMU）包含关联的IMCU的元数据和事务信息。

* In-Memory内存表达式单元（IMEU）

In-Memory表达式单元（IMEU）是用于实现In-Memory表达式（IM表达式）和用户定义的虚拟列的存储容器。


### 2.2.1 In-Memory 压缩单元（IMCU）

In-Memory 压缩单元（IMCU）是包含用于一个或多个列的数据的压缩的只读存储单元。

IMCU类似于表空间范围。 IMCU具有两个部分：一组列压缩单元（CU）和包含诸如IM存储索引的元数据的头。

此部分包含以下主题：

* IMCUs 和 Schema 对象

IM列存储将单个对象（表、分区、物化视图）的数据存储在一组IMCU中。 IMCU存储一个且仅一个对象的列数据。

* 列压缩单元 (CU)

列压缩单元（CU）是IMCU中的单个列的连续存储。 每个IMCU具有一个或多个CU。

* In-Memory 存储索引

每个IMCU头都自动创建和管理其CU的In-Memory存储索引（IM存储索引）。 IM存储索引存储IMCU内所有列的最小值和最大值。


#### 2.2.1.1 IMCU 和 Schema 对象

IM列存储将单个对象（表、分区、物化视图）的数据存储在一组IMCU中。 IMCU存储一个且仅一个对象的列数据。

对于指定为 INMEMORY的对象，INMEMORY 子句中列出的每个列都包含在每个IMCU中。 例如，sh.sales 表有7列，如图 2-6 所示。 以下DDL语句将表指定为 INMEMORY，这意味着每个 sales 的IMCU都包括这7列的列数据：

```
ALTER TABLE sh.sales INMEMORY MEMCOMPRESS FOR QUERY LOW;
```

要将 INMEMORY 属性应用于段中的一部分列，必须在一个DDL语句中将所有列指定为 INMEMORY，然后发出第二个DDL语句以指定排除的列上的 NO INMEMORY 属性。 例如，以下语句指定 sh.sales 中的3列为 NO INMEMORY ，这意味着表中的其他4列保留其 INMEMORY 属性：

```
ALTER TABLE sh.sales INMEMORY MEMCOMPRESS FOR QUERY LOW 
  NO INMEMORY (promo_id, quantity_sold, amount_sold);
```

下图显示了IM列存储中填充的sh模式中的三个表：customers、 products 和 sales。 在本示例中，每个表都有指定 INMEMORY 的不同数目的列。 每个表的IMCU只包括指定列的数据。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997uCh4LBoBo2wPko0U6b0yVvDgqjO4KGL9mHjHrPVplGbfjyNic1CZWQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-6 列和IMCU

此部分包含以下主题：

* In-Memory 压缩

IM列存储使用针对访问速度而不是存储缩减优化的特殊压缩格式。 列格式允许直接对压缩列执行查询。

* IMCU 和 行

每个IMCU包含表段中的行的子集的所有列值（包括空值）。 行的子集称为颗粒。

##### 2.2.1.1.1 In-Memory 压缩

IM列存储使用针对访问速度而不是存储缩减优化的特殊压缩格式。 列格式允许直接对压缩列执行查询。

压缩使扫描和过滤操作能够处理少得多的数据，从而优化查询性能。 Oracle数据库仅在结果集需要数据时解压缩数据。

在IM列存储中应用的压缩与混合列压缩密切相关。 两种技术处理列向量，主要区别是用于IM列存储的列向量针对SIMD向量处理进行优化，而混合列压缩的列向量针对磁盘存储进行优化。

当您启用要填充到IM列存储中的对象时，在 INMEMORY 子句中指定压缩类型：FOR DML、FOR QUERY (LOW 或 HIGH)、FOR CAPACITY (LOW 或 HIGH) 或 NONE。

##### 2.2.1.1.2 IMCU 和 行

每个IMCU包含表段中的行的子集的所有列值（包括空值）。 行的子集称为颗粒。

给定段的所有IMCU包含大致相同的行数。 Oracle数据库根据数据类型、数据格式和压缩类型自动确定颗粒的大小。 较高的压缩级别导致IMCU中的更多行。

在IMCU和一组数据库块之间存在一对多映射。 如示例 2-2 所示，每个IMCU存储用于不同块集合的列的值。

IMCU中的列不排序。 Oracle数据库按照从磁盘读取的顺序填充它们。

IMCU中的行数决定了IMCU消耗的空间量。 如果目标行数导致IMCU增长超过在1MB池中可用的连续1MB区段的量，则IMCU创建附加区段（块）以保持剩余的列CU。 IMCU始终以1 MB为增量分配空间。

*示例 2-2 IMCU和行子集*

在此简化示例中，只有 customers 表的以下4列具有 INMEMORY 属性：cust_id、cust_first_name、cust_last_name 和 cust_gender。 表中仅存在5行，存储在2个数据块中。 概念上，第一数据块存储其行如下：

```
82,Madeline,Li,F;37004,Abel,Embrey,M;1714,Hardy,Gentle,M
```

第二个数据块按如下所示存储行：

```
100439,Uma,Campbell,F;3047,Lucia,Downey,F
```

假设IMCU 1存储第一数据块的数据。 在这种情况下，该数据块存储中的3行的 cust_id 列值如下所示“垂直”存储在CU内：

```
82
37004
1714
```
IMCU 2存储来自第二数据块的数据。 这两行的 cust_id 列值存储在CU中，如下所示：

```
100439
3047
```

因为 cust_id 值是数据块中每行的第一个值，所以 cust_id 列位于IMCU中的第一个位置。 列始终占据相同的位置，因此Oracle数据库可以通过读取段的IMCU重建行。

#### 2.2.1.2 列压缩单元 (CU)

列压缩单元（CU）是IMCU中的单个列的连续存储。 每个IMCU具有一个或多个CU。

此部分包含以下主题：

* CU的结构

CU被划分为主体和头部。

* 本地词典（Local Dictionary）

在CU中，本地字典具有不同值的列表及其对应的字典代码。

##### 2.2.1.2.1 CU的结构

CU被划分为主体和头部。

每个CU的主体存储包括在IMCU中的行范围的列值。 头包含关于存储在CU体中的值的元数据，例如CU内的最小值和最大值。 它还可以包含本地字典，其是该列中的不同值的排序列表及其对应的字典代码。

下图显示了 sales 表的4个CU的IMCU：prod_id、cust_id、time_id 和 channel_id。 每个CU存储包括在IMCU中的行范围的列值。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997ibzvsCLB6xYIBeDfetRu7IcW8xVPLvcBb4OhzfSoJicxibhRAOvgxBKPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-7 IMCU中的CU

CU按rowid顺序存储值。 因此，数据库可以通过将行“拼接”在一起来回答查询。 例如，应用程序发出以下查询：

```
SELECT cust_id, time_id, channel_id FROM   sales WHERE  prod_id = 5;
```

数据库通过对值为5的条目 prod_id 列开始扫描。假设数据库在 prod_id 列中的位置2中找到5。 数据库现在必须找到此行的相应cust_id，time_id和channel_id。 因为CU按rowid顺序存储数据，所以数据库可以在那些列的位置2中找到对应的 cust_id、time_id, and channel_id 值。 因此，为了回答查询，数据库必须从 cust_id、time_id, and channel_id 列中的位置2提取值，然后将该行拼接在一起以将其返回给最终用户。


##### 2.2.1.2.2 本地词典（Local Dictionary）

在CU中，本地字典具有不同值的列表及其对应的字典代码。

本地字典存储列中包含的符号。 下图说明了CU如何在 vehicles 表中存储 name 列。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997nUkZJbmrJhHFDTSdyQffNk5NN3TJGpjuJ0IaKkbIgFTVsnqsKu800w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-8 本地词典

在前面的图中，CU只包含7行。 该CU中的每个不同值（例如 Cadillac 或 Audi）被分配不同的字典代码，诸如对于 Cadillac 为2，对于 Audi 为 0。 CU存储字典代码而不是原始值。

```
注:
当数据库对连接组（join group）使用公共字典（common dictionary）时，本地字典包含对公共字典的引用，而不是符号。 例如，不是存储用于 vehicles.name 列的值 Audi, BWM 和 Cadillac，而是本地字典存储诸如101，220和66的字典代码。
```

CU头包含列的最小值和最大值。 在本示例中，最小值为 Audi，最大值为 Cadillac。 本地词典存储不同值的列表：Audi, BMW 和 Cadillac。 它们对应的字典代码（0, 1 和 2）是隐式的。 每个IMCU中的CU的本地字典独立于其他IMCU中的本地字典。

如果一个查询过滤 Audi 汽车，那么数据库只扫描这个IMCU只有 0 个代码。


#### 2.2.1.3 In-Memory 存储索引

每个IMCU头都自动创建和管理其CU的In-Memory存储索引（IM存储索引）。 IM存储索引存储IMCU内所有列的最小值和最大值。

例如，sales 填充在IM列存储中。 此表的每个IMCU都有所有列。  sales.prod_id 列存储在每个IMCU内的单独CU中。 IMCU报头具有每个 prod_id  CU（以及其它所有CU）的最小值和最大值。

为了消除不必要的扫描，数据库可以基于SQL过滤谓词执行IMCU修剪。 数据库仅扫描满足查询谓词的IMCU，如下图中的 WHERE prod_id > 14 AND prod_id < 29  示例所示。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997dic0gpibg4oQ1cngAjBLGqDJSXHvhKph9ZKAjTMZQWENppr5ruSmUvcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-9 列数据的存储索引

### 2.2.2 快照元数据单元（SMU）

快照元数据单元（SMU）包含关联的IMCU的元数据和事务信息。

此部分包含以下主题：

* IMCU 和 SMU

In-Memory Area的列池存储实际数据：IMCU和IMEU。 In-Memory Area中的元数据池存储SMU。

* 事务日志（Transaction Journal）

每个SMU包含一个事务日志。 数据库使用事务日志来使IMCU在事务上保持一致。

#### 2.2.2.1 IMCU 和 SMU

In-Memory Area 的列池存储实际数据：IMCU和IMEU。 In-Memory Area 中的元数据池存储SMU。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr997cKWjibppNnJiaW5h1QCVdSAiaSCUmo1XJjBFmlYHx6ftTdEibIo505ooxQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-10 IMCU和SMU

此图显示数据池中的IMCU和元数据池中的SMU。


每个IMCU映射到单独的SMU。 因此，如果列式数据池包含100个IMCU，则元数据池包含100个SMU。 SMU为其关联的IMCU存储多种类型的元数据，包括以下内容：

* 对象号

* 列号

* 映射行的信息

事务日志（Transaction Journal）
每个SMU包含一个事务日志。 数据库使用事务日志来使IMCU在事务上保持一致。

数据库使用缓冲区高速缓存（buffer cache）来处理DML，就像未启用IM列存储一样。 例如，UPDATE 语句可能修改IMCU中的行。 在这种情况下，数据库将已修改行的rowid添加到事务日志，并将其标记为从DML语句的SCN起已过期。 如果查询需要访问该行的新版本，则数据库从数据库缓冲区高速缓存中获取该行。

Description of Figure 2-11 follows

图2-11事务日志（Transaction Journal）

数据库通过合并列、事务日志（transaction journal）和缓冲区高速缓存（buffer cache）的内容来实现读取一致性。 当IMCU在重新填充期间刷新时，查询可以直接从IMCU访问最新的行。


In-Memory 表达式单元 (IMEU)
In-Memory Expression Unit (IMEU) 是用于实现内存表达式（IM表达式）和用户定义的虚拟列的存储容器。

数据库将物化表达式视为IMCU中的其他列。 从概念上讲，IMEU是其父IMCU的逻辑扩展。 正如IMCU可以包含多个列，IMEU可以包含多个虚拟列。

每个IMEU映射到一个IMCU，映射到相同的行集。 IMEU包含其相关IMCU中包含的数据的表达式结果。 当IMCU被填充时，相关联的IMEU也被填充。

典型的IM表达式涉及一个或多个列，可能具有常量，并且与表中的行具有一对一映射。 例如，employees 表的IMCU包含列为 weekly_salary 的行1-1000。 对于存储在此IMCU中的行，IMEU计算自动检测到的IM表达式 weekly_salary*52和用户定义的虚拟列 quarterly_salary 定义为 weekly_salary*12。 IMCU中的第三行向下映射到IMEU中的第三行。

IMEU是特定段的IMCU的逻辑扩展。 默认情况下，IMEU从基段继承 INMEMORY 子句属性，包括Oracle Real Application Clusters（Oracle RAC）属性，如 DISTRIBUTE 和 DUPLICATE。 您可以选择性地启用或禁用IMEU中存储的虚拟列。 您还可以为不同的列指定压缩级别。


表达式统计存储 (ESS)
表达式统计存储（ESS）是由优化器维护的存储关于表达式求值的统计的存储库。 ESS驻留在SGA中，并且仍保留在磁盘上。

启用IM列存储时，数据库会利用ESS的 In-Memory 表达式（IM表达式）功能。 但是，ESS独立于IM列存储。 ESS是数据库的永久组件，不能禁用。

数据库使用ESS来确定表达式是否“热”（经常访问），并且因此是IM表达式的候选。 在查询的硬解析期间，ESS在 SELECT 列表中查找活动表达式，WHERE 子句、GROUP BY 子句等。

对于每个段，ESS维护表达式统计信息，例如：

执行频率

评估成本

时间戳评估

优化器根据成本和评估的次数，为每个表达式分配一个加权分数。 这些值是近似值而不是精确值。 更活跃的表达式具有更高的分数。 ESS维护最常访问的表达式的内部列表。

使用 DBMS_INMEMORY_ADMIN 包控制IM表达式的行为。 例如，IME_CAPTURE_EXPRESSIONS 过程提示数据库标识并逐渐填充数据库中最热的表达式。 IME_POPULATE_EXPRESSIONS 过程强制数据库立即填充表达式。

ESS信息存储在数据字典中，并在 DBA_EXPRESSION_STATISTICS 视图中显示。 此视图显示优化程序发送到ESS的元数据。 IM表达式在 DBA_IM_EXPRESSIONS 视图中显示为系统生成的虚拟列，前缀为字符串 SYS_IME。


In-Memory 进程架构
响应于查询和DML，服务器进程扫描列数据并更新SMU元数据。 后台进程将磁盘中的行数据填充到IM列存储中。

此部分包含以下主题：

In-Memory 协调器进程（IMCO）
In-Memory协调器进程（IMCO）管理IM列存储的许多任务。 它的主要任务是启动背景填充和列数据的重新填充。

空间管理工作进程（Wnnn）
空间管理工作进程（Wnnn）代表IMCO填充或重新填充数据。


In-Memory 协调器进程 (IMCO)
In-Memory 协调器进程（IMCO）管理IM列存储的许多任务。 它的主要任务是启动后台填充和列数据的重新填充。

Population是一种流式处理机制，将行数据转换为列格式，然后压缩它。 IMCO自动启动具有除  NONE 之外的任何优先级的 INMEMORY 对象的填充。 当访问优先级为  NONE 的对象时，IMCO使用空间管理工作进程（Wnnn）进程填充它们。

当IMCO后台进程满足临时阈值时，它还启动IM列存储对象的基于阈值的重新填充。 IMCO可以对具有过期条目但不满足过期阈值的IM列存储中的任何IMCU发起涓流（trickle）重新填充。

涓流重新填充（Trickle repopulation）在后台自动发生。 步骤如下：

IMCO 唤醒。

IMCO确定是否需要执行群体任务，包括IMCU中是否存在过时的条目。

如果IMCO找到过时的条目，则它触发空间管理工作进程以重新填充IMCU中的这些条目。

IMCO睡眠两分钟，然后返回到步骤1。




空间管理工作进程（Wnnn）
空间管理工作进程（Wnnn）代表IMCO填充或重新填充数据。

在填充期间，Wnnn进程负责创建IMCU、SMU和IMEU。 创建IMEU时，工作进程执行以下任务：

识别人口的虚拟列

创建虚拟列值

计算每一行的值，将数据转换为列格式，并压缩它

向空间层注册对象

将IMEU与其对应的IMCU关联

注:

在IMEU创建期间，父IMCU仍可用于查询。

在重新填充期间，Wnnn进程基于现有的IMCU和事务日志创建IMCU的新版本，同时临时保留旧版本。 这种机制称为双缓冲。

数据库可以快速地将IM表达式移入和移出IM列存储。 例如，如果IMCU是在没有IMEU的情况下创建的，则数据库可以稍后添加IMEU，而不强制IMCU经历完全重新填充机制。

INMEMORY_MAX_POPULATE_SERVERS 初始化参数控制可以启动用于填充的工作进程的最大数量。INMEMORY_TRICKLE_REPOPULATE_PERCENT 初始化参数控制工作进程可以执行涓流重新填充（trickle repopulation）的最大时间百分比。

## 2.3 CPU架构：SIMD向量处理（Vector Processing）

对于需要在IM列存储中扫描的数据，数据库使用SIMD（单指令，多数据）向量处理。

IM列存储最大化了可以加载到向量寄存器和求值的列条目的数量。 不是一次一个地评估列中的每个条目，数据库在单个CPU指令中评估一组列值。 SIMD向量处理使数据库能够每秒扫描数十亿行。

例如，应用程序发出查询以查找 sales 表中使用 promo_id 值为 9999 的订单总数。sales 表驻留在IM列存储中。 查询通过仅扫描 sales.promo_id 列开始，如下图所示：

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKspYLOUzYSWx6V9Rz0sx3VPWb4q72hRxL05yEEwVibPNRZTsypWxpwjBpm98UJYTGia1hzZNhHW7Oww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
图 2-12 SIMD向量处理

CPU按如下方式计算数据：

* 将前8个值（数值根据数据类型和压缩模式而变化）从 promo_id 列装入SIMD寄存器，然后将它们与单个指令中的值9999进行比较。

* 丢弃条目。

* 将另外8个值加载到SIMD寄存器中，然后以此方式继续，直到它已评估所有条目。
