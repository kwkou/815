<properties 
	pageTitle="Azure SQL 弹性数据库池参考" 
	description="本参考提供弹性数据库池文章和可编程性信息的链接和详情。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" ms.date="04/29/2015" wacn.date="" />


# SQL Database 弹性数据库池参考（预览版）

对于拥有数十、数百甚至数千数据库的 SaaS 开发人员，则可以通过弹性数据库池来简化整个数据库组的创建、维护以及性能和成本管理流程。

本参考提供弹性数据库池文章和可编程性信息的链接和详情。

## 概述

弹性数据库池集合了数据库吞吐量单位 (DTU)，以及由多个数据库共享的存储 (GB)。可以随时在池中添加和删除弹性数据库。池中的弹性数据库只使用所需的池中资源，同时会释放可用资源供需要它们的活动数据库使用。



## 创建和管理弹性数据库池的先决条件


- 弹性池只能在 Azure SQL Database V12 服务器中使用。   
- [Microsoft Azure 门户](https://manage.windowsazure.cn)支持创建和管理弹性池。 


## 当前预览版的限制

- 当前预览版中弹性数据库池的定价层为“标准”。  
- 不支持直接将数据库导入池。你可以先导入到独立的数据库中，然后将该数据库移到池中。支持从池中导出数据库。
- 每个池最多可以有 100 个数据库。


## 文章列表

以下文章可帮助你着手使用弹性数据库和弹性作业：

| 文章 | 说明 |
| :-- | :-- |
| [SQL Database 弹性数据库池](sql-database-elastic-pool) | 弹性池概述 |
| [使用 Azure 门户创建和管理 SQL Database 弹性数据库池](sql-database-elastic-pool-portal) | 如何使用 Azure 门户创建和管理弹性池 |
| [使用 PowerShell 创建和管理 SQL Database 弹性数据库池](sql-database-elastic-pool-powershell) | 如何使用 PowerShell cmdlet 创建和管理弹性池 |
| [弹性数据库作业概述](sql-database-elastic-jobs-overview) | 概述弹性作业服务，该服务可用于跨池中的所有弹性数据库运行 T-SQL 脚本 |
| [安装弹性数据库作业组件](sql-database-elastic-jobs-service-installation) | 如何安装弹性数据库作业服务 |
| [为弹性作业服务创建所需的用户](sql-database-elastic-jobs-add-logins-to-dbs) | 若要运行弹性数据库作业脚本，必须将具有相应权限的用户添加到池中的每个数据库。 |
| [如何卸载弹性数据库作业组件](sql-database-elastic-jobs-uninstall) | 尝试安装弹性数据库作业服务时从故障中进行恢复 |



## 命名空间和终结点详细信息
弹性池是 Microsoft Azure SQL Database 中类型为“ElasticPool”的 ARM 资源。

- **命名空间**：Microsoft.Sql/ElasticPool
- 用于 REST API 调用的**管理终结点**（Azure 资源管理器）：https://management.windowsazure.cn



## 弹性数据库池属性

| 属性 | 说明 |
| :-- | :-- |
| creationDate | 创建池的日期。 |
| databaseDtuMax | 池中单个数据库可以使用的 DTU 最大数目。数据库 DTU 最大值不是资源保障。DTU 最大值适用于池中的所有数据库。 |
| databaseDtuMin | 池中单个数据库可以确保获得的 DTU 最小数目。数据库 DTU 最小值可以设置为 0。DTU 最小值适用于池中的所有数据库。请注意，池中数据库数目和数据库 DTU 最小值的积不能超过池本身的 DTU 数。 |
| Dtu | 池中所有数据库共享的 DTU 数。 |
| edition | 池的服务层。池中的每个数据库都是此版本。 |
| elasticPoolId | 池实例的 GUID。 |
| elasticPoolName | 池的名称。该名称相对于其父服务器来说是唯一的。 |
| location | 创建池时所在的数据中心的位置。 |
| state | 如果订阅的帐单付款延迟，则状态为“禁用”，否则状态为“就绪”。 |
| storageMB | 池的存储限制 (MB)。池中任何单个的数据库都可以用到标准版本存储限制 (250 GB) 为止，但池中所有数据库所用的存储总量不能超过此池限制。 |


## 弹性池和弹性数据库的 DTU 和存储限制

池的存储限制决定于池的 DTU 量；每个 DTU = 1 GB 存储。例如，一个 200 DTU 的池的存储限制为 200 GB。

| 属性 | 默认值 | 有效值 |
| :-- | :-- | :-- |
| Dtu | 100 | 100、200、400、800、1200 |
| databaseDtuMax | 100 | 10、20、50、100 |
| databaseDtuMin | 0 | 0、10、20、50 |
| storageMB | 100 GB* | 100 GB、200 GB、400 GB、800 GB、1200 GB |

*API 中的单位为 MB，而不是 GB

## 辅助进程和会话限制

弹性池中所有数据库所支持的并发辅助进程和并发会话的最大数目取决于池的 DTU 设置：

| DTU | 最大并发辅助进程数 | 最大并发会话数 |
| :-- | :-- | :-- |
| 100 | 200 | 2,400 |
| 200 | 400 | 4,800 |
| 400 | 800 | 9,600 |
| 800 | 1,600 | 19,200 |
| 1,200 | 2,400 | 28,800 |


## Azure 资源管理器限制

Azure SQL Database V12 服务器位于资源组中。

- 每个资源组最多可以有 800 个服务器。
- 每个服务器最多可以有 800 个弹性池。



## 弹性池操作延迟

- 更改单个数据库的保障 DTU 数 (databaseDtuMin) 或单个数据库的最大 DTU 数 (databaseDtuMax) 通常在 5 分钟或更少的时间内完成。 
- 如何更改池的 DTU/存储限制 (storageMB) 取决于池中所有数据库使用的空间总容量。更改平均起来每 100 GB 需要 90 分钟或更短的时间。例如，如果池中所有数据库使用的总空间为 200 GB，则更改池的 DTU/存储限制时，预计延迟为 3 小时或更短的时间。 



## 弹性数据库池 PowerShell cmdlet 和 REST API 命令（仅限 Azure 资源管理器）

以下 PowerShell cmdlet 和 REST API 命令可用于创建和管理弹性池：

| [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt125356.aspx) | [REST API 命令](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx) |
| :-- | :-- |
| Get-AzureSqlDatabase | 获取 Azure SQL 数据库 |
| Get-AzureSqLElasticPool | 获取 Azure SQL Database 弹性数据库池 |
| Get-AzureSqlElasticPoolActivity | 获取 Azure SQL Database 弹性数据库池操作 |
| Get-AzureSqlElasticPoolDatabase | 获取 Azure SQL Database 弹性数据库 |
| Get-AzureSqlElasticPoolDatabaseActivity | 获取 Azure SQL Database 弹性数据库操作 |
| Get-AzureSqlServer | 获取 Azure SQL Database 服务器 |
| Get-AzureSqlServerFirewallRule | 获取 Azure SQL Database 服务器防火墙规则 |
| Get-AzureSqlServerServiceObjective | 获取 Azure SQL Database 服务器服务目标 |
| New-AzureSqlDatabase | 创建 Azure SQL 数据库 |
| New-AzureSqlElasticPool | 创建 Azure SQL Database 弹性数据库池 |
| New-AzureSqlServer | 创建 Azure SQL Database 服务器 |
| New-AzureSqlServerFirewallRule | 创建 Azure SQL Database 服务器防火墙规则 |
| Remove-AzureSqlDatabase | 删除 Azure SQL 数据库 |
| Remove-AzureSqlElasticPool | 删除 Azure SQL Database 弹性数据库池 |
| Remove-AzureSqlServer | 删除 Azure SQL Database 服务器 |
| Set-AzureSqlDatabase | 设置 Azure SQL 数据库 |
| Set-AzureSqlElasticPool | 设置 Azure SQL Database 弹性数据库池 |
| Set-AzureSqlServer | 设置 Azure SQL Database 服务器 |
| Set-AzureSqlServerFirewallRule | 设置 Azure SQL Database 服务器防火墙规则 |
| Get-Metrics | 获取度量值 |


## 计费和定价信息

弹性数据库池按以下情况计费：

- 弹性池一创建即计费，即使池中没有数据库。 
- 弹性池按小时计费。该计量频率与独立数据库性能级别的计量频率相同。 
- 如果将弹性池的大小调整为新的 DTU 量，则在调整操作完成之前，不会按新的 DTU 量计费。这种计费所遵循的模式与更改独立数据库的性能级别所遵循的模式相同。 


- 弹性池的价格取决于池的 DTU 数和池中数据库的数量。
- 价格按 (池 DTU 数)x(每个 DTU 的单价) + (数据库数)x(每个数据库的单价) 计算

弹性池的 DTU 单价高于同一服务层中独立数据库的 DTU 单价。有关详细信息，请参阅 [SQL Database 定价](/home/features/sql-database/#price)。

## 弹性数据库池错误

| ErrorNumber | ErrorSeverity | ErrorFormat | ErrorInserts | ErrorCause | ErrorCorrectiveAction |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 1132 | EX_RESOURCE | 弹性池已达到其存储限制。弹性池的存储使用不能超过 (%d) MB。 | 弹性池空间限制 (MB)。 | 到达弹性池的存储限制时，尝试向数据库写入数据。 | 在可能的情况下，请考虑增加弹性池的 DTU 数，以便提高其存储限制，降低弹性池中各数据库使用的存储，或者从弹性池中删除数据库。 |
| 10929 | EX_USER | %s 最小保证为 %d，最大限制为 %d，数据库的当前使用率为 %d。但是，服务器当前太忙，无法支持针对该数据库的数目大于 %d 的请求。如需帮助，请参阅 [https://msdn.microsoft.com/zh-cn/library/dn338081.aspx](https://msdn.microsoft.com/zh-cn/library/dn338081.aspx)。否则，请稍后再试。 | 每个数据库的 DTU 最小值；每个数据库的 DTU 最大值 | 弹性池中所有数据库的并发辅助进程（请求）总数尝试超过池限制。 | 可能情况下，请考虑增加弹性池的 DTU 数，以便提高其辅助进程限制，或者从弹性池中删除数据库。 |
| 40844 | EX_USER | 弹性池中数据库“%ls”（位于服务器“%ls”上）是“%ls”版本的数据库，不能有连续的复制关系。 | 数据库名称、数据库版本、服务器名称 | 针对弹性池中非高级数据库发出 StartDatabaseCopy 命令。 | 即将支持 |
| 40857 | EX_USER | 找不到服务器“%ls”的弹性池，弹性池名称:“%ls”。 | 服务器名称；弹性池名称 | 指定的弹性池在指定的服务器中不存在。 | 请提供有效的弹性池名称。 |
| 40858 | EX_USER | 弹性池“%ls”已存在于服务器“%ls”中 | 弹性池名称、服务器名称 | 指定的弹性池已存在于指定的逻辑服务器中。 | 提供新弹性池名称。 |
| 40859 | EX_USER | 弹性池不支持服务层“%ls”。 | 弹性池服务层 | 进行弹性池设置时，不支持指定服务层。 | 提供正确的版本，或者将服务层留空以使用默认服务层。 |
| 40860 | EX_USER | 弹性池“%ls”和服务目标“%ls”的组合无效。 | 弹性池名称；服务级别目标名称 | 仅当服务目标指定为 ‘ElasticPool’ 的情况下，才能一起指定弹性池和服务目标。 | 请指定正确的弹性池和服务目标组合。 |
| 40861 | EX_USER | 数据库版本“%.*ls”不能不同于弹性池服务层“%.*ls”。 | 数据库版本、弹性池服务层 | 数据库版本不同于弹性池服务层。 | 请勿指定不同于弹性池服务层的数据库版本。请注意，数据库版本不需要指定。 |
| 40862 | EX_USER | 如果指定了弹性池服务目标，则必须指定弹性池名称。 | 无 | 弹性池服务目标没有唯一地标识弹性池。 | 如果使用弹性池服务目标，请指定弹性池名称。 |
| 40864 | EX_USER | 对于服务层“%.*ls”来说，弹性池的 DTU 数必须至少为 (%d) 个 DTU。 | 弹性池的 DTU 数；弹性池服务层。 | 尝试将弹性池的 DTU 数设置为最小限制以下。 | 请重新尝试将弹性池的 DTU 数至少设置为最小限制。 |
| 40865 | EX_USER | 对于服务层“%.*ls”来说，弹性池的 DTU 数不能超过 (%d) 个 DTU。 | 弹性池的 DTU 数；弹性池服务层。 | 尝试将弹性池的 DTU 数设置为高出最大限制。 | 请重新尝试将弹性池的 DTU 数设置为不超过最大限制。 |
| 40867 | EX_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最大值必须至少为 (%d)。 | 每个数据库的 DTU 最大值；弹性池服务层。 | 尝试将单个数据库的 DTU 最大值设置为低于支持的限制。 | 请考虑使用支持所需设置的弹性池服务层。 |
| 40868 | EX_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最大值不能超过 (%d)。 | 每个数据库的 DTU 最大值；弹性池服务层。 | 尝试将单个数据库的 DTU 最大值设置为超出支持的限制。 | 请考虑使用支持所需设置的弹性池服务层。 |
| 40870 | EX_USER | 对于服务层“%.*ls”来说，每个数据库的 DTU 最小值不能超过 (%d)。 | 每个数据库的 DTU 最小值；弹性池服务层。 | 尝试将单个数据库的 DTU 最小值设置为超出支持的限制。 | 请考虑使用支持所需设置的弹性池服务层。 |
| 40873 | EX_USER | 数据库数目 (%d) 和每个数据库的 DTU 最小值 (%d) 不能超过弹性池的 DTU 数 (%d)。 | 弹性池中的数据库数；每个数据库的 DTU 最小值；弹性池的 DTU 数。 | 尝试指定弹性池中数据库的 DTU 最小值，该最小值超出弹性池的 DTU 数。 | 请考虑增加弹性池的 DTU 数，或者降低单个数据库的 DTU 最小值，或者降低弹性池中数据库的数目。 |
| 40877 | EX_USER | 除非弹性池不含任何数据库，否则不能将其删除。 | 无 | 弹性池包含一个或多个数据库，因此无法将其删除。 | 请删除弹性池中的数据库，以便删除弹性池。 |
| 40881 | EX_USER | 弹性池“%.*ls”已达到其数据库计数限制。弹性池的数据库计数限制不能超出 (%d) 是针对 DTU 数为 (%d) 的弹性池的。 | 弹性池名称；弹性池的数据库计数限制；资源池的 DTU 数。 | 达到弹性池的数据库计数限制时，尝试创建数据库或将其添加到弹性池。 | 可能情况下，请考虑增加弹性池的 DTU 数，以便提高其数据库限制，或者从弹性池中删除数据库。 |
| 40889 | EX_USER | 弹性池“%.*ls”的 DTU 数或存储限制不能降低，因为这样就无法为其数据库提供足够的存储空间。 | 弹性池的名称。 | 尝试将弹性池的存储限制降低到其存储使用量以下。 | 请考虑降低弹性池中各个数据库的存储使用量，或者从池中删除数据库以降低其 DTU 或存储限制。 |
| 40891 | EX_USER | 每个数据库的 DTU 最小值 (%d) 不能超过每个数据库的 DTU 最大值 (%d)。 | 每个数据库的 DTU 最小值；每个数据库的 DTU 最大值。 | 尝试将单个数据库的 DTU 最小值设置为高于单个数据库的 DTU 最大值。 | 请确保单个数据库的 DTU 最小值不超过单个数据库的 DTU 最大值。 |
| TBD | EX_USER | 弹性池中单个数据库的存储大小不能超过“%.*ls”服务层弹性池所允许的最大大小。 | 弹性池服务层 | 数据库的最大大小超过弹性池服务层允许的最大大小。 | 请将数据库的最大大小设置为处于弹性池服务层允许的最大大小限制范围内。 |

<!---HONumber=66-->