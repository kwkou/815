<properties
	pageTitle="管理扩大的云数据库 | Azure" 
	description="安装弹性数据库作业服务" 
	metaKeywords="azure sql database elastic databases" 
	services="sql-database" 
    documentationCenter=""  
	manager="jhubbard" 
	authors="ddove"/>

<tags 
	ms.service="sql-database" 
	ms.date="04/25/2016" 
	wacn.date="06/14/2016" />

# 管理扩大的云数据库

若要管理扩大的分区数据库，可使用**弹性数据库作业**功能（预览版）在一组数据库中可靠地执行 Transact-SQL (T-SQL) 脚本或应用 DACPAC（[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)），这些数据库包括：

* 自定义的数据库集合（下面将会介绍）
* [弹性数据库池](/documentation/articles/sql-database-elastic-pool/)中的所有数据库
* 分片集（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)创建） 
 
## 文档

* [安装弹性数据库作业组件](/documentation/articles/sql-database-elastic-jobs-service-installation/)。 
* [弹性数据库作业入门](/documentation/articles/sql-database-elastic-jobs-getting-started/)。
* [使用 Powershell 创建和管理作业](/documentation/articles/sql-database-elastic-jobs-powershell/)。
* [创建和管理扩大的 SQL Azure 数据库](/documentation/articles/sql-database-elastic-jobs-getting-started/)

**弹性数据库作业**是当前的客户托管 Azure 云服务，可以让你执行即席任务和计划的管理任务，称为**作业**。使用作业可以通过运行 Transact-SQL 脚本来执行管理操作，从而轻松可靠地管理大型 Azure SQL 数据库组。

![弹性数据库作业服务][1]

## 为何使用作业？

**管理**

轻松执行架构更改、凭据管理、引用数据更新、性能数据收集或租户（客户）遥测数据收集。

**报告**

将 Azure SQL 数据库集合中的数据聚合到单个目标表中。

**减少开销**

通常，你必须单独连接到每个数据库才能运行 Transact-SQL 语句或执行其他管理任务。作业可处理登录到目标组中每个数据库的任务。此外，可定义、维护及保存要在一组 Azure SQL 数据库中执行的 Transact-SQL 脚本。

**计帐**

作业运行脚本并记录每个数据库的执行状态。失败时也会自动重试。

**灵活性**

定义 Azure SQL 数据库的自定义组，并定义作业运行计划。

**部署**

部署数据层应用程序 (DACPAC)。


## 应用程序 

* 执行管理任务，例如部署新架构。
* 更新引用数据，例如，所有数据库的公用产品信息。或安排每个工作日数小时后自动更新。
* 重建索引以提升查询性能。可以配置重建操作，以便定期（例如，在非高峰时段）对一系列数据库执行查询。
* 持续将一组数据库中的查询结果收集到中央表中。性能查询可以持续执行，并可配置为触发执行其他任务。
* 对大量的数据库执行长时间运行的数据处理查询，例如，收集客户遥测数据。结果将收集到单个目标表以供进一步分析。

## 弹性数据库作业：端到端 
1.	安装**弹性数据库作业**组件。有关详细信息，请参阅[安装弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-service-installation/)。如果安装失败，请参阅[如何卸载](/documentation/articles/sql-database-elastic-jobs-uninstall/)。
2.	使用 PowerShell API 可以访问更多功能，例如创建自定义数据库集合、添加计划和/或收集结果集。使用门户可以方便安装和创建/监视限制为针对**弹性数据库池**执行的作业。 
3.	针对作业执行创建加密的凭据，并[将用户（或角色）添加到组中的每个数据库](/documentation/articles/sql-database-security/)。
4.	创建可针对组中每个数据库运行的幂等 T-SQL 脚本。 
5.	使用 PowerShell 脚本：[使用 PowerShell 创建和管理 SQL 数据库弹性数据库作业（预览版）](/documentation/articles/sql-database-elastic-jobs-powershell/)。

## 幂等脚本

脚本必须是[幂等的](https://en.wikipedia.org/wiki/Idempotence)。简单而言，“幂等”是指如果脚本成功，则再次运行时，会出现相同的结果。脚本可能由于暂时性网络问题而失败。在此情况下，作业将自动重试运行脚本，达到默认的次数才停止。即使幂等脚本已成功运行两次，也仍会返回相同的结果。

一个简单的策略是在创建对象之前测试其是否存在。

	IF NOT EXIST (some_object)
	-- Create the object 
	-- If it exists, drop the object before recreating it.

同样地，脚本必须以逻辑方式测试并反驳它所找到的任何条件，才能成功执行。

## 失败和日志

如果脚本在多次尝试之后失败，作业将会记录错误并继续。在作业结束后（即已针对组中的所有数据库运行），你可以检查其失败的尝试列表。日志提供详细信息用于调试错误脚本。

## 组类型和创建

有两种类型的组：

1. 分片集
2. 自定义组

分片集组是使用[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-introduction/)创建的。当你创建分片集组时，将在组中自动添加或删除数据库。例如，当你将新分片添加到分片映射中时，该分片会自动出现在组中。然后就可以对组运行作业。

另一方面，自定义组的定义方式很严格。你必须在自定义组中显式添加或删除数据库。如果组中的数据库已被删除，作业将尝试对最终导致失败的数据库运行脚本。


## 组件和定价
 
以下组件配合工作可以创建支持即席执行管理作业的 Azure 云服务。在安装期间，这些组件将自动在你的订阅中安装和配置。你可以识别这些服务，因为它们具有相同的自动生成名称。名称是唯一的，包括前缀“edj”后接 21 个随机生成的字符。

* **Azure 云服务**：弹性数据库作业（预览版）以客户托管的 Azure 云服务交付，可执行请求的任务。从门户开始，服务部署并托管在你的 Azure 订阅中。默认部署的服务将结合最少两个辅助角色运行，以实现高可用性。每个默认大小的辅助角色 (ElasticDatabaseJobWorker) 在 A0 实例上运行。有关价格，请参阅[云服务定价](/home/features/cloud-services/pricing/)。 
* **Azure SQL 数据库**：服务使用名为**控制数据库**的 Azure SQL 数据库来存储所有的作业元数据。默认的服务层是 S0。有关价格，请参阅 [SQL 数据库定价](/home/features/sql-database/pricing/)。
* **Azure Service Bus**：Azure Service Bus 用于协调 Azure 云服务中的工作。请参阅 [Service Bus 定价](/home/features/messaging/pricing/)。
* **Azure 存储空间**：在某个问题需要进一步调试时，使用 Azure 存储帐户来存储诊断输出日志记录（请参阅[在 Azure 云服务和虚拟机中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)）。有关价格，请参阅 [Azure 存储空间定价](/home/features/storage/pricing/)。

## 弹性数据库作业的工作原理

1.	Azure SQL 数据库是一个**控制数据库**，用于存储所有元数据和状态数据。
2.	在启动和跟踪要执行的作业时，**作业服务**将访问该控制数据库。
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
[安装组件](/documentation/articles/sql-database-elastic-jobs-service-installation/)，然后[创建一个登录名并将其添加到数据库组的每个数据库中](/documentation/articles/sql-database-security/)。另请参阅[弹性数据库作业入门](/documentation/articles/sql-database-elastic-jobs-getting-started/)。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-overview/elastic-jobs.png
<!--anchors-->

 

<!---HONumber=Mooncake_0530_2016-->
