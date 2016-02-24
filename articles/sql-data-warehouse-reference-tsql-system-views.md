<properties
   pageTitle="SQL 数据仓库系统视图 | Microsoft Azure"
   description="SQL 数据仓库的系统视图内容链接。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="09/22/2015"
   wacn.date="01/20/2016"/>

# 系统视图

## SQL 数据仓库目录视图

- [sys.pdw\_column\_distribution\_properties](http://msdn.microsoft.com/zh-cn/library/mt204022.aspx)
- [sys.pdw\_database\_mappings](http://msdn.microsoft.com/zh-cn/library/mt203891.aspx)
- [sys.pdw\_diag\_event\_properties](http://msdn.microsoft.com/zh-cn/library/mt203921.aspx)
- [sys.pdw\_diag\_events](http://msdn.microsoft.com/zh-cn/library/mt203895.aspx)
- [sys.pdw\_diag\_sessions](http://msdn.microsoft.com/zh-cn/library/mt203890.aspx)
- [sys.pdw\_distributions](http://msdn.microsoft.com/zh-cn/library/mt203892.aspx)
- [sys.pdw\_index\_mappings](http://msdn.microsoft.com/zh-cn/library/mt203912.aspx)
- [sys.pdw\_loader\_backup\_run\_details](http://msdn.microsoft.com/zh-cn/library/mt203877.aspx)
- [sys.pdw\_loader\_backup\_runs](http://msdn.microsoft.com/zh-cn/library/mt203884.aspx)
- [sys.pdw\_loader\_run\_stages](http://msdn.microsoft.com/zh-cn/library/mt203879.aspx)
- [sys.pdw\_nodes\_column\_store\_dictionaries](http://msdn.microsoft.com/zh-cn/library/mt203902.aspx)
- [sys.pdw\_nodes\_column\_store\_row\_groups](http://msdn.microsoft.com/zh-cn/library/mt203880.aspx)
- [sys.pdw\_nodes\_column\_store\_segments](http://msdn.microsoft.com/zh-cn/library/mt203916.aspx)
- [sys.pdw\_nodes\_columns](http://msdn.microsoft.com/zh-cn/library/mt203881.aspx)
- [sys.pdw\_nodes\_indexes](http://msdn.microsoft.com/zh-cn/library/mt203885.aspx)
- [sys.pdw\_nodes\_partitions](http://msdn.microsoft.com/zh-cn/library/mt203908.aspx)
- [sys.pdw\_nodes\_pdw\_physical\_databases](http://msdn.microsoft.com/zh-cn/library/mt203897.aspx)
- [sys.pdw\_nodes\_tables](http://msdn.microsoft.com/zh-cn/library/mt203886.aspx)
- [sys.pdw\_table\_distribution\_properties](http://msdn.microsoft.com/zh-cn/library/mt203896.aspx)
- [sys.pdw\_table\_mappings](http://msdn.microsoft.com/zh-cn/library/mt203876.aspx)

## SQL 数据库目录视图

- [dbo.server\_quotas（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn308512.aspx)
- [sys.bandwidth\_usage（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn269985.aspx)
- [sys.database\_connection\_stats（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn269986.aspx)
- [sys.database\_firewall\_rules（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn269982.aspx)
- [sys.database\_usage（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/azure/ff951626.aspx)
- [sys.event\_log（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn270018.aspx)
- [sys.database\_firewall\_rules（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn269982.aspx)
- [sys.firewall\_rules（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/azure/ff951627.aspx)
- [sys.resource\_stats（Azure SQL 数据库）](http://msdn.microsoft.com/zh-cn/library/dn269979.aspx)
- sys.resource\_usage（Azure SQL 数据库）


## SQL 数据仓库动态管理视图 (DMV)

- [sys.dm\_pdw\_diag\_processing\_stats](http://msdn.microsoft.com/zh-cn/library/mt203914.aspx)
- [sys.dm\_pdw\_dms\_cores](http://msdn.microsoft.com/zh-cn/library/mt203911.aspx)
- [sys.dm\_pdw\_dms\_workers](http://msdn.microsoft.com/zh-cn/library/mt203878.aspx)
- [sys.dm\_pdw\_errors](http://msdn.microsoft.com/zh-cn/library/mt203904.aspx)
- [sys.dm\_pdw\_exec\_connections](http://msdn.microsoft.com/zh-cn/library/mt203882.aspx)
- [sys.dm\_pdw\_exec\_requests](http://msdn.microsoft.com/zh-cn/library/mt203887.aspx)
- [sys.dm\_pdw\_exec\_sessions](http://msdn.microsoft.com/zh-cn/library/mt203883.aspx)
- [sys.dm\_pdw\_network\_credentials](http://msdn.microsoft.com/zh-cn/library/mt203915.aspx)
- [sys.dm\_pdw\_nodes](http://msdn.microsoft.com/zh-cn/library/mt203907.aspx)
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys](http://msdn.microsoft.com/zh-cn/library/mt203922.aspx)
- [sys.dm\_pdw\_node\_status](http://msdn.microsoft.com/zh-cn/library/mt203905.aspx)
- [sys.dm\_pdw\_os\_event\_logs](http://msdn.microsoft.com/zh-cn/library/mt203910.aspx)
- [sys.dm\_pdw\_or\_performance\_counters](http://msdn.microsoft.com/zh-cn/library/mt203875.aspx)
- [sys.dm\_pdw\_os\_threads](http://msdn.microsoft.com/zh-cn/library/mt203917.aspx)
- [sys.dm\_pdw\_query\_stats\_xe](http://msdn.microsoft.com/zh-cn/library/mt203898.aspx)
- [sys.dm\_pdw\_query\_stats\_xe\_file](http://msdn.microsoft.com/zh-cn/library/mt203919.aspx)
- [sys.dm\_pdw\_request\_steps](http://msdn.microsoft.com/zh-cn/library/mt203913.aspx)
- [sys.dm\_pdw\_resource\_waits](http://msdn.microsoft.com/zh-cn/library/mt203906.aspx)
- [sys.dm\_pdw\_sql\_requests](http://msdn.microsoft.com/zh-cn/library/mt203889.aspx)
- [sys.dm\_pdw\_sys\_info](http://msdn.microsoft.com/zh-cn/library/mt203900.aspx)
- [sys.dm\_pdw\_wait\_stats](http://msdn.microsoft.com/zh-cn/library/mt203909.aspx)
- [sys.dm\_pdw\_waits](http://msdn.microsoft.com/zh-cn/library/mt203909.aspx)
- [sys.dm\_pdw\_lock\_waits](http://msdn.microsoft.com/zh-cn/library/mt203901.aspx)

## SQL Server 目录视图

- [sys.all\_columns](http://msdn.microsoft.com/zh-cn/library/ms177522.aspx)
- [sys.all\_objects](http://msdn.microsoft.com/zh-cn/library/ms178618.aspx)
- [sys.all\_sql\_modules](http://msdn.microsoft.com/zh-cn/library/ms184389.aspx)
- [sys.all\_views](http://msdn.microsoft.com/zh-cn/library/ms189510.aspx)
- [sys.assemblies](http://msdn.microsoft.com/zh-cn/library/ms189790.aspx)
- [sys.assembly\_modules](http://msdn.microsoft.com/zh-cn/library/ms180052.aspx)
- [sys.assembly\_types](http://msdn.microsoft.com/zh-cn/library/ms178020.aspx)
- [sys.check\_constraints](http://msdn.microsoft.com/zh-cn/library/ms187388.aspx)
- [sys.certificates](http://msdn.microsoft.com/zh-cn/library/ms189774.aspx)
- [sys.columns](http://msdn.microsoft.com/zh-cn/library/ms176106.aspx)
- [sys.computed\_columns](http://msdn.microsoft.com/zh-cn/library/ms188744.aspx)
- [sys.data\_spaces](http://msdn.microsoft.com/zh-cn/library/ms190289.aspx)
- [sys.database\_files](http://msdn.microsoft.com/zh-cn/library/ms174397.aspx)
- [sys.database\_permissions](http://msdn.microsoft.com/zh-cn/library/ms188367.aspx)
- [sys.database\_principals](http://msdn.microsoft.com/zh-cn/library/ms187328.aspx)
- [sys.database\_role\_members](http://msdn.microsoft.com/zh-cn/library/ms189780.aspx)
- [sys.databases](http://msdn.microsoft.com/zh-cn/library/ms178534.aspx)
- [sys.default\_constraints](http://msdn.microsoft.com/zh-cn/library/ms173758.aspx)
- [sys.extended\_properties](http://msdn.microsoft.com/zh-cn/library/ms177541.aspx)
- [sys.external\_file formats](http://msdn.microsoft.com/zh-cn/library/dn935025.aspx)
- [sys.external\_tables](http://msdn.microsoft.com/zh-cn/library/dn935029.aspx)
- [sys.external\_data\_sources](http://msdn.microsoft.com/zh-cn/library/dn935019.aspx)
- [sys.filegroups](http://msdn.microsoft.com/zh-cn/library/ms187782.aspx)
- [sys.foreign\_key\_columns](http://msdn.microsoft.com/zh-cn/library/ms186306.aspx)
- [sys.identity\_columns](http://msdn.microsoft.com/zh-cn/library/ms187334.aspx)
- [sys.index\_columns](http://msdn.microsoft.com/zh-cn/library/ms175105.aspx)
- [sys.indexes](http://msdn.microsoft.com/zh-cn/library/ms173760.aspx)
- [sys.key\_constraints](http://msdn.microsoft.com/zh-cn/library/ms174321.aspx)
- [sys.master\_files](http://msdn.microsoft.com/zh-cn/library/ms186782.aspx)
- [sys.numbered\_procedures](http://msdn.microsoft.com/zh-cn/library/ms179865.aspx)
- [sys.objects](http://msdn.microsoft.com/zh-cn/library/ms190324.aspx)
- [sys.partition\_functions](http://msdn.microsoft.com/zh-cn/library/ms187381.aspx)
- [sys.partition\_parameters](http://msdn.microsoft.com/zh-cn/library/ms175054.aspx)
- [sys.partition\_range\_values](http://msdn.microsoft.com/zh-cn/library/ms187780.aspx)
- [sys.partition\_schemes](http://msdn.microsoft.com/zh-cn/library/ms189752.aspx)
- [sys.partitions](http://msdn.microsoft.com/zh-cn/library/ms175012.aspx)
- [sys.parameters](http://msdn.microsoft.com/zh-cn/library/ms176074.aspx)
- [sys.procedures](http://msdn.microsoft.com/zh-cn/library/ms188737.aspx)
- [sys.schemas](http://msdn.microsoft.com/zh-cn/library/ms176011.aspx)
- [sys.securable\_classes](http://msdn.microsoft.com/zh-cn/library/ms408301.aspx)
- [sys.server\_role\_members](http://msdn.microsoft.com/zh-cn/library/ms190331.aspx)
- [sys.server\_permissions](http://msdn.microsoft.com/zh-cn/library/ms186260.aspx)
- [sys.server\_principals](http://msdn.microsoft.com/zh-cn/library/ms188786.aspx)
- [sys.sql\_expression\_dependencies](http://msdn.microsoft.com/zh-cn/library/bb677315.aspx)
- [sys.sql\_logins](http://msdn.microsoft.com/ms174355.aspx)
- [sys.sql\_modules](http://msdn.microsoft.com/zh-cn/library/ms175081.aspx)
- [sys.stats](http://msdn.microsoft.com/zh-cn/library/ms177623.aspx)
- [sys.stats\_columns](http://msdn.microsoft.com/zh-cn/library/ms187340.aspx)
- [sys.system\_columns](http://msdn.microsoft.com/zh-cn/library/ms178596.aspx****)
- [sys.system\_objects](http://msdn.microsoft.com/zh-cn/library/ms173551.aspx)
- [sys.system\_sql\_modules](http://msdn.microsoft.com/zh-cn/library/ms188034.aspx)
- [sys.system\_views](http://msdn.microsoft.com/zh-cn/library/ms187764.aspx)
- [sys.tables](http://msdn.microsoft.com/zh-cn/library/ms187406.aspx)
- [sys.types](http://msdn.microsoft.com/zh-cn/library/ms188021.aspx)
- [sys.views](http://msdn.microsoft.com/zh-cn/library/ms190334.aspx)

## SQL 数据仓库中提供的 SQL Server DMV 列表

SQL 数据仓库公开许多 SQL Server 动态管理视图 (DMV)。在 SQL 数据仓库中查询这些视图时，它们将报告分布区上运行的 SQL 数据库的状态。

由于 SQL 数据仓库采用 Microsoft 的 MPP 技术，因此 SQL 数据仓库和分析平台系统的并行数据仓库 (PDW) 都使用相同的系统视图。

这就是为何其中每个 DMV 都有一个名为 pdw\_node\_id 的特定列的原因。这是计算节点的标识符。在 PDW 中，计算节点是体系结构的强势概念。在 SQL 数据仓库中，体系结构更严重依赖于分布区。

>[AZURE.NOTE]若要使用这些视图，请在名称中插入“pdw\_nodes\_”，如下表所示。


| SQL 数据仓库中的 DMV 名称 | MSDN 上的 SQL Server Transact-SQL 主题链接 |
| :----------------------------- | :-------------------------------------------- |
| sys.dm\_pdw\_nodes\_db\_file\_space\_usage | [sys.dm\_db\_file\_space\_usage (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms174412.aspx) |
| sys.dm\_pdw\_nodes\_db\_index\_usage\_stats | [sys.dm\_db\_index\_usage\_stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188755.aspx) |
| sys.dm\_pdw\_nodes\_db\_partition\_stats | [sys.dm\_db\_partition\_stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187737.aspx) |
| sys.dm\_pdw\_nodes\_db\_session\_space\_usage | [sys.dm\_db\_session\_space\_usage (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187938.aspx) |
| sys.dm\_pdw\_nodes\_db\_task\_space\_usage | [sys.dm\_db\_task\_space\_usage (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190288.aspx) |
| sys.dm\_pdw\_nodes\_exec\_background\_job\_queue | [sys.dm\_exec\_background\_job\_queue (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173512.aspx) |
| sys.dm\_pdw\_nodes\_exec\_background\_job\_queue\_stats | [sys.dm\_exec\_background\_job\_queue\_stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms176059.aspx) |
| sys.dm\_pdw\_nodes\_exec\_cached\_plans | [sys.dm\_exec\_cached\_plans (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187404.aspx) |
| sys.dm\_pdw\_nodes\_exec\_connections | [sys.dm\_exec\_connections (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms181509.aspx) |
| sys.dm\_pdw\_nodes\_exec\_procedure\_stats | [sys.dm\_exec\_procedure\_stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/cc280701.aspx) |
| sys.dm\_pdw\_nodes\_exec\_query\_memory\_grants | [sys.dm\_exec\_query\_memory\_grants (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms365393.aspx) |
| sys.dm\_pdw\_nodes\_exec\_query\_optimizer\_info | [sys.dm\_exec\_query\_optimizer\_info (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms175002.aspx) |
| sys.dm\_pdw\_nodes\_exec\_query\_resource\_semaphores | [sys.dm\_exec\_query\_resource\_semaphores (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms366321.aspx) |
| sys.dm\_pdw\_nodes\_exec\_query\_stats | [sys.dm\_exec\_query\_stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189741.aspx) |
| sys.dm\_pdw\_nodes\_exec\_requests | [sys.dm\_exec\_requests (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms177648.aspx) |
| sys.dm\_pdw\_nodes\_exec\_sessions | sys.dm\_pdw\_nodes\_exec\_sessions (Transact-SQL) |
| sys.dm\_pdw\_nodes\_io\_cluster\_shared\_drives | [sys.dm\_io\_cluster\_shared\_drives (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188930.aspx) |
| sys.dm\_pdw\_nodes\_io\_pending\_io\_requests | [sys.dm\_io\_pending\_io\_requests (Transact-SQl)](http://msdn.microsoft.com/zh-cn/library/ms188762.aspx) |
| sys.dm\_pdw\_nodes\_os\_buffer\_descriptors | [sys.dm\_os\_buffer\_descriptors (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173442.aspx) |
| sys.dm\_pdw\_nodes\_os\_child\_instances | [sys.dm\_os\_child\_instances (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms165698.aspx) |
| sys.dm\_pdw\_nodes\_os\_cluster\_nodes | [sys.dm\_os\_cluster\_nodes (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187341.aspx) |
| sys.dm\_pdw\_nodes\_os\_dispatcher\_pools | [sys.dm\_os\_dispatcher\_pools (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb630336.aspx) |
| sys.dm\_pdw\_nodes\_os\_dispatchers | 未提供 Transact-SQl 文档。 |
| sys.dm\_pdw\_nodes\_os\_hosts | [sys.dm\_os\_hosts (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187800.aspx) |
| sys.dm\_pdw\_nodes\_os\_latch\_stats | [sys.dm\_os\_latch stats (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms175066.aspx) |
| sys.dm\_pdw\_nodes\_os\_loaded\_modules | [sys.dm\_os\_loaded\_modules (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms179907.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_brokers | [sys.dm\_os\_memory\_brokers (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb522548.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_cache\_clock\_hands | [sys.dm\_os\_memory\_cache\_clock\_hands (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173786.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_cache\_counters | [sys.dm\_os\_memory\_cache\_counters (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188760.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_cache\_entries | [sys.dm\_os\_memory\_cache\_entries (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189488.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_cache\_hash\_tables | [sys.dm\_os\_memory\_cache\_hash\_tables (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms182388.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_clerks | [sys.dm\_os\_memory\_clerks (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms175019.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_node\_access\_stats | 未提供 Transact-SQl 文档。 |
| sys.dm\_pdw\_nodes\_os\_memory\_nodes | [sys.dm\_os\_memory\_nodes (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb510622.aspx) |
| sys.dm\_pdw\_nodes\_os\_memory\_pools | [sys.dm\_os\_memory\_pools (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms175022.aspx) |
| sys.dm\_pdw\_nodes\_os\_nodes | [sys.dm\_os\_nodes (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb510628.aspx) |
| sys.dm\_pdw\_nodes\_os\_performance\_counters | [sys.dm\_os\_performance\_counters (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187743.aspx) |
| sys.dm\_pdw\_nodes\_os\_process\_memory | [sys.dm\_os\_process\_memory (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb510747.aspx) |
| sys.dm\_pdw\_nodes\_os\_schedulers | [sys.dm\_os\_schedulers (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms177526.aspx) |
| sys.dm\_pdw\_nodes\_os\_spinlock\_stats | sys.dm\_os\_spinlock\_stats (Transact-SQL) |
| sys.dm\_pdw\_nodes\_os\_sys\_info | [sys.dm\_os\_sys\_info (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms175048.aspx) |
| sys.dm\_pdw\_nodes\_os\_sys\_memory | [sys.dm\_os\_memory\_nodes (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb510622.aspx) |
| sys.dm\_pdw\_nodes\_os\_tasks | [sys.dm\_os\_tasks (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms174963.aspx) |
| sys.dm\_pdw\_nodes\_os\_threads | [sys.dm\_os\_threads (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187818.aspx) |
| sys.dm\_pdw\_nodes\_os\_virtual\_address\_dump | sys.dm\_virtual\_address\_dump (Transact-SQL) |
| sys.dm\_pdw\_nodes\_os\_wait\_stats | sys.ldm\_pdw\_nodes\_os\_wait\_stats (Transact-SQL) |
| sys.dm\_pdw\_nodes\_os\_waiting\_tasks | sys.dm\_waiting\_tasks (Transact-SQL) |
| sys.dm\_pdw\_nodes\_os\_workers | sys.dm\_workers (Transact-SQL) |
| sys.dm\_pdw\_nodes\_resource\_governor\_resource\_pools | [sys.dm\_resource\_governor\_resource\_pools (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb934023.aspx) |
| sys.dm\_pdw\_nodes\_resource\_governor\_workload\_groups | [sys.dm\_resource\_governor\_workload\_groups (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb934197.aspx) |
| sys.dm\_pdw\_nodes\_tran\_active\_snapshot\_database\_transactions | [sys.dm\_tran\_active\_snapshot\_database\_transactions (Transact\_SQL)](http://msdn.microsoft.com/zh-cn/library/ms180023.aspx) |
| sys.dm\_pdw\_nodes\_tran\_active\_transactions | [sys.dm\_tran\_active\_transactions (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms174302.aspx) |
| sys.dm\_pdw\_nodes\_tran\_commit\_table | 未提供 Transact-SQL 文档。 |
| sys.dm\_pdw\_nodes\_tran\_current\_snapshot | [sys.dm\_tran\_current\_snapshot (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms184390.aspx) |
| sys.dm\_pdw\_nodes\_tran\_locks | [sys.dm\_tran\_locks (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190345.aspx) |
| sys.dm\_pdw\_nodes\_tran\_session\_transactions | [sys.dm\_tran\_session\_transactions (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188739.aspx) |
| sys.dm\_pdw\_nodes\_tran\_top\_version\_generators | [sys.dm\_tran\_top\_version\_generators (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188778.aspx) |

## SQL 数据仓库中提供的 SQL Server 2016 PolyBase DMV

- [sys.dm\_exec\_compute\_node\_errors (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/mt146380.aspx)
- [sys.dm\_exec\_compute\_node\_status (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/mt146382.aspx)
- sys.dm\_exec\_compute\_nodes (Transact-SQL)
- sys.dm\_exec\_distributed\_request\_steps (Transact-SQL)
- [sys.dm\_exec\_distributed\_requests (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/mt146385.aspx)
- [sys.dm\_exec\_distributed\_sql\_requests (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/mt146390.aspx) 
- sys.dm\_exec\_dms\_services (Transact-SQL)
- sys.dm\_exec\_dms\_workers (Transact-SQL)
- sys.dm\_exec\_external\_operations (Transact-SQL)
- sys.dm\_exec\_external\_work (Transact-SQL)

## SQL 数据仓库中提供的 SQL 数据库 DMV

- sys.dm\_continuous\_copy\_status
- [sys.dm\_database\_copies](http://msdn.microsoft.com/zh-cn/library/azure/ff951634.aspx)
- sys.dm\_db\_objects\_impacted\_on\_version\_change
- sys.dm\_db\_resource\_stats
- sys.dm\_db\_wait\_stats
- [sys.dm\_operation\_status](http://msdn.microsoft.com/zh-cn/library/azure/jj126282.aspx)


## SQL Server information\_schema views

这是 SQL Server PDW 中提供的 SQL Server 信息架构视图的链接。预期在 SQL Server 中显示的所有 INFORMATION\_SCHEMA 视图表都可以在 SQL Server PDW 中找到。

- [CHECK\_CONSTRAINTS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189772.aspx)
- [COLUMN\_DOMAIN\_USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189447.aspx)
- [COLUMN\_PRIVILEGES (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms186812.aspx)
- [COLUMNS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188348.aspx)
- [CONSTRAINT COLUMN USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms174431.aspx)
- [CONSTRAINT\_TABLE\_USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms179883.aspx)
- [DOMAINS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190371.aspx)
- [KEY\_COLUMN\_USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189789.aspx)
- [PARAMETERS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173796.aspx)
- [REFERENTIAL CONSTRAINTS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms179987.aspx)
- [ROUTINE COLUMNS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms187350.aspx)
- [ROUTINES (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms188757.aspx)
- [SCHEMATA (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms182642.aspx)
- [TABLE CONSTRAINTS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms181757.aspx)
- [TABLE PRIVILEGES (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms186233.aspx)
- [TABLES (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms186224.aspx)
- [VIEW COLUMN USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190492.aspx)
- [VIEW TABLE USAGE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173869.aspx)
- [VIEWS (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms181381.aspx)

## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库参考概述]: /documentation/articles/sql-data-warehouse-overview-reference

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_1207_2015-->
