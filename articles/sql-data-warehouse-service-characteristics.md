<properties
   pageTitle="SQL 数据仓库服务特征 | Azure"
   description="SQL 数据仓库的连接、查询、Transact-SQL DDL 和 DML，以及系统视图的最大值。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/03/2016"
   wacn.date="04/25/2016"/>

# SQL 数据仓库容量限制

用于支持大部分所需的分析工作负荷，同时确保每个单独查询具有获取最佳性能所需的资源的最大值。

## 连接

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| 会话 | 并发打开的会话 | 1024<br/><br/>我们支持最多 1024 个活动连接，它们可同时将请求提交到数据仓库。请注意，实际可并发执行的查询数量是有限制的。当超出限制时，请求进入内部队列等待处理。|
| 会话 | 预处理语句的最大内存 | 20 MB |


## 查询处理

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| 查询 | 用户表的并发查询数。 | 32<br/><br/>这是并发用户查询执行的上限。其他查询将转到内部队列等待处理。无论同时执行的查询数量有多少，都会优化每个查询来充分利用大规模并行处理的体系结构。注意：实际并发可能根据数据库实例的 DWU 及运行中查询所选的资源类型而受限于其他限制。|
| 查询 | 用户表的排队查询数。 | 1000 |
| 查询 | 系统视图的并发查询数。 | 100 |
| 查询 | 系统视图的排队查询数。 | 1000 |
| 查询 | 最大值参数 | 2098 |
| 批处理 | 最大大小 | 65,536*4096 |


## 数据定义语言 (DDL)

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| 表 | 每个数据库的表数 | 20 亿 |
| 表 | 每个表的列数 | 1024 个列 |
| 表 | 每个列的字节数 | 8000 字节 |
| 表 | 每行的字节数，定义的大小 | 8060 字节<br/><br/>每行的字节数计算方式同于已启用页面压缩的 SQL Server。与 SQL Server 一样，SQL 数据仓库支持行溢出存储，使可变长度列能够脱行推送。主记录中只存储脱行推送行的可变长度列的 24 字节根。有关详细信息，请参阅 SQL Server 联机丛书中的 [Row-Overflow Data Exceeding 8 KB（超过 8 KB 的行溢出数据）](https://msdn.microsoft.com/zh-cn/library/ms186981.aspx)主题。<br/><br/>有关 SQL 数据仓库数据类型大小的列表，请参阅 [CREATE TABLE (Azure SQL Data Warehouse)（CREATE TABLE（Azure SQL 数据仓库））](https://msdn.microsoft.com/zh-cn/library/mt203953.aspx)。 |
| 表 | 每行的字节数，移动数据时的内部缓冲区大小 | 32,768<br/><br/>注意：目前有此限制，但即将删除。<br/><br/>在分布式 SQL 数据仓库系统中，SQL 数据仓库使用内部缓冲区来移动行。负责移动行的服务称为数据移动服务 (DMS)，它以不同于 SQL Server 的格式存储行。<br/><br/>如果行无法填入内部缓冲区，将发生查询编译错误或内部数据移动错误。若要避免此问题，请参阅[有关 DMS 缓冲区大小的详细信息](#details-about-the-dms-buffer-size)。|
| 表 | 每个表的分区数 | 15,000<br/><br/>为了实现高性能，我们建议在满足你的业务需求的情况下尽量减少所需的分区数。随着分区数目的增长，数据定义语言 (DDL) 和数据操作语言 (DML) 操作的开销也会增长，导致性能下降。|
| 表 | 每个分区边界值的字符数。| 4000 |
| 索引 | 每个表的非聚集索引数。 | 999<br/><br/>仅适用于行存储表。|
| 索引 | 每个表的聚集索引数。 | 1<br><br/>适用于行存储和列存储表。|
| 索引 | 列存储索引行组中的行数 | 1,024<br/><br/>将每个列存储索引实现为多个列存储索引。请注意，如果在 SQL 数据仓库列存储索引中插入 1,024 行，那么这些行并不都在同一行组。|
| 索引 | 聚集列存储索引的并发生成数。 | 32<br/><br/>当所有聚集列存储索引均在不同表中生成时适用。每个表只允许一个聚集列存储索引生成。其他请求将在队列中等待。|
| 索引 | 索引键大小。 | 900 字节。<br/><br/>仅适用于行存储索引。<br/><br/>如果创建索引时列中的现有数据未超过 900 字节，那么可以创建最大值超过 900 字节的 varchar 列上的索引。但是，以后导致总大小超过 900 字节的对列的 INSERT 或 UPDATE 操作将失败。|
| 索引 | 每个索引的键列数。 | 16<br/><br/>仅适用于行存储索引。聚集列存储索引包括所有列。|
| 统计信息 | 组合的列值的大小。 | 900 字节。 |
| 统计信息 | 每个统计对象的列数。 | 32 |
| 统计信息 | 每个表的列上创建的统计信息条数。 | 30,000 |
| 存储过程 | 最大嵌套级数。 | 8 |
| 查看 | 每个视图的列数 | 1,024 |


## 数据操作语言 (DML)

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| SELECT 结果 | 每个行的列数 | 4096<br/><br/>在 SELECT 结果中每行的列数始终不得超过 4096。无法保证最大值始终为 4096。如果查询计划需要一个临时表，那么将应用每个表最多 1024 列的最大值。|
| SELECT | 嵌套子查询 | 32<br/><br/>在 SELECT 语句中的嵌套子查询数始终不得超过 32 个。无法保证最大值始终为 32 个。例如，JOIN 可以将子查询引入查询计划。还可以通过可用内存来限制子查询的数量。|
| SELECT | 每个 JOIN 的列数 | 1024 列<br/><br/>JOIN 中的列数始终不得超过 1024。无法保证最大值始终为 1024。如果 JOIN 计划需要列数多于 JOIN 结果的临时表，那么将 1024 限制应用于此临时表。 |
| SELECT | 每个 GROUP BY 列的字节数。 | 8060<br/><br/>GROUP BY 子句中的列的字节数最大为 8060 字节。|
| SELECT | 每个 ORDER BY 列的字节数 | 8060 字节。<br/><br/>ORDER BY 子句中的列的字节数最大为 8060 字节。|
| 每个语句的标识符和常量数 | 被引用的标识符和常量的数量。 | 65,535<br/><br/>SQL 数据仓库限制一条查询的单个表达式中可包含的标识符和常量数。此限制为 65,535。超过此数字将导致 SQL Server 错误 8632。有关详细信息，请参阅 [Internal error: An expression services limit has been reached（内部错误：已达到表达式服务限制）](http://support.microsoft.com/kb/913050/)。|

## 系统视图

| 系统视图 | 最大行数 |
| :--------------------------------- | :------------|
| sys.dm\_pdw\_component\_health\_alerts | 10,000 |
| sys.dm\_pdw\_dms\_cores | 100 |
| sys.dm\_pdw\_dms\_workers | 最近 1000 个 SQL 请求的 DMS 辅助角色的总数。 |
| sys.dm\_pdw\_errors | 10,000 |
| sys.dm\_pdw\_exec\_requests | 10,000 |
| sys.dm\_pdw\_exec\_sessions | 10,000 |
| sys.dm\_pdw\_request\_steps | sys.dm\_pdw\_exec\_requests 中存储的最近 1000 个 SQL 请求的步骤总数。 |
| sys.dm\_pdw\_os\_event\_logs | 10,000 |
| sys.dm\_pdw\_sql\_requests | sys.dm\_pdw\_exec\_requests 中存储的最近 1000 个 SQL 请求。 |


## 有关 DMS 缓冲区大小的详细信息

SQL 数据仓库使用内部缓冲区在后端计算节点之间移动行。负责移动行的服务称为数据移动服务 (DMS)，它并使用不同于 SQL Server 的存储格式。

为了改善并行查询性能，DMS 将所有可变长度数据填补到 SQL 数据库定义的大小上限。例如，`nvarchar(2000) NOT NULL` 的值 'hello' 在 DMS 缓冲区中实际使用 4002 个字节。2000 个字符的每个字符使用 2 个字节，再加上 NULL 终止符的 2 个字节。

> [AZURE.NOTE] 当 DMS 尝试移动的行超过 32,768 个字节的 DMS 缓冲区大小时，将发生内部错误。如果行大小超过 DMS 缓冲区大小，你需要修改表定义，使行可放入 DMS 缓冲区。

### 如何确定 DMS 缓冲区的行大小
此示例说明 DMS 定义的可变长度数据大小，然后演示如何计算行的 DMS 缓冲区大小。
 
在 DMS 缓冲区中，可变长度的数据大小是以下各项的总和：

- 定义的字符大小。
- NULL 类型使用 8 个字节作为 NULL 指示符。
- ASCII 类型使用 1 个字节作为 NULL 终止符。
- Unicode 类型使用 2 个字节作为 NULL 终止符。
             
| 数据类型 | DMS 缓冲区大小 |
| :---------------------- | :-------------------------- |                
| char(1000) NULL | 1009 字节 = 1000 +8 + 1 |
| char(1000) NOT NULL | 1001 字节 = 1000 + 1 |
| nchar(1000 NULL | 2010 字节 = 2000 + 8 + 2 |
| varchar(1000) NULL | 1009 字节 = 1000 + 8 + 1 |
| varchar(1000) NOT NULL | 1009 字节 = 1000 + 1 |
| nvarchar(1000) NULL | 2010 字节 = 2000 + 8 + 2 |
| nvarchar(1000) NOT NULL | 2010 字节 = 2000 + 2 |
                
在 DMS 缓冲区中，固定宽度列使用 SQL Server 本机大小。如果类型可为 null，DMS 需要额外的 8 字节。关于 SQL Server 大小，请参阅 sys.types 中的 max\_length 字段。

例如：

| 固定宽度数据类型 | DMS 缓冲区大小 |
| :-------------------- | :----------------- |
| int NULL | 12 字节 = 4 + 8 |
| int NOT NULL | 4 字节 = 4 + 0 | 
                
以下行的 DMS 定义缓冲区大小合计为 31,134 字节，可放入 DMS 缓冲区。

| 列 | 数据类型 | 列大小 |
| :----- | :------------------ | :------------------------ |
| col1 | datetime2 (7) NULL | 20 字节 = 8 + 8 |
| col2 | float (4) NULL | 12 字节 = 4 + 8 |
| col3 | nchar (6) NULL | 22 字节 = 12 + 8 + 2 |
| col4 | char (7000) NULL | 7009 字节 = 7000 + 8 + 1 |
| col5 | nvarchar (4000) | 8002 字节 = 8000 + 2. |
| col6 | varchar (8000) NULL | 8009 字节 = 8000 + 8 + 1 |
| col7 | varbinary (8000) | 8000 字节 |
| col8 | binary (60) | 60 字节 |
                
              
### 超出 DMS 缓冲区大小的示例

此示例演示如何成功将行插入 SQL 数据仓库，但之后当 DMS 由于与分布区不兼容的联接而需要移动行时，将导致 DMS 溢出错误。学到的经验是创建可放入 DMS 缓冲区的较小行。

在以下示例中，我们将创建表 T1。当所有 nvarchar 变量都有 4000 个 Unicode 字符时，最大可能的行大小超过 40,000 字节，这已超过 DMS 缓冲区大小上限。

由于 nvarchar 实际定义的大小使用 26 字节，而行定义小于 8060 字节，因此可放在 SQL Server 页中。因此，即使当 DMS 尝试将此行加载到 DMS 缓冲区时失败，CREATE TABLE 语句也仍可成功。

````
CREATE TABLE T1
  (
    c0 int NOT NULL,
    CustomerKey int NOT NULL,
    c1 nvarchar(4000),
    c2 nvarchar(4000),
    c3 nvarchar(4000),
    c4 nvarchar(4000),
    c5 nvarchar(4000)
  )
WITH ( DISTRIBUTION = HASH (c0) )
;
````
下一个步骤演示我们可以成功地使用 INSERT 将数据插入表中。此语句不使用 DMS，而是直接将数据加载到 SQL Server，因此不会引发 DMS 缓冲区溢出失败。集成服务也会成功加载此行。</para>

````
--The INSERT operation succeeds because the row is inserted directly into SQL Server without requiring DMS to buffer the row.
INSERT INTO T1
VALUES (
    25,
    429817,
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.'
  )
````

为了准备演示数据移动，此示例创建了第二个表，其中的 CustomerKey 用作分布列。

````
--This second table is distributed on CustomerKey. 
CREATE TABLE T2
  (
    c0 int NOT NULL,
    CustomerKey int NOT NULL,
    c1 nvarchar(4000),
    c2 nvarchar(4000),
    c3 nvarchar(4000),
    c4 nvarchar(4000),
    c5 nvarchar(4000)
  )
WITH ( DISTRIBUTION = HASH (CustomerKey) )
;

INSERT INTO T2
VALUES (
    32,
    429817,
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.',
    N'Each row must fit into the DMS buffer size of 32,768 bytes.'
  )
````
由于这两个表不是根据 CustomerKey 分布的，T1 和 T2 之间根据 CustomerKey 的联接与分布区不兼容。DMS 需要加载至少一个行，并将它复制到不同的分布区。

````
SELECT * FROM T1 JOIN T2 ON T1.CustomerKey = T2.CustomerKey;
````

如同预期，DMS 无法执行该联接，因为当填补 nvarchar 列时，行超过 32,768 字节的 DMS 缓冲区大小。将出现以下错误消息。

````
Msg 110802, Level 16, State 1, Line 126

An internal DMS error occurred that caused this operation to fail. Details: Exception: Microsoft.SqlServer.DataWarehouse.DataMovement.Workers.DmsSqlNativeException, Message: SqlNativeBufferReader.ReadBuffer, error in OdbcReadBuffer: SqlState: , NativeError: 0, 'COdbcReadConnection::ReadBuffer: not enough buffer space for one row | Error calling: pReadConn-&gt;ReadBuffer(pBuffer, bufferOffset, bufferLength, pBytesRead, pRowsRead) | state: FFFF, number: 81, active connections: 8', Connection String: Driver={SQL Server Native Client 11.0};APP=DmsNativeReader:P13521-CMP02\sqldwdms (4556) - ODBC;Trusted_Connection=yes;AutoTranslate=no;Server=P13521-SQLCMP02,1500
````


## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库参考概述]: /documentation/articles/sql-data-warehouse-overview-reference/

<!--MSDN references-->

<!---HONumber=Mooncake_0418_2016-->