# 4 为In-Memory填充(population)启用对象

本章介绍如何在IM列存储中启用和禁用填充对象，包括设置压缩和优先级选项。

本章包含以下主题：

* 关于 In-Memory

填充当数据库从磁盘读取现有行格式数据，将其转换为列格式，然后将其存储在IM列存储中时，发生In-Memory 填充 (population)。只有具有  INMEMORY属性的对象才有资格进行填充。

* 启用和禁用IM列存储的表

通过在CREATE TABLE 或 ALTER TABLE 语句中包含 INMEMORY 子句来启用IM列存储的表。通过在CREATE TABLE 或 ALTER TABLE 语句中包含 NO INMEMORY 子句来禁用IM列存储的表。

* 启用和禁用内存表的列

您可以为单独的列指定 INMEMORY 子句。非虚拟列和In-Memory虚拟列（IM虚拟列）都有资格进入IM列存储。

* 启用和禁用IM列存储的表空间

您可以启用或禁用IM列存储的表空间。

* 启用和禁用IM列存储的物化视图

您可以为IM列存储启用和禁用物化视图。

* In-Memory对象的强制填充：教程

启用In-Memory填充的对象不会立即填充该对象。

* 为IM列存储启用ADO

信息生命周期管理（ILM）是一组用于管理从创建到归档或删除的数据的过程和策略。

## 4.1 关于In-Memory 填充

当数据库从磁盘读取现有行格式数据，将其转换为列格式，然后将其存储在IM列存储中时，发生In-Memory填充(population)（填充）。只有具有 INMEMORY 属性的对象才有资格进行填充。

此部分包含以下主题：

* In-Memory 填充的目的

IM列存储不会自动将数据库中的所有对象加载到IM列存储中。

* In-Memory填充如何工作

您可以指定数据库在数据库实例启动时或访问 INMEMORY 对象时填充IM列存储中的对象。填充算法也会因使用单实例还是Oracle     RAC而有所不同。

* In-Memory 填充的控制

使用数据定义语言（DDL）语句中的INMEMORY子句指定哪些对象适合填充到IM列存储中。您可以启用表空间、表、分区和物化视图。

### 4.1.1 In-Memory 填充的目的

IM列存储不会自动将数据库中的所有对象加载到IM列存储中。

如果不使用DDL将任何对象指定为 INMEMORY，则IM列存储器保持为空。要将行从用户指定的 INMEMORY对象转换为列格式，以便它们可用于分析查询，需要填充。

将磁盘上的现有数据转换为列格式的填充与将新数据加载到IM列存储中的重新填充不同。由于IMCU是只读结构，因此当行更改时，Oracle数据库不会填充它们。相反，数据库在事务日志中记录行更改，然后创建新的IMCU作为重新填充的一部分。

### 4.1.2 In-Memory 填充如何工作

您可以指定数据库在数据库实例启动时或访问INMEMORY 对象时填充IM列存储中的对象。填充算法也会因使用单实例还是Oracle RAC而有所不同。

此部分包含以下主题：

* In-Memory 填充优先级

DDL语句包括 INMEMORY     PRIORITY子句，它提供对群体队列的更多控制。

* 后台进程如何填充IMCU

在填充期间，数据库以其行格式从磁盘读取数据，扭转行以创建列，然后将数据压缩到。

#### 4.1.2.1 In-Memory 填充优先级

DDL语句包括 INMEMORY PRIORITY 子句，它提供对群体队列的更多控制。

```
注:
INMEMORY PRIORITY 子句控制填充的优先级，但不控制填充的速度。
```

优先级级别设置适用于整个表、分区或子分区，而不适用于不同的列子集。在对象上设置 INMEMORY 属性意味着该对象是IM列存储中的填充的候选对象。这并不意味着数据库立即填充对象。 

Oracle数据库管理优先级如下：

* 按需（On-demand）填充

默认情况下，INMEMORY PRIORITY 参数设置为 NONE。在这种情况下，数据库只在通过全表扫描访问对象时填充该对象。如果对象从未被访问过，或者只能通过索引扫描或者通过rowid获取，那么将不会发生填充。

* 优先级（Priority-based）的填充

当 PRIORITY 设置为 NONE以外的值时，Oracle数据库使用内部管理的优先级队列自动填充对象。在这种情况下，全扫描不是填充的必要条件。数据库执行以下操作：

    * 在数据库实例重新启动后自动填充IM列存储中的列数据

    * 根据指定的优先级排列 INMEMORY 对象的队列数

    例如，使用 INMEMORY PRIORITYCRITICAL 更改的表优先于使用 INMEMORY PRIORITYHIGH修改的表， INMEMORY PRIORITY HIGH的表优先于 INMEMORY PRIORITYLOW修改的表。如果IM列存储空间不足，则Oracle数据库在空间可用之前不会填充其他对象。

    * 等待从 ALTER TABLE 或 ALTER MATERIALIZED VIEW 语句返回，直到对象的更改记录在IM列存储中

在IM列存储中填充了段之后，数据库只会在删除或移动段时将其逐出，或者使用 NO INMEMORY 属性更新段。您可以手动或通过ADO策略驱逐段。


*示例4-1 IM列存储中的对象的填充*

在完成此示例之前，必须为数据库启用IM列存储。

1.以管理员身份登录到数据库，然后查询customers表，如下所示：

```
SELECT cust_id, cust_last_name, cust_first_name 
FROM   sh.customers 
WHERE  cust_city = 'Hyderabad' 
AND    cust_income_level LIKE 'C%' 
AND    cust_year_of_birth > 1960;
```

2.显示查询的执行计划：

```
SQL> SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(FORMAT=>'+ALLSTATS'));
SQL_ID  frgk9dbaftmm9, child number 0
-------------------------------------
SELECT cust_id, cust_last_name, cust_first_name FROM   sh.customers
WHERE  cust_city = 'Hyderabad' AND    cust_income_level LIKE 'C%' AND
 cust_year_of_birth > 1960
 
Plan hash value: 2008213504
 
-------------------------------------------------------------------------------
| Id| Operation         | Name      |Starts|E-Rows|A-Rows|   A-Time   |Buffers|
-------------------------------------------------------------------------------
|  0| SELECT STATEMENT  |           |     1|      |    6 |00:00:00.01 |   1523|
|* 1|  TABLE ACCESS FULL| CUSTOMERS |     1|    6 |    6 |00:00:00.01 |   1523|
-------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter(("CUST_CITY"='Hyderabad' AND "CUST_YEAR_OF_BIRTH">1960 AND
              "CUST_INCOME_LEVEL" LIKE 'C%'))
```

3.在IM列存储中启用 sh.customers 表以进行填充：

```
ALTER TABLE sh.customers INMEMORY;
```

上面的语句使用默认优先级NONE。需要进行完全扫描才能填充没有优先级的对象。

4. 要确定来自  sh.customers 表的数据是否已填充到IM列存储中，请执行以下查询（包括样例输出）：

```
SELECT SEGMENT_NAME, POPULATE_STATUS 
FROM   V$IM_SEGMENTS 
WHERE  SEGMENT_NAME = 'CUSTOMERS';
 
no rows selected
```

在这种情况下，IM列存储中不会填充任何段，因为尚未扫描 sh.customers 表。

5.使用与步骤1中相同的语句查询 sh.customers：

```
SELECT cust_id, cust_last_name, cust_first_name 
FROM   sh.customers 
WHERE  cust_city = 'Hyderabad' 
AND    cust_income_level LIKE 'C%' 
AND    cust_year_of_birth > 1960;
```

6.查询游标显示数据库执行完整扫描并访问IM列存储：

```
SQL> SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(FORMAT=>'+ALLSTATS'));
SQL_ID  frgk9dbaftmm9, child number 0
-------------------------------------
SELECT cust_id, cust_last_name, cust_first_name FROM   sh.customers
WHERE  cust_city = 'Hyderabad' AND    cust_income_level LIKE 'C%' AND
 cust_year_of_birth > 1960
 
Plan hash value: 2008213504
 
---------------------------------------------------------------------------------
| Id| Operation           | Name            |Starts|E-Rows|A-Rows|A-Time|Buffers|
---------------------------------------------------------------------------------
|  0| SELECT STATEMENT           |           |    1|     | 6 |00:00:00.02| 1523 |
|* 1|  TABLE ACCESS INMEMORY FULL| CUSTOMERS |    1|    6| 6 |00:00:00.02| 1523 |
---------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - inmemory(("CUST_CITY"='Hyderabad' AND "CUST_YEAR_OF_BIRTH">1960 AND
              "CUST_INCOME_LEVEL" LIKE 'C%'))
       filter(("CUST_CITY"='Hyderabad' AND "CUST_YEAR_OF_BIRTH">1960 AND
              "CUST_INCOME_LEVEL" LIKE 'C%'))
```

7.再次查询 V$IM_SEGMENTS （包括样例输出）：

```
COL SEGMENT_NAME FORMAT a20
 
SELECT SEGMENT_NAME, POPULATE_STATUS 
FROM   V$IM_SEGMENTS 
WHERE  SEGMENT_NAME = 'CUSTOMERS';
 
SEGMENT_NAME         POPULATE_STATUS
-------------------- ---------------
CUSTOMERS            COMPLETED
```

POPULATE_STATUS 中的值COMPLETED 意味着该表填充在IM列存储中。

DBA_FEATURE_USAGE_STATISTICS 视图确认数据库使用IM列存储检索结果：

```
COL NAME FORMAT a25
SELECT ul.NAME, ul.DETECTED_USAGES 
FROM   DBA_FEATURE_USAGE_STATISTICS ul 
WHERE  ul.VERSION= (SELECT MAX(u2.VERSION) 
                    FROM   DBA_FEATURE_USAGE_STATISTICS u2 
                    WHERE  u2.NAME = ul.NAME 
                    AND    ul.NAME LIKE '%Column Store%');
 
NAME                      DETECTED_USAGES
------------------------- ---------------
In-Memory Column Store    1
```

#### 4.1.2.2 后台进程如何填充IMCU

在填充期间，数据库以其行格式从磁盘读取数据，扭转行以创建列，然后将数据压缩到内存压缩单元（IMCU）。

工作进程（Wnnn）填充IM列存储中的数据。每个工作进程对来自对象的数据库块的子集进行操作。Population是一种流式传输机制，同时压缩数据并将其转换为列式格式。

INMEMORY_MAX_POPULATE_SERVERS 初始化参数指定要用于IM列存储填充的工作进程的最大数目。默认情况下，设置为 CPU_COUNT 的一半。将此参数设置为适合您环境的值。更多的工作进程导致更快的填充，但他们使用更多的CPU资源。较少的工作进程导致较慢的填充，这减少了CPU开销。

```
注:
如果INMEMORY_MAX_POPULATE_SERVERS 设置为 0，则禁用填充。
```

### 4.1.3 In-Memory 填充控制

使用数据定义语言（DDL）语句中的 INMEMORY 子句指定哪些对象适合填充到IM列存储中。您可以启用表空间、表、分区和物化视图。

此部分包含以下主题：

* INMEMORY子句

INMEMORY 是段级属性，而不是列级属性。但是，可以将INMEMORY 属性应用于特定对象中的列子集。

* In-Memory 填充优先级选项

为IM列存储启用数据库对象时，可以启用Oracle数据库以控制在IM列存储中填充对象的时间（默认），或者，您可以指定确定对象在填充队列中的优先级的优先级。

* IM列存储压缩方法

根据您的要求，您可以在不同级别压缩内存中的对象。

* Oracle Compression Advisor

Oracle Compression Advisor 估计您可以使用MEMCOMPRESS子句实现的压缩率。顾问程序使用  DBMS_COMPRESSION 接口。

### 4.1.3.1 INMEMORY 子句

INMEMORY 是段级属性，而不是列级属性。但是，可以将INMEMORY 属性应用于特定对象中的列子集。

要启用或禁用IM列存储的对象，请在以下任何语句中指定 INMEMORY 子句：

```
CREATE TABLESPACE 或ALTER TABLESPACE
```

默认情况下，为IM列存储启用表空间中的所有表和物化视图。表空间中的单个表和物化视图可能具有不同的 INMEMORY属性。单个数据库对象的属性将覆盖表空间的属性。

```
CREATE TABLE 或ALTER TABLE
```

默认情况下，IM列存储填充表中的所有非虚拟列。您可以为表指定全部或部分列。例如，您可以将  oe.product_information 中的  weight_class和 catalog_url 列从合格（eligibility）中排除。对于分区表，可以填充IM列存储中的所有分区或子集的分区。默认情况下，对于分区表，所有表分区都继承 INMEMORY 属性。

```
CREATEMATERIALIZED VIEW 或ALTER MATERIALIZEDVIEW
```

对于分区物化视图，可以填充IM列存储中的所有分区或子集的分区。

DBA_TABLES 视图中的INMEMORY列指示哪些表具有INMEMORY属性集（ENABLED）或未设置（DISABLED）。

以下对象不适用于IM列存储中的填充：

* 索引

* 索引组织表

* Hash集群

* SYS 用户拥有并存储在SYSTEM 或SYSAUX 表空间中的对象

如果为IM列存储启用表，并且它包含以下任何类型的列，则不会在IM列存储中填充这些列：

* 行外列（数组、嵌套表列和行外LOB）

* 使用LONG或LONG RAW数据类型的列

* 扩展数据类型列


*示例4-2将表指定为INMEMORY*

假设您以用户 sh连接到数据库。使用默认压缩级别 FOR QUERY LOW（请参见 “In-Memory压缩”）启用IM列存储中的用于填充的 customers 表：

```
SQL> SELECT TABLE_NAME, INMEMORY FROM USER_TABLES WHERE TABLE_NAME = 'CUSTOMERS';
 
TABLE_NAME INMEMORY
---------- --------
CUSTOMERS  DISABLED
 
SQL> ALTER TABLE customers INMEMORY;
 
Table altered.
 
SQL> SELECT TABLE_NAME, INMEMORY, INMEMORY_COMPRESSION FROM USER_TABLES WHERE TABLE_NAME='CUSTOMERS';
 
TABLE_NAME INMEMORY INMEMORY_COMPRESS
---------- -------- -----------------
CUSTOMERS  ENABLED  FOR QUERY LOW
```

### 4.1.3.2 In-Memory 填充优先级选项

为IM列存储启用数据库对象时，可以启用Oracle数据库以控制在IM列存储中填充对象的时间（默认），或者，您可以指定确定对象在填充队列中的优先级的优先级。

Oracle SQL包括一个 INMEMORY PRIORITY 子句，可以更好地控制队列以进行填充。例如，在填充其他数据库对象的数据之前填充数据库对象的数据可能更重要或更不重要。

下表描述了支持的优先级。

表4-1填充IM列存储中的数据库对象的优先级

|CREATE/ALTER 语法|描述|
|---|---|
|PRIORITY NONE|数据库仅按需填充对象。数据库对象的完全扫描触发对象的填充到IM列存储中。<br>当 PRIORITY 不包括在 INMEMORY 子句中时，这是默认级别。|
|PRIORITY LOW|数据库为对象分配低优先级，并根据其在队列中的位置在启动后填充它。填充不依赖于对象是否被访问。<br>对象将在具有以下优先级别的数据库对象之前的IM列存储中填充： NONE。数据库对象的数据将在具有以下优先级别的数据库对象之后的IM列存储中填充：MEDIUM、HIGH或 CRITICAL。|
|PRIORITY MEDIUM|数据库为对象分配中等优先级，并根据其在队列中的位置在启动后填充它。填充不依赖于对象是否被访问。<br>数据库对象在具有以下优先级别的数据库对象之前的IM列存储中填充：NONE 或LOW。数据库对象的数据在具有以下优先级别的数据库对象之后的IM列存储中填充：HIGH 或CRITICAL。|
|PRIORITY HIGH|数据库为对象分配高优先级，并根据它在队列中的位置在启动后填充它。填充不依赖于对象是否被访问。<br>数据库对象的数据将在具有以下优先级别的数据库对象之前的IM列存储中填充：NONE、LOW或MEDIUM。数据库对象的数据将在具有以下优先级别的数据库对象之后的IM列存储中填充：CRITICAL。|
|PRIORITY CRITICAL|数据库为对象分配低优先级，并根据其在队列中的位置在启动后填充它。填充不依赖于对象是否被访问。<br>数据库对象的数据在具有以下优先级别的数据库对象之前的IM列存储中填充：NONE、LOW、MEDIUM或HIGH。|

当多个数据库对象的优先级等级不是NONE时，Oracle数据库将根据优先级将要填充到IM列存储中的数据库对象的所有数据排队。首先填充具有CRITICAL 优先级的数据库对象; 接下来填充具有HIGH优先级级别的数据库对象，等等。如果IM列存储中没有剩余空间，则不会在其中填充任何其他对象，直到有足够的空间可用。



注:

如果将所有对象指定为CRITICAL，则数据库不会将任何对象视为比任何其他对象更关键。



重新启动数据库时，启动期间将在IM列存储中填充优先级别不为NONE的数据库对象的所有数据。对于优先级为非NONE的数据库对象，在DDL更改记录到IM列存储之前，不会返回涉及数据库对象的 ALTER TABLE 或 ALTERMATERIALIZED VIEWDDL语句。



注:

·      优先级设置必须适用于整个表或表分区。不允许为表中不同的列子集指定不同的IM列存储优先级。

·      如果磁盘上的段为64 KB或更小，则它不会填充到IM列存储中。因此，可能不会填充为IM列存储启用的某些小型数据库对象。


IM列存储压缩方法
根据您的要求，您可以在不同级别压缩内存中的对象。

通常，压缩是一种节省空间的机制。而IM列存储可以压缩数据，并使用一套新的算法提高查询性能。如果使用 FOR DML 或 FOR QUERY 选项压缩列数据，则SQL查询直接对压缩数据执行。因此，扫描和过滤操作在小得多的数据量上执行。数据库仅在结果集需要数据时才解压缩数据。



V$IM_SEGMENTS 和 V$IM_COLUMN_LEVEL 视图指示当前的压缩级别。您可以使用相应的ALTER命令更改压缩级别。如果当前在IM列存储中填充了表，并且如果更改了 PRIORITY之外的表的任何 INMEMORY 属性，则数据库会从IM列存储中逐出该表。重新填充行为取决于 INMEMORY 设置。



下表总结了IM列存储中支持的数据压缩方法。

表4-2 IM列存储压缩方法

CREATE/ALTER 语法

描述

NO MEMCOMPRESS

数据不进行压缩。

MEMCOMPRESS FOR DML

此方法产生最佳的DML性能。

除了NO MEMCOMPRESS外，此方法压缩IM列存储数据最少。

MEMCOMPRESS FOR QUERY LOW

此方法产生最佳的查询性能。

此方法会压缩IM列存储数据超过 MEMCOMPRESS FOR DML，但小于 MEMCOMPRESS FOR QUERY HIGH。

当在CREATE 或ALTER  SQL语句中指定INMEMORY 子句而不使用压缩方法时，或者指定MEMCOMPRESS FOR  QUERY 而不包括LOW 或HIGH时，此方法是缺省方法。

MEMCOMPRESS FOR QUERY HIGH

此方法产生良好的查询性能，并节省空间。

此方法压缩IM列存储数据超过MEMCOMPRESS FOR  QUERY LOW，但小于 MEMCOMPRESS FOR CAPACITY LOW。

MEMCOMPRESS FOR CAPACITY LOW

此方法平衡了空间节省和查询性能，偏向于节省空间。

此方法会压缩IM列存储数据超过MEMCOMPRESS FOR  QUERY HIGH，但小于 MEMCOMPRESS FOR CAPACITY HIGH。此方法使用称为Oracle Zip（OZIP）的专有压缩技术，可提供专为Oracle数据库专门调整的极快速解压缩。该数据必须解压缩才能扫描。

当指定MEMCOMPRESS FOR  CAPACITY而不包括LOW 或HIGH时，此方法是默认值。

MEMCOMPRESS FOR CAPACITY HIGH

这种方法导致最佳的空间节省。

此方法最多压缩IM列存储数据。

在SQL语句中，MEMCOMPRESS 关键字必须在INMEMORY 关键字前面。



Oracle Compression Advisor

Oracle Compression Advisor估计您可以使用MEMCOMPRESS 子句实现的压缩率。顾问程序使用DBMS_COMPRESSION接口。

对表运行DBMS_COMPRESSION.GET_COMPRESSION_RATIO时，Oracle数据库会分析一个示例行。因此，Oracle压缩顾问提供了一个表，在填充到IM列存储后，表实现的压缩结果的良好估计。
