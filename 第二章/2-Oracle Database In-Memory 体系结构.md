# 2 In-Memory 列存储体系结构
In-Memory 列存储 （IM列存储）在内存中使用为快速扫描优化的列格式存储表和分区。 Oracle数据库使用复杂的架构同时以列和行格式管理数据。

### 两种格式(dual-format)：列和行

启用IM列存储时，SGA在单独的位置管理数据：In-Memory区域和数据库数据库缓冲区高速缓存（Buffer Cache）。

IM列存储以列格式对数据进行编码：每个列是单独的结构。 这些列是连续存储的，它们对分析查询进行优化。 数据库缓冲区高速缓存（buffer cache ）可以修改对象，也可以在IM列存储中填充的对象。 但是，缓冲区高速缓存（buffer cache ）以传统的行格式存储数据。 数据块连续存储行，优化它们的事务。

下图说明了基于行的存储和列式存储之间的区别。

![](http://mmbiz.qpic.cn/mmbiz_png/6F1WRDupvKs0vwRBm8SRbGEkJLyCr9978JKo74haAfsLpEEFPHvn9YNuicPtIjdicUxeyF7uDqbwtlEzZ64dzfDQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图 2-1 列和基于行的存储

本节创建以下主题：

* In-Memory  Area 中的列数据
* In-Memory  Area 包含IM列存储的可选SGA组件。

数据库缓冲区高速缓存（Buffer Cache）中的行数据
无论IM列存储是启用还是禁用，数据库缓冲区高速缓存（buffer cache）都以相同的方式存储和处理数据块。 缓冲区I / O和缓冲池功能完全相同。

* **In-Memory 区域中的列数据**

In-Memory  Area 是包含IM列存储的可选SGA组件。

此部分包含以下主题：

* In-Memory  Area 的大小

In-Memory  Area 由 INMEMORY_SIZE 初始化参数控制。 默认情况下，In-Memory  Area 的大小为0，这意味着IM列存储被禁用。

* In-Memory  Area 中的内存池（Memory Pools）

In-Memory  Area 分为列数据和元数据的子池。


* In-Memory 区域的大小

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


In-Memory Area 中的内存池（Memory Pools）
In-Memory Area 为列数据和元数据的子池。

In-Memory area 被细分为以下子池：

列数据池

此子池存储IMCU，其中包含列数据。 V$INMEMORY_AREA.POOL 列将此子池标识为1MB POOL，如示例2-1所示。

元数据池

此子池存储有关驻留在IM列存储中的对象的元数据。 V$INMEMORY_AREA.POOL 列将此子池标识为 64KB POOL，如示例2-1所示。

Description of Figure 2-3 follows
图2-3内存区域中的子池

数据库使用内部启发式算法确定两个子池的相对大小。 数据库将 In-Memory Area 中的大部分空间分配给列式数据池（1 MB pool）。


注:

Oracle数据库自动确定子池大小。 您不能更改空间分配。

示例 2-1 V$INMEMORY_AREA 视图

此示例查询 V$INMEMORY_AREA 视图以确定每个子池（包括示例输出）中的可用内存量：

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
In-Memory area 的当前大小在V$SGA 视图中看到：

SELECT NAME, VALUE/(1024*1024*1024) "SIZE_IN_GB"
FROM   V$SGA 
WHERE  NAME LIKE '%Mem%';

NAME                 SIZE_IN_GB
-------------------- ----------
In-Memory Area       10
在此示例中，分配给子池的内存为9.97 GB，而 In-Memory Area 的大小为10 GB。 数据库使用小百分比的内存用于内部管理结构。

数据库缓冲区高速缓存（Buffer Cache）中的行数据

无论IM列存储是启用还是禁用，数据库缓冲区高速缓存（buffer cache）都以相同的方式存储和处理数据块。 缓冲区I / O和缓冲池功能完全相同。

IM列存储允许在SGA中以传统行格式（缓冲区高速缓存）和列格式同时填充数据。 数据库透明地将OLTP查询（例如主键查找）发送到缓冲区高速缓存，以及分析和报告查询到IM列存储。 在提取数据时，Oracle数据库还可以从同一查询中的两个内存区域读取数据。

注:

在执行计划中，TABLE ACCESS IN MEMORY FULL 操作表示在IM列存储中访问一些或所有数据。

双格式（dual-format）架构不会增加内存需求。 缓冲区高速缓存（buffer cache）被优化为以比数据库的大小小得多的大小运行。

下图显示了示例IM列存储。 数据库以传统行格式将 sh.sales 表存储在磁盘上。 SGA将数据以列格式存储在IM列存储中，并以行格式存储在数据库缓冲区高速缓存（buffer cache）中。

Description of Figure 2-4 follows
图2-4 IM列存储

IM列存储支持永久性，堆组织表（heap-organized tables）的每种磁盘数据格式。 列格式不会影响存储在数据文件或缓冲区高速缓存中的数据的格式，也不会影响 undo 数据和联机 redo 日志记录。

数据库以相同的方式处理DML修改，无论是否启用IM列存储，通过更新缓冲区高速缓存（buffer cache）、联机 redo 日志和 undo 表空间。 但是，数据库使用内部机制来跟踪更改，并确保IM列存储与数据库的其余部分一致。 例如，如果 sales 表填充在IM列存储中，并且如果应用程序更新 sales 中的行，则数据库自动使IM列存储中的 sales 表副本保持事务一致。 访问IM列存储的查询始终对访问缓冲区高速缓存（buffer cache）的查询返回相同的结果。