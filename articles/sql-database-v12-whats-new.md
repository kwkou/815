<properties 
	pageTitle="SQL Database V12 的新增功能" 
	description="介绍 Azure SQL Database 中添加的最新功能。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/24/2015"
	wacn.date="05/25/2015" 
	ms.author="genemi"/>


# SQL Database V12 的新增功能


Azure SQL Database 版本 V12（[在某些区域以预览版提供](sql-database-v12-whats-new#V12AzureSqlDbPreviewGaTable)）现在几乎与 Microsoft SQL Server 引擎完全兼容。最近的增强有助于简化将 SQL Server 应用程序迁移到 SQL Database，并且可以帮助你的系统可靠处理更大的数据库工作负载。


[注册](https://manage.windowsazure.cn) SQL Database，然后即可[开始使用](sql-database-get-started)这款全新一代的 Windows Azure。你首先需要订阅 Windows Azure。你可以注册 [Azure 试用版](/pricing/1rmb-trial)并查看[价格](/home/features/sql-database/#price)信息。


若你打算将早期版本的数据库升级到 V12，应该先阅读[规划和准备升级到 SQL Database V12](sql-database-v12-plan-prepare-upgrade)。


## 产品亮点


- 可以[使用](http://manage.windowsazure.cn) **Windows Azure 门户**创建可在版本 12（有时还包括早期版本）上运行的 SQL Database 数据库和服务器。在 Azure 门户中，请指定你的 SQL Database 数据库，然后继续指定用于托管该数据库的 SQL Database 服务器。目前也支持早期的 [Windows Azure 经典门户](http://manage.windowsazure.com)。


- 在使用 Azure 门户创建新数据库时，请**选择 SQL Database 服务器的版本**。默认版本为 V12，不过，你也可以选择以前的 SQL Database 服务器版本。


- **安全**享用新功能[包含的数据库中的用户](sql-database-v12-whats-new#UsersInContainedDatabases)。另外两项功能分别为[行级安全性](sql-database-v12-whats-new#RowLevelSecurity)和[动态数据屏蔽](sql-database-v12-whats-new#DynamicDataMasking)，不过，这两项功能仍为预览版，尚未正式推出。


- 使用并行查询（仅限高级版）、[表分区](http://msdn.microsoft.com/zh-cn/library/ms187802.aspx)、[联机索引](http://msdn.microsoft.com/zh-cn/library/ms188388.aspx)、尽情重建大型索引（去除了 2GB 大小限制）和 [ALTER DATABASE](http://msdn.microsoft.com/zh-cn/library/bb522682.aspx) 命令的更多选项，**更轻松地管理**大型数据库。


- 凭借 [CLR 集成](http://msdn.microsoft.com/zh-cn/library/ms189524.aspx)、Transact-SQL [开窗函数](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx)、[XML 索引](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx)和[数据更改跟踪](http://msdn.microsoft.com/zh-cn/library/bb933875.aspx)**支持关键的编程功能**，推出更稳健的应用程序设计。


- 支持对数据集市和小型分析工作负载运行内存中[列存储索引](http://msdn.microsoft.com/zh-cn/library/gg492153.aspx)查询（仅限高级版），**性能有所突破**。


- **监视和故障排除**已得到改进，现在你可以在一组扩展的数据库管理视图 ([DMV](http://msdn.microsoft.com/zh-cn/library/ms188754.aspx)) 中查看 100 多个新表视图。


- **标准层中新增了 S3 性能级别：**在标准和高级层之间提供更高的[价格](sql-database-upgrade-new-service-tiers)弹性。S3 将提供更多的 DTU（数据库吞吐量单位）以及标准层所提供的全部功能。


## V12 增强功能


### 扩展的数据库管理


| 功能 | 说明 |
| :--- | :--- |
| <a name="UsersInContainedDatabases" id="UsersInContainedDatabases"></a>包含的数据库中的用户 | 现在，你可以在包含的数据库中创建用户，而无需在 master 数据库中创建相应的登录名。这样，将数据库迁移到另一台服务器就要方便得多。无论你是否使用包含的数据库用户，客户端应用程序中的连接代码都会相同。<br/><br/>使用此功能是受益于更高的、有保证的服务级别协议的极佳途径。<br/><br/>一般情况下，我们建议你考虑使用此功能。但是，在你的某些方案中，传统的 *登录名+用户*对形式可能暂时更适合你。<br/><br/>有关详细信息，请参阅：<br/>- [Azure SQL Database 安全指导原则和限制](http://msdn.microsoft.com/zh-cn/library/azure/ff394108.aspx)<br/>- [在 Azure SQL Database 中管理数据库和登录名](http://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)<br/>- [包含的数据库](http://msdn.microsoft.com/zh-cn/library/azure/ff929071.aspx) |
| 表分区 | 以前对[表分区](http://msdn.microsoft.com/zh-cn/library/ms190787.aspx)的限制现已去除。|
| 更大的事务 | 使用 V12 时，你不再受限于最多只能在一个事务中进行 2 GB 的数据修改。 <br/><br/> 该版本的优势之一在于，重建大型索引不再受限于 2 GB 事务大小限制。有关一般信息，请参阅 [Azure SQL Database 资源限制](http://msdn.microsoft.com/zh-cn/library/azure/dn338081.aspx)。 |
| 联机索引生成和重建 | 在 V12 以前的版本中，Azure SQL Database 一般支持 [ALTER INDEX](http://msdn.microsoft.com/zh-cn/library/ms188388.aspx) 命令的 (ONLINE=ON) 子句，但 BLOB 类型列的索引不支持此子句。现在，即使是对于 BLOB 列的索引，V12 也能支持 (ONLINE=ON)。<br/><br/> ONLINE 功能使查询能够受益于索引，即使索引正在重建。 |
| CHECKPOINT 支持 | 使用 V12 时，你可以对数据库发出 Transact-SQL [CHECKPOINT](http://msdn.microsoft.com/zh-cn/library/ms188748.aspx) 命令。|
| ALTER TABLE 增强 | 允许在保持表活动状态的同时执行许多 alter column 操作。有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190273.aspx) |
| TRUNCATE TRUNCATE TABLE 增强 | 允许截断特定分区。有关详细信息，请参阅 [TRUNCATE TABLE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms177570.aspx)。 |
| ALTER DATABASE 的更多选项 | V12 支持 [ALTER DATABASE](http://msdn.microsoft.com/zh-cn/library/ms174269.aspx) 命令可用的更多选项。 <br/><br/> 有关详细信息，请参阅 [Azure SQL Database Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |
| 更多的 DBCC 命令 | V12 中现在提供了更多的 [DBCC](http://msdn.microsoft.com/zh-cn/library/ms188796.aspx) 命令。有关详细信息，请参阅 [Azure SQL Database Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |


### 编程和应用程序支持


| 功能 | 说明 |
| :--- | :--- |
| <a name="DynamicDataMasking" id="DynamicDataMasking"></a> 动态数据屏蔽预览版 | 在基于查询生成行集时，建立的数据屏蔽策略可将部分数据替换为"X"字符，以覆盖和保护敏感信息。在完成屏蔽操作后，会将修改的行集发送到客户端。<br/><br/>示例用法之一是屏蔽信用卡号中除最后几位数以外的所有数字。<br/><br/>**注意：**此功能处于预览状态，尚未宣布可在生产环境中使用的正式版。<br/><br/>有关详细信息，请参阅 [SQL Database 动态数据屏蔽入门](sql-database-dynamic-data-masking-get-started)。 |
| <a name="RowLevelSecurity" id="RowLevelSecurity"></a> 行级安全性 (RLS) 预览版 | 你可以使用 Transact-SQL 中新的 [CREATE SECURITY POLICY](http://msdn.microsoft.com/zh-cn/library/dn765135.aspx) 命令来实现 RLS。数据库服务器可通过 RLS 添加条件，以便在将行集返回给调用方之前筛选掉某些数据行。<br/><br/>在行业中，RLS 有时也称为精细访问控制。<br/><br/>**注意：**此功能处于预览状态，尚未宣布可在生产环境中使用的正式版。<br/><br/>有关代码示例和详细信息，请参阅[行级别安全性预览版](http://msdn.microsoft.com/zh-cn/library/7221fa4e-ca4a-4d5c-9f93-1b8a4af7b9e8.aspx)。 |
| Transact-SQL 查询中的开窗函数 | 可以使用 [OVER 子句](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx)访问 ANSI 开窗函数。 <br/><br/> Itzik Ben-Gan 发表了一篇很好的[博客文章](http://sqlmag.com/sql-server-2012/how-use-microsoft-sql-server-2012s-window-functions-part-1)，其中对开窗函数和 OVER 子句做了深层解释。 |
| .NET CLR 集成 | .NET framework 的 CLR（公共语言运行时）已集成到 V12。 <br/><br/> 仅支持已完全编译到二进制代码中的 SAFE 程序集。有关详细信息，请参阅 [CLR 集成编程模型限制](http://msdn.microsoft.com/zh-cn/library/ms403273.aspx)。 <br/><br/> 可在以下主题中找到有关此功能的信息： <br/> * [SQL Server CLR 集成简介](http://msdn.microsoft.com/zh-cn/library/ms254498.aspx) <br/> * [CREATE ASSEMBLY (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189524.aspx)。 |
| 数据更改跟踪 | 现在，可以在数据库或表级别配置数据更改跟踪。 <br/><br/> 有关更改跟踪的信息，请参阅[关于更改跟踪 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/bb933875.aspx)。 |
| XML 索引 | V12 允许你使用 Transact-SQL 命令 [CREATE XML INDEX](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx) 和 [CREATE SELECTIVE XML INDEX](http://msdn.microsoft.com/zh-cn/library/jj670104.aspx)。|
| 表即堆 | V12 允许你创建不带聚集索引的表。 <br/><br/> 此功能支持 Transact-SQL [SELECT...INTO](http://msdn.microsoft.com/zh-cn/library/ms188029.aspx) 命令（基于查询结果创建表），因此特别有用。 |
| 应用程序角色 | 在安全和权限控制方面，V12 允许你针对[应用程序角色](http://msdn.microsoft.com/zh-cn/library/ms190998.aspx)发出 [GRANT](http://msdn.microsoft.com/zh-cn/library/ms187965.aspx) - [DENY](http://msdn.microsoft.com/zh-cn/library/ms188338.aspx) - [REVOKE](http://msdn.microsoft.com/zh-cn/library/ms187728.aspx) 命令。|


### 改进了客户洞察功能


| 功能 | 说明 |
| :--- | :--- |
| DMV（动态管理视图） | V12 中增加了多个 DMV。有关详细信息，请参阅 [Azure SQL Database Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |
| 更改跟踪 | V12 完全支持更改跟踪。 <br/><br/> 有关此功能的详细信息，请参阅[启用和禁用更改跟踪 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/bb964713.aspx)。 |
| 列存储索引 | 当索引列包含相同值的重复条目时，列存储索引可以改进数据仓库的系统性能。[列存储索引将压缩](http://msdn.microsoft.com/zh-cn/library/gg492088.aspx)重复值，以减少查询期间必须访问的物理行数。 |


### 高级服务层的性能改进


这些性能增强适用于高级服务层中的 P2 和 P3 级别。


| 功能 | 说明 |
| :--- | :--- |
| 查询的并行处理 | 对于可从并行度受益的查询，V12 提供更高的并行度。|
| 更短的 I/O 延迟 | V12 明显缩短了输入/输出操作的延迟时间。|
| 提高了 IOPS | V12 可以处理更大的每秒输入/输出数量 (IOPS)。|
| 日志速率 | V12 每秒可以记录更多的数据更改。|


### 增强功能摘要


V12 增强了 SQL Database，使其功能几乎完全与我们的 SQL Server 产品兼容。V12 注重最流行的功能和编程支持。这样就使得开发工作变得更高效且充满乐趣。现在，你可以更轻松地将 SQL 数据库应用程序迁移到云中。


此外，高级层 V12 为性能带来了重大改进。某些应用程序的查询速度将会得到极大提高。工作负载较重的应用程序可以提供可靠稳定的吞吐量。


## 如何继续下一步


你可以在[规划和准备升级到 Azure SQL Database V12](sql-database-v12-plan-prepare-upgrade) 中了解如何将数据库从 Azure SQL Database V11 升级到 V12。


类似于 V12 的版本号是指以下 Transact-SQL 命令返回的值：<br/>
**SELECT @@version;**


- 11.0.9228.18 *(V11)*
- 12.0.2000.8 *(or a bit higher, V12)*


有关 V12 最新定价详细信息，请参阅 [SQL Database 定价](/home/features/sql-database/#price)。


## V12 升级注意事项


在将 V11 数据库升级到 V12 期间和之后，需要了解一些有关限制的重要事项。你可以通过这个指向 *规划和准备升级到 Azure SQL Database V12* 主题[中间部分](sql-database-v12-plan-prepare-upgrade#limitations)的链接来了解详细信息。
