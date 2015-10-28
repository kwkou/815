<properties
	pageTitle="SQL 数据库 V12 的新增功能"
	description="介绍 Azure SQL 数据库 中添加的最新功能。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"  
	ms.date="04/28/2015"
	wacn.date="06/30/2015"/>


# SQL 数据库 V12 的新增功能




Azure SQL 数据库 版本 V12（[在某些区域以预览版提供](/documentation/articles/sql-database-preview-whats-new#V12AzureSqlDbPreviewGaTable)）现在几乎与 Microsoft SQL Server 引擎完全兼容。最近的增强有助于简化将 SQL Server 应用程序迁移到 SQL 数据库，并且可以帮助你的系统可靠处理更大的数据库工作负载。


[注册](https://manage.windowsazure.cn) SQL 数据库，然后即可[开始使用](/documentation/articles/sql-database-get-started)这款全新一代的 Windows Azure。你首先需要订阅 Windows Azure。你可以注册 [Azure 试用版](/pricing/1rmb-trial)并查看[价格](/home/features/sql-database/#price)信息。


若你打算将早期版本的数据库升级到 V12，应该先阅读[规划和准备升级到 SQL 数据库 V12](/documentation/articles/sql-database-preview-plan-prepare-upgrade)。


## 产品亮点


- 可以[使用](http://manage.windowsazure.cn/) **Azure 门户**创建可在版本 12（有时还包括早期版本）上运行的 SQL 数据库 数据库和服务器。在 Azure 门户中，请指定你的 SQL 数据库 数据库，然后继续指定用于托管该数据库的 SQL 数据库 服务器。


- 在使用 Azure 门户创建新数据库时，请**选择 SQL 数据库 服务器的版本**。默认版本为 V12，不过，你也可以选择以前的 SQL 数据库 服务器版本。


- **安全**享用新功能包含的[数据库中的用户](/documentation/articles/sql-database-preview-whats-new#UsersInContainedDatabases)。另外两项功能分别为[行级安全性](/documentation/articles/sql-database-preview-whats-new#RowLevelSecurity)和[动态数据屏蔽](/documentation/articles/sql-database-preview-whats-new#DynamicDataMasking)，不过，这两项功能仍为预览版，尚未正式推出。


- 使用并行查询（仅限高级版）、[表分区](http://msdn.microsoft.com/zh-cn/library/ms187802.aspx)、[联机索引](http://msdn.microsoft.com/zh-cn/library/ms188388.aspx)、尽情重建大型索引（去除了 2GB 大小限制）和 [ALTER DATABASE](http://msdn.microsoft.com/zh-cn/library/bb522682.aspx) 命令的更多选项，**更轻松地管理**大型数据库。


- 凭借 **CLR 集成**、Transact-SQL [开窗函数](http://msdn.microsoft.com/zh-cn/library/ms189524.aspx)、[XML 索引](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx)和数据[更改跟踪](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx)[支持关键的编程功能](http://msdn.microsoft.com/zh-cn/library/bb933875.aspx)，推出更稳健的应用程序设计。


- 支持对数据集市和小型分析工作负载运行内存中[列存储索引](http://msdn.microsoft.com/zh-cn/library/gg492153.aspx)查询（仅限高级版），**性能有所突破**。


- **监视和故障排除**已得到改进，现在你可以在一组扩展的数据库管理视图 ([DMV](http://msdn.microsoft.com/zh-cn/library/ms188754.aspx)) 中查看 100 多个新表视图。


- **标准层中新增了 S3 性能级别：**在标准和高级层之间提供更高的[价格](/documentation/articles/sql-database-upgrade-new-service-tiers)弹性。S3 将提供更多的 DTU（数据库吞吐量单位）以及标准层所提供的全部功能。


## V12 增强功能


### 扩展的数据库管理


| 功能 | 说明 |
| :--- | :--- |
| 。 | ***2014 年 12 月：*** |
| <a name="UsersInContainedDatabases" id="UsersInContainedDatabases"></a>包含的数据库中的用户 | 现在，你可以在包含的数据库中创建用户，而无需在 master 数据库中创建相应的登录名。这样，将数据库迁移到另一台服务器就要方便得多。无论你是否使用包含的数据库用户，客户端应用程序中的连接代码都会相同。<br/><br/>使用此功能是受益于更高的、有保证的服务级别协议的极佳途径。<br/><br/>一般情况下，我们建议你考虑使用此功能。但是，在你的某些方案中，传统的*登录名+用户*对形式可能暂时更适合你。<br/><br/>有关详细信息，请参阅：<br/>- [Azure SQL 数据库 安全指导原则和限制](http://msdn.microsoft.com/zh-cn/library/azure/ff394108.aspx)<br/>- [在 Azure SQL 数据库 中管理数据库和登录名](http://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)<br/>- [包含的数据库](http://msdn.microsoft.com/zh-cn/library/azure/ff929071.aspx) |
| 表分区 | 以前对[表分区](http://msdn.microsoft.com/zh-cn/library/ms190787.aspx)的限制现已去除。 |
| 更大的事务 | 使用 V12 时，你不再受限于最多只能在一个事务中进行 2 GB 的数据修改。<br/><br/> 该版本的优势之一在于，重建大型索引不再受限于 2 GB 事务大小限制。有关一般信息，请参阅 [Azure SQL 数据库 资源限制](http://msdn.microsoft.com/zh-cn/library/azure/dn338081.aspx)。 |
| 联机索引生成和重建 | 在 V12 以前的版本中，Azure SQL 数据库 一般支持 [ALTER INDEX](http://msdn.microsoft.com/zh-cn/library/ms188388.aspx) 命令的 (ONLINE=ON) 子句，但 BLOB 类型列的索引不支持此子句。现在，即使是对于 BLOB 列的索引，V12 也能支持 (ONLINE=ON)。<br/><br/> ONLINE 功能使查询能够受益于索引，即使索引正在重建。 |
| CHECKPOINT 支持 | 使用 V12 时，你可以对数据库发出 Transact-SQL [CHECKPOINT](http://msdn.microsoft.com/zh-cn/library/ms188748.aspx) 命令。 |
| ALTER TABLE 增强 | 允许在保持表活动状态的同时执行许多 alter column 操作。有关详细信息，请参阅 [ALTER TABLE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms190273.aspx) |
| TRUNCATE TABLE 增强 | 允许截断特定分区。有关详细信息，请参阅 [TRUNCATE TABLE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms177570.aspx)。 |
| ALTER DATABASE 的更多选项 | V12 支持 [ALTER DATABASE](http://msdn.microsoft.com/zh-cn/library/ms174269.aspx) 命令可用的更多选项。<br/><br/>有关详细信息，请参阅 [Azure SQL 数据库 Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |
| 更多的 DBCC 命令 | V12 中现在提供了更多的 [DBCC](http://msdn.microsoft.com/zh-cn/library/ms188796.aspx) 命令。有关详细信息，请参阅 [Azure SQL 数据库 Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |


### 编程和应用程序支持


| 功能 | 说明 |
| :--- | :--- |
| 。 | ***2015 年 2 月：*** |
| <a name="DynamicDataMasking" id="DynamicDataMasking"></a> 动态数据屏蔽预览版 | 在基于查询生成行集时，建立的数据屏蔽策略可将部分数据替换为“X”字符，以覆盖和保护敏感信息。在完成屏蔽操作后，会将修改的行集发送到客户端。<br/><br/>示例用法之一是屏蔽信用卡号中除最后几位数以外的所有数字。<br/><br/>**注意：**此功能处于预览状态，尚未宣布可在生产环境中使用的正式版。<br/><br/>有关详细信息，请参阅 [SQL 数据库 动态数据屏蔽入门](/documentation/articles/sql-database-dynamic-data-masking-get-started)。 |
| 。 | ***2015 年 1 月：*** |
| <a name="RowLevelSecurity" id="RowLevelSecurity"></a> 行级安全性 (RLS) 预览版 | **注意：**RLS 功能目前只提供*预览版*，即使在 V12 已发布正式版 (GA) 的地理区域，也是如此。RLS 在发布正式版之前，尚不适合在业务关键型生产数据库中使用。<br/><br/>你可以使用 Transact-SQL 中新的 [CREATE SECURITY POLICY](http://msdn.microsoft.com/zh-cn/library/dn765135.aspx) 命令来实现 RLS。数据库服务器可通过 RLS 添加条件，以便在将行集返回给调用方之前筛选掉某些数据行。<br/><br/>在行业中，RLS 有时也称为精细访问控制。<br/><br/>有关代码示例和详细信息，请参阅[行级别安全性预览版](http://msdn.microsoft.com/zh-cn/library/7221fa4e-ca4a-4d5c-9f93-1b8a4af7b9e8.aspx)。 |
| 。 | ***2014 年 12 月：*** |
| Transact-SQL 查询中的开窗函数 | 可以使用 [OVER 子句](http://msdn.microsoft.com/zh-cn/library/ms189461.aspx)访问 ANSI 开窗函数。<br/><br/> Itzik Ben-Gan 发表了一篇很好的[博客文章](http://sqlmag.com/sql-server-2012/how-use-microsoft-sql-server-2012s-window-functions-part-1)，其中对开窗函数和 OVER 子句做了深层解释。 |
| .NET CLR 集成 | .NET framework 的 CLR（公共语言运行时）已集成到 V12。<br/><br/> 仅支持已完全编译到二进制代码中的 SAFE 程序集。有关详细信息，请参阅 [CLR 集成编程模型限制](http://msdn.microsoft.com/zh-cn/library/ms403273.aspx)。<br/><br/> 可在以下主题中找到有关此功能的信息：<br/> * [SQL Server CLR 集成简介](http://msdn.microsoft.com/zh-cn/library/ms254498.aspx) <br/> * [CREATE ASSEMBLY (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms189524.aspx)。 |
| 数据更改跟踪 | 现在，可以在数据库或表级别配置数据更改跟踪。<br/><br/> 有关更改跟踪的信息，请参阅[关于更改跟踪 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/bb933875.aspx)。 |
| XML 索引 | V12 允许你使用 Transact-SQL 命令 [CREATE XML INDEX](http://msdn.microsoft.com/zh-cn/library/bb934097.aspx) 和 [CREATE SELECTIVE XML INDEX](http://msdn.microsoft.com/zh-cn/library/jj670104.aspx)。 |
| 表即堆 | V12 允许你创建不带聚集索引的表。<br/><br/> 此功能支持 Transact-SQL [SELECT...INTO](http://msdn.microsoft.com/zh-cn/library/ms188029.aspx) 命令（基于查询结果创建表），因此特别有用。 |
| 应用程序角色 | 在安全和权限控制方面，V12 允许你针对[应用程序角色](http://msdn.microsoft.com/zh-cn/library/ms190998.aspx)发出 [GRANT](http://msdn.microsoft.com/zh-cn/library/ms187965.aspx) - [DENY](http://msdn.microsoft.com/zh-cn/library/ms188338.aspx) - [REVOKE](http://msdn.microsoft.com/zh-cn/library/ms187728.aspx) 命令。 |


### 改进了客户洞察功能


| 功能 | 说明 |
| :--- | :--- |
| 。 | ***2014 年 12 月：*** |
| DMV（动态管理视图） | V12 中增加了多个 DMV。有关详细信息，请参阅 [Azure SQL 数据库 Transact-SQL 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee336281.aspx)。 |
| 更改跟踪 | V12 完全支持更改跟踪。<br/><br/> 有关此功能的详细信息，请参阅[启用和禁用更改跟踪 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/bb964713.aspx)。 |
| 列存储索引 | 当索引列包含相同值的重复条目时，列存储索引可以改进数据仓库的系统性能。[列存储索引将压缩](http://msdn.microsoft.com/zh-cn/library/gg492088.aspx)重复值，以减少查询期间必须访问的物理行数。 |


### 高级服务层的性能改进


这些性能增强适用于高级服务层中的 P2 和 P3 级别。


| 功能 | 说明 |
| :--- | :--- |
| 。 | ***2014 年 12 月：*** |
| 查询的并行处理 | 对于可从并行度受益的查询，V12 提供更高的并行度。 |
| 更短的 I/O 延迟 | V12 明显缩短了输入/输出操作的延迟时间。 |
| 提高了 IOPS | V12 可以处理更大的每秒输入/输出数量 (IOPS)。 |
| 日志速率 | V12 每秒可以记录更多的数据更改。 |


| 功能 | 说明 |  
| :--- | :--- |  
| 。 | ***2015 年 8 月：*** |
| 使用 REST 或 PowerShell 创建服务器 | 当你在未指定服务器版本的情况下创建服务器时，默认版本将从 V11 更改为 V12。<br/><br/>这与 [Azure 门户](http://manage.windowsazure.cn)保持一致。 |  
| 使用 Transact-SQL、REST 或 PowerShell 创建数据库 | 在 V11 服务器上，当你在未指定版本或服务目标的情况下创建新的数据库时，默认的服务目标将是 S0 而不是 Web-and-Business。这与 V12 Azure 预览版门户中的服务器保持一致。 |  


### 增强功能摘要


V12 增强了 SQL 数据库，使其功能几乎完全与我们的 SQL Server 产品兼容。V12 注重最流行的功能和编程支持。这样就使得开发工作变得更高效且充满乐趣。现在，你可以更轻松地将 SQL 数据库应用程序迁移到云中。


此外，高级层 V12 为性能带来了重大改进。某些应用程序的查询速度将会得到极大提高。工作负载较重的应用程序可以提供可靠稳定的吞吐量。


## <a name="V12AzureSqlDbPreviewGaTable"></a>各区域的 V12 正式版 (GA) 发布状态


> [AZURE.NOTE]
> [Pricing](/home/features/sql-database/#price) 预览版期间的价格是折扣价。从 2015 年 3 月 31 日星期二开始，价格将回归到正式版的价位。


在已经推出正式版的任何区域，所有新订阅及其后续数据库将使用 V12 服务体系结构，因此可以访问最新功能。本文列出的是 V12 附带的新功能。


在正式版推出后，如果你使用的是低于 V12 版的服务器和数据库，你可以选择将数据库升级（或迁移）到 V12 服务体系结构。然后，你便可以在生产环境中使用新功能。V12 数据库必须位于基本、标准版或高级[定价层](/documentation/articles/sql-database-upgrade-new-service-tiers)。


## 如何继续下一步


你可以在[规划和准备升级到 Azure SQL 数据库 最新更新预览版](/documentation/articles/sql-database-preview-plan-prepare-upgrade)中了解如何将数据库从 Azure SQL 数据库 V11 升级到 V12。


类似于 V12 的版本号是指以下 Transact-SQL 语句返回的值：<br/> **SELECT @@version;**


- 11.0.9228.18 *(V11)*
- 12.0.2000.8 *(or a bit higher, V12)*


有关 V12 最新定价详细信息，请参阅 [SQL 数据库 定价](/home/features/sql-database/#price)。


## V12 预览版注意事项


在将 V11 数据库升级到 V12 期间和之后，需要了解一些有关限制的重要事项。你可以通过这个指向[规划和准备升级到 Azure SQL 数据库 V12](/documentation/articles/sql-database-preview-plan-prepare-upgrade#limitations) 主题*中间部分*的链接来了解详细信息。


[V12 general availability (GA) status per region]: #V12AzureSqlDbPreviewGaTable


<!-- EndOfFile -->

<!---HONumber=61-->