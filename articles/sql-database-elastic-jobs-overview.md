<properties
	title="Elastic database jobs overview"
	pageTitle="弹性数据库作业概述"
	description="安装弹性数据库作业服务"
	metaKeywords="azure SQL 数据库 elastic databases"
	services="sql-database" documentationCenter=""  
	manager="jeffreyg" 
	authors="sidneyh"/>

<tags 
	ms.service="sql-database" 
	ms.date="07/21/2015" 
	wacn.date="09/15/2015" />

# 弹性数据库作业概述

**弹性数据库作业**功能（预览版）可让你可靠地执行 Transact-SQL (T-SQL) 脚本或跨数据库组应用 DACPAC，该组包含自定义的数据库集合、[弹性数据库池（预览版）](/documentation/articles/sql-database-elastic-pool)中的所有数据库，或分片集合（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library)创建）。在预览版中，**弹性数据库作业**是当前的客户托管 Azure 云服务，可以让你执行即席任务和计划的管理任务，称为“作业”。使用这个功能，你可以轻松可靠地大规模管理 Azure SQL 数据库中的所有数据库，方法是运行 Transact-SQL 脚本来执行管理操作，例如架构更改、凭据管理、引用数据更新、性能数据收集或租户（客户）遥测数据收集。通常，你必须单独连接到每个数据库才能运行 Transact-SQL 语句或执行其他管理任务。**弹性数据库作业**会处理登录任务，并可靠地运行脚本，同时还会记录每个数据库的执行状态。有关安装说明，请转到[安装弹性数据库作业组件](/documentation/articles/sql-database-elastic-jobs-service-installation)。

![弹性数据库作业服务][1]

## 优点
* 定义 Azure SQL 数据库的自定义组
* 定义、维护及保存要在 Azure SQL 数据库组中执行的 Transact-SQL 脚本 
* 部署数据层应用程序 (DACPAC)
* 使用自动重试大规模且可靠地执行 Transact-SQL 脚本或应用 DACPAC
* 跟踪作业执行状态
* 定义执行计划
* 在 Azure SQL 数据库集合中执行数据收集，并将结果保存到单个目标表中

> [AZURE.NOTE]**弹性数据库作业**功能在 Azure 门户中呈现精简功能集，也限制为 SQL Azure 弹性池。使用 PowerShell API 可以访问当前功能的完整集。

## 方案

* 性能管理任务，例如部署新架构
* 更新引用数据（例如所有数据库通用的产品信息），甚至使用计划在每个工作日的几个小时后自动进行更新。
* 重建索引以提升查询性能。可以配置重建操作，以便定期（例如，在非高峰时段）对一系列数据库执行查询。
* 持续将一组数据库中的查询结果收集到中央表中。性能查询可以持续执行，并可配置为触发执行其他任务。
* 对大量的数据库执行长时间运行的数据处理查询，例如，收集客户遥测数据。结果将收集到单个目标表以供进一步分析。

## 弹性数据库作业的简单端到端复查
1.	安装**弹性数据库作业**组件。有关详细信息，请参阅[安装弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-service-installation)。如果安装失败，请参阅[如何卸载](/documentation/articles/sql-database-elastic-jobs-uninstall)。
2.	使用 PowerShell API 可以访问更多功能，例如创建自定义数据库集合、添加计划和/或收集结果集。使用门户可以方便安装和创建/监视限制为针对**弹性数据库池**执行的作业。 
3.	针对作业执行创建加密的凭据，并[将用户（或角色）添加到组中的每个数据库](/documentation/articles/sql-database-elastic-jobs-add-logins-to-dbs)。
4.	创建可针对组中每个数据库运行的幂等 T-SQL 脚本。
5.	遵循以下步骤来运行该脚本：[创建和管理弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-create-and-manage) 

## 组件和定价 
以下组件配合工作可以创建支持即席执行管理作业的 Azure 云服务。在安装期间，这些组件将自动在你的订阅中安装和配置。你可以识别这些服务，因为它们具有相同的自动生成名称。名称是唯一的，包括前缀“edj”后接 21 个随机生成的字符。

* **Azure 云服务**：弹性数据库作业（预览版）以客户托管的 Azure 云服务交付，可执行请求的任务。服务可从门户部署，并托管在你的 Windows Azure 订阅中。默认部署的服务将结合最少两个辅助角色运行，以实现高可用性。每个默认大小的辅助角色 (ElasticDatabaseJobWorker) 在 A0 实例上运行。有关价格，请参阅[云服务定价](/home/features/cloud-services/#price)。
* **Azure SQL 数据库**：服务使用名为**控制数据库**的 Azure SQL 数据库 来存储所有的作业元数据。默认的服务层是 S0。有关价格，请参阅 [SQL 数据库 定价](/home/features/sql-database/#price)。
* **Azure Service Bus**：Azure Service Bus 用于协调 Azure 云服务中的工作。请参阅 [Service Bus 定价](/home/features/messaging/#price)。
* **Azure 存储空间**：在某个问题需要进一步调试时，将使用 Azure 存储帐户来存储诊断输出（[Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics)的常见做法）。有关价格，请参阅 [Azure 存储空间定价](/home/features/storage/#price)。

## 弹性数据库作业的工作原理
1.	Azure SQL 数据库是一个控制数据库，用于存储所有元数据和状态数据。
2.	在启动和跟踪要执行的作业时，**弹性数据库作业**将访问该控制数据库。
3.	与控制数据库通信的两个不同角色： 
	* 控制器：确定哪些作业要求任务执行请求的作业，并通过创建新的作业任务来重试失败的作业。
	* 作业任务执行：执行作业任务。

### 作业任务类型
有多种类型的作业任务会执行作业：

* ShardMapRefresh：查询分片映射，以确定用作分片的所有数据库
* ScriptSplit：通过“GO”语句将脚本拆分为多个批
* ExpandJob：基于面向一组数据库的作业创建每个数据库的子作业
* ScriptExecution：使用定义的凭据针对特定数据库执行脚本
* Dacpac：使用特定的凭据将 DACPAC 应用到特定的数据库

## 端到端作业执行工作流
1.	使用门户或 PowerShell API 将作业插入**控制数据库**。作业将使用特定的凭据，请求针对一组数据库执行 Transact-SQL 脚本。
2.	控制器将识别新作业。创建并执行作业任务，以拆分脚本并刷新组的数据库。最后，将创建并执行新的作业以扩展该作业，并创建新的作业，其中指定了每个子作业针对据组中的单个数据库执行 Transact-SQL 脚本。
3.	控制器将识别创建的子作业。对于每个作业，控制器将创建并触发作业任务，以针对数据库执行脚本。 
4.	完成所有作业任务后，控制器会将作业更新为已完成状态。在作业执行期间，可以随时使用 PowerShell API 来查看作业执行的当前状态。PowerShell API 返回的所有时间都以 UTC 表示。如果需要，你可以启动取消请求来停止作业。 

## 后续步骤
[安装组件](/documentation/articles/sql-database-elastic-jobs-service-installation)，然后[创建一个登录名并将其添加到池中的每个数据库](/documentation/articles/sql-database-elastic-jobs-add-logins-to-dbs)。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-overview/elastic-jobs.png
<!--anchors-->

<!---HONumber=69-->