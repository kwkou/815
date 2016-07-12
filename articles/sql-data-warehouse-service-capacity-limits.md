<properties
   pageTitle="SQL 数据仓库容量限制 | Azure"
   description="SQL 数据仓库的连接、数据库、表和查询的最大值。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/13/2016"
   wacn.date="06/27/2016"/>

# SQL 数据仓库容量限制

下表包含 Azure SQL 数据仓库的各个组件允许的最大值。


## 工作负荷管理

| 类别 | 说明 | 最大值 |
| :------------------ | :------------------------------------------- | :----------------- |
| 数据仓库单位 (DWU)| 计算、内存和 IO 资源 | 2000 |
| 数据库连接 | 并发打开的会话 | 1024<br/><br/>我们支持最多 1024 个活动连接，每个活动连接可同时将请求提交到 SQL 数据仓库数据库。请注意，实际可并发执行的查询数量是有限制的。当超出并发限制时，请求将进入内部队列等待处理。|
| 数据库连接 | 预处理语句的最大内存 | 20 MB |
| 工作负荷管理 | 并发查询数上限 | 32<br/><br/>默认情况下，SQL 数据仓库可执行最多 32 个并发查询，剩余的查询将排队。<br/><br/>当为用户分配了更高的资源类时，并发级别可能会降低。某些查询（如 DMV 查询）始终可以运行。有关详细信息，请参阅 [Concurrency and workload management（并发性和工作负荷管理）][]。|


## 数据库对象

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| 数据库 | 最大大小 | 磁盘上压缩后 60 TB<br/><br/>SQL 数据仓库允许每个数据库的磁盘上最多有 60 TB 的原始空间。磁盘上的空间是永久表的压缩大小。此空间与 tempdb 或日志空间无关，因此，此空间专用于永久表。聚集列存储压缩估计为 5 倍，这意味着当所有表都是聚集列存储（默认的表类型）时，未压缩的数据库大小可能增长到大约 300 TB。在公共预览期结束时，60 TB 限制将增加到 240 TB，这样，大部分数据库应可增长到超过 1 PB 的未压缩数据。|
| 表 | 最大大小 | 磁盘上压缩后 60 TB |
| 表 | 每个数据库的表数 | 20 亿 |
| 表 | 每个表的列数 | 1024 个列 |
| 表 | 每个列的字节数 | 8000 字节 |
| 表 | 每行的字节数，定义的大小 | 8060 字节<br/><br/>每行字节数的计算方式同于已启用页面压缩的 SQL Server。与 SQL Server 一样，SQL 数据仓库支持行溢出存储，使可变长度列能够脱行推送。主记录中只存储脱行推送行的可变长度列的 24 字节根。有关详细信息，请参阅 SQL Server 联机丛书中的 [Row-Overflow Data Exceeding 8 KB（超过 8 KB 的行溢出数据）][]主题。<br/><br/>有关 SQL 数据仓库数据类型大小的列表，请参阅 [CREATE TABLE (Azure SQL Data Warehouse)（CREATE TABLE（Azure SQL 数据仓库））][]。 |
| 表 | 每个表的分区数 | 15,000<br/><br/>为了实现高性能，我们建议在满足你的业务需求的情况下尽量减少所需的分区数。随着分区数目的增长，数据定义语言 (DDL) 和数据操作语言 (DML) 操作的开销也会增长，导致性能下降。|
| 表 | 每个分区边界值的字符数。| 4000 |
| 索引 | 每个表的非聚集索引数。 | 999<br/><br/>仅适用于行存储表。|
| 索引 | 每个表的聚集索引数。 | 1<br><br/>适用于行存储表和列存储表。|
| 索引 | 列存储索引行组中的行数 | 1,024<br/><br/>将每个列存储索引实现为多个列存储索引。请注意，如果在 SQL 数据仓库列存储索引中插入 1,024 行，那么这些行并不都在同一行组。|
| 索引 | 聚集列存储索引的并发生成数。 | 32<br/><br/>当所有聚集列存储索引均在不同表中生成时适用。每个表只允许一个聚集列存储索引生成。其他请求将在队列中等待。|
| 索引 | 索引键大小。 | 900 字节。<br/><br/>仅适用于行存储索引。<br/><br/>如果创建索引时列中的现有数据未超过 900 字节，那么可以创建最大大小超过 900 字节的 varchar 列上的索引。但是，以后导致总大小超过 900 字节的对列的 INSERT 或 UPDATE 操作将失败。|
| 索引 | 每个索引的键列数。 | 16<br/><br/>仅适用于行存储索引。聚集列存储索引包括所有列。|
| 统计信息 | 组合的列值的大小。 | 900 字节。 |
| 统计信息 | 每个统计对象的列数。 | 32 |
| 统计信息 | 每个表的列上创建的统计信息条数。 | 30,000 |
| 存储过程 | 最大嵌套级数。 | 8 |
| 查看 | 每个视图的列数 | 1,024 |


## 加载

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| Polybase 加载 | 每行的字节数 | 32768<br/><br/>Polybase 加载限制为加载小于 32k 的行，并且无法加载到 VARCHR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX)。虽然此限制目前仍存在，但将很快去除。<br/><br/>


## 查询

| 类别 | 说明 | 最大值 |
| :---------------- | :------------------------------------------- | :----------------- |
| 查询 | 用户表的排队查询数。 | 1000 |
| 查询 | 系统视图的并发查询数。 | 100 |
| 查询 | 系统视图的排队查询数。 | 1000 |
| 查询 | 最大值参数 | 2098 |
| 批处理 | 最大大小 | 65,536*4096 |
| SELECT 结果 | 每个行的列数 | 4096<br/><br/>在 SELECT 结果中每行的列数始终不得超过 4096。无法保证最大值始终为 4096。如果查询计划需要一个临时表，那么将应用每个表最多 1024 列的最大值。|
| SELECT | 嵌套子查询 | 32<br/><br/>在 SELECT 语句中的嵌套子查询数始终不得超过 32 个。无法保证最大值始终为 32 个。例如，JOIN 可以将子查询引入查询计划。还可以通过可用内存来限制子查询的数量。|
| SELECT | 每个 JOIN 的列数 | 1024 列<br/><br/>JOIN 中的列数始终不得超过 1024。无法保证最大值始终为 1024。如果 JOIN 计划需要列数多于 JOIN 结果的临时表，那么将 1024 限制应用于此临时表。 |
| SELECT | 每个 GROUP BY 列的字节数。 | 8060<br/><br/>GROUP BY 子句中的列的字节数最大为 8060 字节。|
| SELECT | 每个 ORDER BY 列的字节数 | 8060 字节。<br/><br/>ORDER BY 子句中的列的字节数最大为 8060 字节。|
| 每个语句的标识符和常量数 | 被引用的标识符和常量的数量。 | 65,535<br/><br/>SQL 数据仓库限制一条查询的单个表达式中可包含的标识符和常量数。此限制为 65,535。超过此数字将导致 SQL Server 错误 8632。有关详细信息，请参阅 [Internal error: An expression services limit has been reached（内部错误：已达到表达式服务限制）][]。|


## Metadata

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


## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库参考概述]: /documentation/articles/sql-data-warehouse-overview-reference/
[Concurrency and workload management（并发性和工作负荷管理）]: /documentation/articles/sql-data-warehouse-develop-concurrency/

<!--MSDN references-->
[Row-Overflow Data Exceeding 8 KB（超过 8 KB 的行溢出数据）]: https://msdn.microsoft.com/zh-cn/library/ms186981.aspx
[CREATE TABLE (Azure SQL Data Warehouse)（CREATE TABLE（Azure SQL 数据仓库））]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[Internal error: An expression services limit has been reached（内部错误：已达到表达式服务限制）]: https://support.microsoft.com/kb/913050

<!---HONumber=Mooncake_0620_2016-->