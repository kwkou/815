<properties 
	pageTitle="SQL 数据库中的兼容性级别 | Windows Azure" 
	description="介绍如何设置 Azure SQL 数据库的兼容性级别，以及受影响的功能。"
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="09/17/2015" 
	wacn.date="10/17/2015"/>


# SQL 数据库中的兼容性级别


对于 Azure SQL 数据库，本主题介绍你可以通过使用 Transact-SQL 语句 **ALTER DATABASE** 实现的功能选项。兼容性级别概念旨在降低任何升级风险。


按兼容性级别激活或停用的大部分功能是查询优化器的一部分。在以下情况下，Microsoft 根据兼容性级别激活新的查询优化器功能：


- 升级后的上下文更改了以前功能的语义。
- 查询优化有可能损害某些不常见但合法的 SQL 查询的性能。


如果对 Azure SQL 数据库服务器或数据库进行版本升级后，执行查询的效果不如以前，你可以选择降低兼容性级别。较低的设置可以停用少部分优化器增强功能，其中可能包括不利于你的查询的增强功能。


## 兼容级别的工作原理


Microsoft SQL Server 多年来一直具有可设置的兼容性级别。现在从版本 V12 开始，Azure SQL 数据库同样也采用了不同的兼容性级别。一些新功能的激活由每个 SQL 数据库中的兼容性级别设置控制。可能的设置包括：


- 110：可能的最低设置，因此该设置将不包括大多数新功能。
- 120：宽松，对应 SQL 数据库 V11（意味着 `SELECT @@version;` 返回 **11.**0.0.0）。
 - V11 版本也称为“SAWA V2”版的 SQL 数据库。
 - 此 120 是 Microsoft SQL Server 2014 中的最高值。
- 130：宽松，对应 V12（意味着 `SELECT @@version;` 返回 **12.**0.0.0）。
 - 此 130 是 Microsoft SQL Server 2016 中的最高值。


例如，如果设置为 120，意味着数据库有权访问在级别 120 上激活的功能子集，*以及*在任何更低的设置（如 110）下变为活动状态的功能。如果设置为 120，数据库不能访问级别 130 的功能。


## 读取或设置兼容级别


### 读取兼容级别


```
SELECT d.name, d.compatibility_level
	FROM sys.databases AS d
	WHERE d.name = db_name();
```


### 设置兼容级别


```
ALTER DATABASE YourDatabaseName SET COMPATIBILITY_LEVEL = 130;
```


有关详细信息，请参阅：


- [ALTER TABLE 兼容性级别 (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/bb510680.aspx)
- [sys.databases (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms178534.aspx)


## **MEMORY\_OPTIMIZED**，以及列存储索引


大部分在级别 120 和 130 上变为活动状态的兼容性级别功能都与*内存中*的表和索引相关。因此重要的项包括：


- **MEMORY\_OPTIMIZED** 表
- 列存储索引


在兼容性级别 130 上，激活了查询优化器能够以*批处理*模式处理一些查询，其中涉及到内存中的项。在涉及大量行的情况下，批处理模式的速度比*行*模式快。


### *如何*使用它，而不是*是否*使用它


假设以下各项：


- 你有一个常规表，该表具有列存储索引。
- 你查询这个具有列存储索引的表。
- 查询计划未提到使用列存储索引。


在这些情况下，如果尝试用不同的兼容级别来激活或停用与列存储索引相关的查询优化功能，不会对查询计划或查询的性能产生影响。

但是，如果查询计划提到使用列存储索引，则兼容级别就可能影响计划和性能。这意味着兼容性级别可以影响*如何*使用列存储索引，而不是*是否*使用列存储索引。


有关内存中表和列存储索引的详细信息，请参阅：


- [内存优化表](http://msdn.microsoft.com/zh-cn/library/dn133165.aspx)
- [创建群集的列存储索引 (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/dn511016.aspx)
- [列存储索引说明](http://msdn.microsoft.com/zh-cn/library/gg492088.aspx)


## 常规功能要求最低级别 130


基数估计器 (CE) 生成和存储有关表中大致行数的统计信息。查询优化器的很多部分使用该基数信息。


CE 所使用的算法已得到改进。改进会导致更改，而更改可能会导致不常见的特殊情况再次出现。


| 130 是最低<br/>必要<br/>级别 | 查询区域，<br/>常规 | 查询计划<br/>改进<br/>详细信息 |
| :-- | :-- | :-- |
| 130 | 基数估计器 (CE) | 级别 120 下基数估计器 (CE) 相比早期 CE 版本的优化情况。<br/><br/>在查询计划中，你可能会看到：**CardinalityEstimationModelVersion="130"**<br/><br/>**跟踪标志 4199**，以及兼容性级别 130，在 Microsoft SQL Server 2016 完全完成预览并发布通用版本 (GA) 后，可用来选择将修补程序加入或移出 CE。有关详细信息，请参阅：<br/><br/>● [知识库 (KB) 文章 974006](http://support.microsoft.com/kb/974006)。<br/>● **[DBCC TRACEON](http://msdn.microsoft.com/zh-cn/library/ms187329.aspx) (4199);**。|
| 130 | **MEMORY\_OPTIMIZED** 表并行查询计划 | 如果查询针对的是内存中表，可以使用多个线程并可以并行运行，这意味着表通过 **MEMORY\_OPTIMIZED = YES** 子句创建。并行化可以使查询运行得更快。 |


## 列存储索引功能要求最低级别 130


在兼容性级别 130 上，查询优化器增强功能有时会产生新的查询计划。对于涉及到列存储索引的已更改计划，更改通常涉及下列情况之一：


- 必须复制数据以进行进一步的子操作的情况减少。
- 排序等操作从*行*处理更改为***批处理***。


在大多数情况下，计划更改可以提高查询性能。有时改进会导致 **tempdb** 数据库的使用量增加。


| 130 是最低<br/>必要<br/>级别 | 查询区域，<br/>列存储索引 | 查询计划<br/>改进<br/>详细信息 |
| :-- | :-- | :-- |
| 130 | 函数查询 | 在以下情况下，通过切换到批处理模式，性能得以改进：<br/><br/>• 涉及排序。<br/><br/>• 聚合*多个*不同的函数<br/>（以下列表中两个不同的加重号，每个一个函数）：<br/>&nbsp;&nbsp;&nbsp;▫ **COUNT** *或* **COUNT\_BIG**<br/>&nbsp;&nbsp;&nbsp;▫ **AVG** *或* **SUM**<br/>&nbsp;&nbsp;&nbsp;▫ **CHECKSUM\_AGG**<br/>&nbsp;&nbsp;&nbsp;▫ **STDEV** *或* **STDEVP**<br/><br/>• Window 聚合函数<br/>（在 [MSDN 的这篇文档](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx)、以及 [Kathi Kellenberger 的这篇文档](http://www.bidn.com/blogs/KathiKellenberger/sql-server/4397/what-is-a-window-aggregate-function)中进行了介绍）：<br/>&nbsp;&nbsp;&nbsp;▫ **COUNT**、**COUNT\_BIG**、**SUM**、**AVG**、**MIN**、**MAX**、**CLR**<br/><br/>• Window [用户定义的](http://msdn.microsoft.com/zh-cn/library/ms131057.aspx)聚合：<br/>&nbsp;&nbsp;&nbsp;▫ [**CHECKSUM\_AGG**](http://msdn.microsoft.com/zh-cn/library/ms188920.aspx)、[**STDEV**](http://msdn.microsoft.com/zh-cn/library/ms190474.aspx)、[**STDEVP**](http://msdn.microsoft.com/zh-cn/library/ms176080.aspx)、[**VAR**](http://msdn.microsoft.com/zh-cn/library/ms186290.aspx)、[**VARP**](http://msdn.microsoft.com/zh-cn/library/ms188735.aspx)、[**GROUPING**](http://msdn.microsoft.com/zh-cn/library/ms178544.aspx)<br/><br/>• Window 聚合分析函数：<br/>&nbsp;&nbsp;&nbsp;▫ [**LAG**](http://msdn.microsoft.com/zh-cn/library/hh231256.aspx)、[**LEAD**](http://msdn.microsoft.com/zh-cn/library/hh213125.aspx)、[**FIRST\_VALUE**](http://msdn.microsoft.com/zh-cn/library/hh213018.aspx)、[**LAST\_VALUE**](http://msdn.microsoft.com/zh-cn/library/hh231517.aspx)、[**PERCENTILE\_CONT**](http://msdn.microsoft.com/zh-cn/library/hh231473.aspx)、[**PERCENTILE\_DISC**](http://msdn.microsoft.com/zh-cn/library/hh231327.aspx)、[**CUME\_DIST**](http://msdn.microsoft.com/zh-cn/library/hh231078.aspx)、[**PERCENT\_RANK**](http://msdn.microsoft.com/zh-cn/library/hh213573.aspx) |
| 130 | 单线程串行查询计划 | 在单个线程上执行的查询可以在批处理模式下运行。这会使查询的执行速度更快。<br/><br/>查询计划可能会设计为单线程，或者查询可能会在 **MAXDOP 1** 下运行。 |


## 常规功能要求最低级别 120


| 120 是最低<br/>必要<br/>级别 | 查询区域，<br/>常规 | 查询计划<br/>改进<br/>详细信息 |
| :-- | :-- | :-- |
| 120 | [更改 MEMORY\_OPTIMIZED 表](http://msdn.microsoft.com/zh-cn/library/dn269114.aspx) | 使你可以对 **MEMORY\_OPTIMIZED = YES** 的表执行 Transact-SQL **ALTER TABLE** 操作。<br/><br/>数据库应用程序可以继续运行，但访问该表的操作被阻止，直到 **ALTER TABLE** 完成。 |
| 120 | [创建和管理内存优化对象的存储](http://msdn.microsoft.com/zh-cn/library/dn133174.aspx) | 内存中 OLTP 引擎集成在 SQL Server 中。这让你可以同时拥有 **MEMORY\_OPTIMIZED** 表和同一数据库中基于磁盘的传统表。 |
| 120 | [内存中 OLTP 的 Transact-SQL 支持](http://msdn.microsoft.com/zh-cn/library/dn133180.aspx) | 少量的 Transact-SQL 命令已经过改进，可以支持内存中的联机事务处理 (OLTP)。<br/><br/>例如，[CREATE PROCEDURE](http://msdn.microsoft.com/zh-cn/library/ms187926.aspx) 命令的新 **NATIVE\_COMPILATION** 关键字。 |
| 120 | 基数估计器 (CE) | 级别 110 下基数估计器 (CE) 相比早期 CE 版本的优化情况。<br/><br/>在查询计划中，你可能会看到：**CardinalityEstimationModelVersion="120"**<br/><br/>有关**跟踪标志 4199** 如何与兼容性级别值交互的详细信息，请参阅 [KB 974006](http://support.microsoft.com/kb/974006)。|


## 列存储索引功能要求最低级别 120


本部分介绍在兼容性级别 120 或更高级别下激活的[列存储索引编制功能](http://msdn.microsoft.com/zh-cn/library/dn934994.aspx)。


| 120 是最低<br/>兼容性<br/>级别 | 列存储索引<br/>功能名称 | 列存储索引<br/>功能说明 |
| :-- | :-- | :-- |
| 120 | 快照隔离 (SI) 级别，以及<br/><br/>读提交快照隔离 (RCSI) 级别。 | 当查询计划涉及到列存储索引时，SI 和 RCSI 将阻止来自包含在查询结果中的部分完成事务中的数据，无需使用过多的锁定。 |
| 120 | 索引碎片整理 | 删除已删除的行，不显式重新生成索引。<br/><br/>**ALTER INDEX ...REORGANIZE** 从列存储索引删除已删除的行，而表和索引仍可联机操作。 |
| 120 | 可在 AlwaysOn 上访问[可读辅助副本](http://msdn.microsoft.com/zh-cn/library/ff878253.aspx) | 可以通过将分析查询工作负载转移到 AlwaysOn 辅助副本来提高操作分析的性能。 |
| 120 | 在表扫描过程中计算聚合函数 | 提高性能。<br/><br/>适用于 **MIN**、**MAX**、**SUM**、**COUNT**、**AVG**。<br/><br/>仅适用于当数据类型为八个字节或更少的情况，尽管不适用于字符串。 |
| 120 | 下推基于字符串的 **WHERE** 子句 | 谓词下推优化可以加快比较字符串数据类型 &#x5b;var&#x5d;char 或 n&#x5b;var&#x5d;char 的查询。这种优化：<br/><br/>• 适用于常见的比较运算符，包括使用位图筛选器的 **LIKE**。<br/><br/>• 仅当有一个字符串谓词时有效。<br/><br/>• 适用于该产品支持的所有排序规则。<br/><br/>若要了解有关位图筛选器的更多详细信息，请参阅此博客文章：[查询执行位图筛选器简介](http://blogs.msdn.com/b/sqlqueryprocessing/archive/2006/10/27/query-execution-bitmaps.aspx)。 |


## 查询计划按兼容级别更改的示例


对于一个 Transact-SQL **INSERT...SELECT** 语句，本部分显示在其兼容级别 120 和 130 之间的查询计划中的变化。


#### 源表架构


下表包含至少 300000 行。如果数据太少而不值得使用并行化功能，则高级查询计划可能不会受到并行化功能的影响。


```
CREATE TABLE [dbo].[ccitestoriginal](
	[FinanceKey] [int] NOT NULL,
	[DateKey] [int] NOT NULL,
	[OrganizationKey] [int] NOT NULL,
	[DepartmentGroupKey] [int] NOT NULL,
	[ScenarioKey] [int] NOT NULL,
	[AccountKey] [int] NOT NULL,
	[Amount] [float] NOT NULL,
	[Date] [datetime] NULL
) ON [PRIMARY];
GO

CREATE CLUSTERED COLUMNSTORE INDEX [cci_ccitestoriginal]
	ON [dbo].[ccitestoriginal]
	WITH (DROP_EXISTING = OFF) ON [PRIMARY];
GO
```


#### 生成查询计划的脚本


将这些步骤应用到随后的 Transact-SQL 脚本：


1. 在 **Ssms.exe** 中连接到你的数据库。
2. 将 Transact-SQL 脚本粘贴到 **Ssms.exe** 查询窗口中。
3. 编辑该脚本以确保 **120** 是 **ALTER DATABASE** 语句的值。
4. 仅运行该脚本中 **INSERT INTO** 语句*之前*的部分。
5. 激活菜单选项：“查询”>“包括实际的执行计划 (Ctrl + M)”。
6. 仅运行 **INSERT INTO** 语句。<br/><br/>
7. 停用菜单选项：“查询”>“包括实际的执行计划 (Ctrl + M)”。
8. 最后，仅运行该脚本中 **INSERT INTO** 语句*之后*的部分。
9. 请注意，“执行计划”选项卡中 **120** 的查询计划位于“结果”选项卡附近。<br/>-- -- -- -- -- --
10. 编辑该脚本以确保 **130** 是 **ALTER DATABASE** 的值。
11. 如前面的步骤中所述重新运行该脚本。
12. 请注意，**130** 的查询计划不同，其包含并行化功能。


&nbsp;


```
go
SET NOCOUNT ON;
go

ALTER DATABASE YourDatabaseName SET COMPATIBILITY_LEVEL =
	120
	--130
;
go

DROP TABLE ccitest_into_work;
go

SELECT *
	INTO ccitest_into_work  -- Create an empty copy of the table.
	FROM ccitestoriginal
	WHERE 1=2;
go

CREATE CLUSTERED COLUMNSTORE INDEX ccitest_into_work_cci
	ON ccitest_into_work;
go

SET NOCOUNT OFF;
go
	
	-- Run this INSERT statement alone!
	--
	-- First run all the preceding Transact-SQL statements.
	-- Then run this one INSERT statement alone.
	-- Then run all remaining Transact-SQL statements.

INSERT INTO ccitest_into_work WITH (TABLOCK)
	SELECT TOP 300000 *
		FROM ccitestoriginal;
go

SET NOCOUNT ON;
go

DROP TABLE ccitest_into_work;
go
```


#### 两种计划的显示



<br/>**120：**这是兼容性级别 **120** 下的查询计划。


![级别 120 下的计划][1-Plan-at-level-120]


<br/>**130：**这是兼容性级别 **130** 下的查询计划。


**130** 计划包含并行化功能，而 **120** 计划没有。


此计划的显示在 **Ssms.exe** 中更宽。为了在这里更好地显示，以下屏幕快照被分为两个部分。第二部分将接着第一部分继续显示。


![级别 130 下的计划][2-Plan-at-level-130]


## 你的数据库中的默认兼容性级别


当你升级 Azure SQL 数据库服务器时，例如从版本 V11 升级到 V12，每个数据库中的兼容性级别保持不变。升级之后，你可以使用 **ALTER DATABASE** 语句，提高任何给定数据库中的兼容性级别。


新创建的任何新数据库将具有 SQL 数据库版本允许的最大兼容性级别。



<!-- Image references. -->

[1-Plan-at-level-120]: ./media/sql-database-compatibility-level/sql-db-compat-level-query-plan-b-120.png

[2-Plan-at-level-130]: ./media/sql-database-compatibility-level/sql-db-compat-level-query-plan-b12-130.png

<!---HONumber=74-->