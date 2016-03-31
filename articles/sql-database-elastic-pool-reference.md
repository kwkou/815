<properties 
	pageTitle="Azure SQL 弹性数据库池参考" 
	description="本参考提供弹性数据库池文章和可编程性信息的链接和详情。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.date="12/01/2015" 
	wacn.date="03/18/2016"/>


# SQL 数据库弹性数据库池参考

对于拥有数十、数百甚至数千数据库的 SaaS 开发人员，则可以通过弹性数据库池来简化整个数据库组的创建、维护以及性能和成本管理流程。

本参考提供弹性数据库池文章和可编程性信息的链接和详情。

## 概述

弹性数据库池集合了弹性数据库吞吐量单位 (eDTU)，以及由多个数据库共享的存储 (GB)。可以随时在池中添加和删除弹性数据库。池中的弹性数据库只使用所需的池中资源，同时会释放可用资源供需要它们的活动数据库使用。有关确定数据库是否因弹性数据库池而受益的帮助，请参阅[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance)。



## 创建和管理弹性数据库池的先决条件


- 弹性数据库池只能在 Azure SQL 数据库 V12 服务器中使用。若要升级到 V12 并将数据库迁移直接到池中，请参阅[升级到 Azure SQL 数据库 V12](/documentation/articles/sql-database-upgrade-server-powershell)。
- 只有针对 Azure 资源管理器使用 [Azure 门户](https://manage.windowsazure.cn)、PowerShell 和 .NET 客户端库（REST API 的包装）才支持创建和管理弹性数据库池；[经典门户](https://manage.windowsazure.cn)和服务管理命令不受支持。
- 此外，还支持使用 Transact-SQL 创建新的弹性数据库，以及将现有数据库移入和移出弹性数据库池。



## 当前预览版注意事项


- 每个池的数据库数目和池 eDTU 存在上限：

    | 服务层 | 每个池的数据库数目上限* | 每个池的 eDTU 上限* |
    | :-- | :-- | :-- |
    | 基本 | 200 | 1200 | 
    | 标准 | 200 | 1200 |
    | 高级 | 50 | 1500 |

    ****每个池的数据库数和池 eDTU 数的当前限制预期会增加。***




## 文章列表

以下文章可帮助你着手使用弹性数据库和弹性作业：

| 文章 | 说明 |
| :-- | :-- |
| [SQL 数据库弹性数据库池](/documentation/articles/sql-database-elastic-pool) | 弹性数据库池的概述 |
| [价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance) | 如果评估使用弹性数据库池是否符合成本效益 |
| [使用 Azure 门户创建和管理 SQL 数据库弹性数据库池](/documentation/articles/sql-database-elastic-pool-portal) | 如何使用 Azure 门户创建和管理弹性数据库池 |
| [使用 PowerShell 创建和管理 SQL 数据库弹性数据库池](/documentation/articles/sql-database-elastic-pool-powershell) | 如何使用 PowerShell cmdlet 创建和管理弹性数据库池 |
| [使用适用于 .NET 的 Azure SQL 数据库库创建和管理 SQL 数据库](/documentation/articles/sql-database-elastic-pool-powershell) | 如何使用 C# 创建和管理弹性数据库池 |
| [弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview) | 概述弹性作业服务，该服务可用于跨池中的所有弹性数据库运行 T-SQL 脚本 |
| [安装弹性数据库作业组件](/documentation/articles/sql-database-elastic-jobs-service-installation) | 如何安装弹性数据库作业服务 |
| [为弹性作业服务创建所需的用户](/documentation/articles/sql-database-elastic-jobs-add-logins-to-dbs) | 若要运行弹性数据库作业脚本，必须将具有相应权限的用户添加到池中的每个数据库。 |
| [如何卸载弹性数据库作业组件](/documentation/articles/sql-database-elastic-jobs-uninstall) | 尝试安装弹性数据库作业服务时从故障中进行恢复 |



## 命名空间和终结点详细信息
弹性数据库池是 Azure SQL 数据库中“ElasticPool”类型的 Azure 资源管理器资源。

- **命名空间**：Microsoft.Sql/ElasticPool
- 用于 REST API 调用的**管理终结点**（资源管理器）：https://manage.windowsazure.cn



## 弹性数据库池属性

| 属性 | 说明 |
| :-- | :-- |
| creationDate | 创建池的日期。 |
| databaseDtuMax | 池中单一数据库可以使用的 eDTU 最大数目。数据库 eDTU 最大值不是资源保障。eDTU 最大值适用于池中的所有数据库。 |
| databaseDtuMin | 池中单一数据库可以确保获得的 eDTU 最小数目。数据库 eDTU 最小值可以设置为 0。eDTU 最小值适用于池中的所有数据库。请注意，池中数据库数目和数据库 eDTU 最小值的积不能超过池本身的 eDTU 数。 |
| Dtu | 池中所有数据库共享的 eDTU 数。 |
| edition | 池的服务层。池中的每个数据库都是此版本。 |
| elasticPoolId | 池实例的 GUID。 |
| elasticPoolName | 池的名称。该名称相对于其父服务器来说是唯一的。 |
| location | 创建池时所在的数据中心的位置。 |
| state | 如果订阅的帐单付款延迟，则状态为“禁用”，否则状态为“就绪”。 |
| storageMB | 池的存储限制 (MB)。池中任何单个的数据库都可以用到标准版本存储限制 (250 GB) 为止，但池中所有数据库所用的存储总量不能超过此池限制。 |


## 弹性池和弹性数据库的 eDTU 和存储限制

池的存储限制由池的 eDTU 量决定。

| 属性 | 基本 | 标准 | 高级 |
| :-- | :-- | :-- | :-- |
| dtu | **100**、200、400、800、1200 | **100**、200、400、800、1200 | **125**、250、500、1000、1500 |
| databaseDtuMax | **5** | 10、20、50、**100** | **125**、250、500、1000 |
| databaseDtuMin | **0**、5 | **0**、10、20、50、100 | **0**、125、250、500、1000 |
| storageMB* | **10000 MB**、20000 MB、40000 MB、80000 MB、120000 MB | **100 GB**、200 GB、400 GB、800 GB、1200 GB | **62.5 GB**、125 GB、250 GB、500 GB、750 GB |
| 每个 DTU 的存储 | 100 MB | 1 GB | 0\.5 GB |
| 每个池的数据库数目上限 | 200 | 200 | 50 |

默认值以**粗体**显示。

*API 的单位为 MB，而不是 GB。





## 辅助进程和会话限制

弹性池中所有数据库所支持的并发辅助进程和并发会话的最大数目取决于池的 eDTU 设置：

| eDTU | 最大并发辅助进程数 | 最大并发会话数 |
| :-- | :-- | :-- |
| 100（基本/标准）、125（高级） | 200 | 2,400 |
| 200（基本/标准）、250（高级） | 400 | 4,800 |
| 400（基本/标准）、500（高级） | 800 | 9,600 |
| 800（基本/标准）、1,000（高级） | 1,600 | 19,200 |
| 1,200（基本/标准）、1,500（高级） | 2,400 | 28,800 |


## Azure 资源管理器限制

Azure SQL 数据库 V12 服务器位于资源组中。

- 每个资源组最多可以有 800 个服务器。
- 每个服务器最多可以有 800 个弹性池。


## 弹性池操作延迟

- 更改单个数据库的保障 eDTU 数 (databaseDtuMin) 或单个数据库的最大 eDTU 数 (databaseDtuMax) 通常在 5 分钟或更少的时间内完成。 
- 如何更改池的 eDTU/存储限制 (storageMB) 取决于池中所有数据库使用的空间总容量。更改平均起来每 100 GB 需要 90 分钟或更短的时间。例如，如果池中所有数据库使用的总空间为 200 GB，则更改池的 eDTU/存储限制时，预计延迟为 3 小时或更短的时间。 



## PowerShell、REST API 和 .NET 客户端库

有几个 PowerShell cmdlet 和 REST API 命令可用于创建和管理弹性池。有关详细信息和代码示例，请参阅[使用 PowerShell 创建和管理 SQL 数据库弹性数据库池](/documentation/articles/sql-database-elastic-pool-powershell)和[使用 C# 创建和管理 SQL 数据库](/documentation/articles/sql-database-client-library)。


| [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt163521.aspx) | [REST API 命令](https://msdn.microsoft.com/zh-cn/library/mt163571.aspx) |
| :-- | :-- |
| [New-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt619378.aspx) | [创建弹性数据库池](https://msdn.microsoft.com/zh-cn/library/mt163596.aspx) |
| [Set-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt603511.aspx) | [设置弹性数据库池的性能设置](https://msdn.microsoft.com/zh-cn/library/mt163641.aspx) |
| [Remove-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt619355.aspx) | [删除弹性数据库池](https://msdn.microsoft.com/zh-cn/library/mt163672.aspx) |
| [Get-AzureRMSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt603517.aspx) | [获取弹性数据库池及其属性值](https://msdn.microsoft.com/zh-cn/library/mt163646.aspx) |
| [Get-AzureRmSqlElasticPoolActivity](https://msdn.microsoft.com/zh-cn/library/azure/mt603812.aspx) | [获取弹性数据库池的操作状态](https://msdn.microsoft.com/zh-cn/library/mt163669.aspx) |
| [Get-AzureRmSqlElasticPoolDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619484.aspx) | [获取弹性数据库池中的数据库](https://msdn.microsoft.com/zh-cn/library/mt163646.aspx) |
| [Get-AzureRmSqlDatabaseActivity]() | [获取将数据库移入和移出池的状态](https://msdn.microsoft.com/zh-cn/library/mt163669.aspx) |

## Transact-SQL

可以使用 Transact-SQL 执行以下的弹性数据库管理任务：

| 任务 | 详细信息 |
| :-- | :-- |
| 创建新的弹性数据库（直接在池中创建） | [CREATE DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx) |
| 将现有数据库移入和移出池 | [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms174269.aspx) |
| 获取池的资源使用情况统计信息 | [sys.elastic\_pool\_resource\_stats（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt280062.aspx) |


## 计费和定价信息

弹性数据库池按以下情况计费：

- 弹性池一创建即计费，即使池中没有数据库。 
- 弹性池按小时计费。该计量频率与单一数据库性能级别的计量频率相同。 
- 如果将弹性池的大小调整为新的 eDTU 量，则在调整操作完成之前，不会按新的 eDTU 量计费。这种计费所遵循的模式与更改独立数据库的性能级别所遵循的模式相同。 


- 弹性池的价格取决于池的 eDTU 数。弹性池的价格与池内弹性数据库的利用率无关。
- 价格的计算公式为：（池 eDTU 的数量）x（每 eDTU 的单位价格）。

弹性池的 eDTU 单价高于同一服务层中独立数据库的 DTU 单价。有关详细信息，请参阅 [SQL 数据库定价](/home/features/sql-database/#price)。

## 弹性数据库池错误

| ErrorNumber | ErrorSeverity | ErrorFormat | ErrorInserts | ErrorCause | ErrorCorrectiveAction |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 1132 | EX\_RESOURCE | 弹性池已达到其存储限制。弹性池的存储使用不能超过 (%d) MB。 | 弹性池空间限制 (MB)。 | 到达弹性池的存储限制时，尝试向数据库写入数据。 | 在可能的情况下，请考虑增加弹性池的 DTU 数，以便提高其存储限制，降低弹性池中各数据库使用的存储，或者从弹性池中删除数据库。 |
| 10929 | EX\_USER | %s 最小保证为 %d，最大限制为 %d，数据库的当前使用率为 %d。但是，服务器当前太忙，无法支持针对该数据库的数目大于 %d 的请求。请参阅 [http://go.microsoft.com/fwlink/?LinkId=267637](http://go.microsoft.com/fwlink/?LinkId=267637) 以获得帮助。否则，请稍后再试。 | 每个数据库的 DTU 最小值；每个数据库的 DTU 最大值 | 弹性池中所有数据库的并发辅助进程（请求）总数尝试超过池限制。 | 可能情况下，请考虑增加弹性池的 DTU 数，以便提高其辅助进程限制，或者从弹性池中删除数据库。 |
| 40844 | EX\_USER | 弹性池中数据库“%ls”（位于服务器“%ls”上）是“%ls”版本的数据库，不能有连续的复制关系。 | 数据库名称、数据库版本、服务器名称 | 针对弹性池中非高级数据库发出 StartDatabaseCopy 命令。 | 即将支持 |
| 40857 | EX\_USER | 找不到服务器“%ls”的弹性池，弹性池名称:“%ls”。 | 服务器名称；弹性池名称 | 指定的弹性池在指定的服务器中不存在。 | 请提供有效的弹性池名称。 |
| 40858 | EX\_USER | 弹性池“%ls”已存在于服务器“%ls”中 | 弹性池名称、服务器名称 | 指定的弹性池已存在于指定的逻辑服务器中。 | 提供新弹性池名称。 |
| 40859 | EX\_USER | 弹性池不支持服务层“%ls”。 | 弹性池服务层 | 进行弹性池设置时，不支持指定服务层。 | 提供正确的版本，或者将服务层留空以使用默认服务层。 |
| 40860 | EX\_USER | 弹性池“%ls”和服务目标“%ls”的组合无效。 | 弹性池名称；服务级别目标名称 | 仅当服务目标指定为 ‘ElasticPool’ 的情况下，才能一起指定弹性池和服务目标。 | 请指定正确的弹性池和服务目标组合。 |
| 40861 | EX\_USER | 数据库版本“%.ls”不能不同于弹性池服务层“%.ls”。| 数据库版本、弹性池服务层 | 数据库版本不同于弹性池服务层。| 请勿指定不同于弹性池服务层的数据库版本。注意，数据库版本不需要指定。|
| 40862 | EX\_USER | 如果指定了弹性池服务目标，则必须指定弹性池名称。| 无 | 弹性池服务目标没有唯一地标识弹性池。| 如果使用弹性池服务目标，请指定弹性池名称。|
| 40864 | EX\_USER | 对于服务层“%.*ls”来说，弹性池的 DTU 数必须至少为 (%d) 个 DTU。| 弹性池的 DTU 数；弹性池服务层。| 尝试将弹性池的 DTU 数设置为最小限制以下。| 请重新尝试将弹性池的 DTU 数至少设置为最小限制。|
| 40865 | EX\_USER | 对于服务层“%.*ls”来说，弹性池的 DTU 数不能超过 (%d) 个 DTU。| 弹性池的 DTU 数；弹性池服务层。| 尝试将弹性池的 DTU 数设置为高出最大限制。| 请重新尝试将弹性池的 DTU 数设置为不超过最大限制。|
| 40867 | EX\_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最大值必须至少为 (%d)。| 每个数据库的 DTU 最大值；弹性池服务层 | 尝试将每个数据库的 DTU 最大值设置为支持的限制以下。| 请考虑使用支持所需设置的弹性池服务层。|
| 40868 | EX\_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最大值不能超过 (%d)。| 每个数据库的 DTU 最大值；弹性池服务层。| 尝试将每个数据库的 DTU 最大值设置为超出支持的限制。| 请考虑使用支持所需设置的弹性池服务层。|
| 40870 | EX\_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最小值不能超出 (%d)。| 每个数据库的 DTU 最小值；弹性池服务层。| 尝试将每个数据库的 DTU 最小值设置为超出支持的限制。| 请考虑使用支持所需设置的弹性池服务层。|
| 40873 | EX\_USER | 数据库数目 (%d) 和每个数据库的 DTU 最小值 (%d) 不能超过弹性池的 DTU 数 (%d)。| 弹性池中的数据库数；每个数据库的 DTU 最小值；弹性池的 DTU 数。| 尝试指定弹性池中数据库的 DTU 最小值，该最小值超出弹性池的 DTU 数。| 请考虑增加弹性池的 DTU 数，或者降低每个数据库的 DTU 最小值，或者降低弹性池中数据库的数目。|
| 40877 | EX\_USER | 除非弹性池不含任何数据库，否则不能将其删除。| 无 | 弹性池包含一个或多个数据库，因此无法将其删除。| 请删除弹性池中的数据库，以便删除弹性池。|
| 40881 | EX\_USER | 弹性池“%.*ls”已达到其数据库计数限制。弹性池的数据库计数限制不能超出 (%d) 是针对 DTU 数为 (%d) 的弹性池的。| 弹性池名称；弹性池的数据库计数限制；资源池的 DTU 数。| 达到弹性池的数据库计数限制时，尝试创建数据库或将其添加到弹性池。| 可能情况下，请考虑增加弹性池的 DTU 数，以便提高其数据库限制，或者从弹性池中删除数据库。|
| 40889 | EX\_USER | 弹性池“%.*ls”的 DTU 数或存储限制不能降低，因为这样就无法为其数据库提供足够的存储空间。| 弹性池的名称。| 尝试将弹性池的存储限制降低到其存储使用量以下。| 请考虑降低弹性池中各个数据库的存储使用量，或者从池中删除数据库以降低其 DTU 或存储限制。|
| 40891 | EX\_USER | 每个数据库的 DTU 最小值 (%d) 不能超过每个数据库的 DTU 最大值 (%d)。| 每个数据库的 DTU 最小值；每个数据库的 DTU 最大值。| 尝试将每个数据库的 DTU 最小值设置为高于每个数据库的 DTU 最大值。| 请确保每个数据库的 DTU 最小值不超过每个数据库的 DTU 最大值。|
| 待定 | EX\_USER | 弹性池中每个数据库的存储大小不能超过“%.*ls”服务层弹性池所允许的最大大小。| 弹性池服务层 | 数据库的最大大小超过弹性池服务层允许的最大大小。| 请将数据库的最大大小设置为处于弹性池服务层允许的最大大小限制范围内。|

<!---HONumber=Mooncake_0118_2016-->
