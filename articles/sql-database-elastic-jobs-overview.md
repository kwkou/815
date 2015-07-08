<properties
	title="Elastic database jobs overview"
	pageTitle="弹性数据库作业概述"
	description="安装弹性数据库作业服务"
	metaKeywords="azure SQL 数据库 elastic databases"
	services="sql-database" documentationCenter=""  
	manager="jeffreyg"
	authors="sidneyh"/>

<tags ms.service="sql-database" ms.date="04/23/2015" wacn.date="06/23/2015"/>

# 弹性数据库作业概述

使用**弹性数据库作业**（预览版）可对[弹性数据库池（预览版）](sql-database-elastic-pool)中的所有数据库运行 T-SQL（作业）。例如，可以轻松更新每个数据库中的架构以包含新表。通常，你必须单独连接到每个数据库才能运行 T-SQL 语句或执行其他管理任务。**弹性数据库作业**会处理登录任务，并可靠地为你运行脚本，同时还会记录每个数据库的执行状态。

![弹性数据库作业服务][1]

## 优点

* 定义、维护和保留要在整个弹性数据库池中执行的 T-SQL 脚本
* 使用自动重试逻辑按规模可靠执行 T-SQL 脚本
* 跟踪作业执行状态

## 方案

* 性能管理任务，例如部署新架构
* 更新引用数据，例如，所有数据库的公用产品信息
* 重新生成索引以提高查询性能

## 作业工作原理

1.	安装弹性数据库作业使用的服务。请参阅[安装弹性数据库作业](sql-database-elastic-jobs-service-installation)。如果安装失败，请参阅[如何卸载](sql-database-elastic-jobs-uninstall)。
2.	通过[将用户添加到每个数据库](sql-database-elastic-jobs-add-logins-to-dbs)，为作业执行配置弹性数据库池。
3.	在弹性数据库池视图中，单击“创建作业”。
4.	键入将要执行脚本的 SQL 数据库用户名和密码。（该用户名和密码是在安装弹性数据库作业时创建的）。
5.	键入作业名称，然后粘贴或键入脚本。
6.	单击“运行”，随后作业将每个数据库执行脚本。
7.	使用“管理作业”视图可查看正在运行或已运行的所有作业，以及最近的执行状态。
8.	单击任一作业可以查看作业执行详细信息，以及针对每个数据库执行的作业的状态。
9.	如果作业失败，请单击其名称查看错误日志。

## 组件和定价

以下组件配合工作可以创建支持即席执行管理作业的 Azure 云服务。在安装期间，这些组件将自动在你的订阅中安装和配置。你可以识别这些服务，因为它们具有相同的自动生成名称。名称是唯一的，包括前缀“edj”后接 21 个随机生成的字符。

* **Azure 云服务**：弹性数据库作业（预览版）以客户托管的 Azure 云服务交付，可执行请求的任务。服务可从门户部署，并托管在你的 Windows Azure 订阅中。默认部署的服务将结合最少两个辅助角色运行，以实现高可用性。每个默认大小的辅助角色 (ElasticDatabaseJobWorker) 在 A0 实例上运行。有关价格，请参阅[云服务定价](/home/features/cloud-services/#price)。
* **Azure SQL 数据库**：服务使用名为**控制数据库**的 Azure SQL 数据库 来保存元数据。弹性作业可以使用有关弹性数据库池的元数据登录到每个数据库并执行脚本。默认的服务层是 S0。有关价格，请参阅 [SQL 数据库 定价](/home/features/sql-database/#price)。
* **Azure Service Bus**：Azure Service Bus 用于协调 Azure 云服务中的工作。请参阅 [Service Bus 定价](/home/features/messaging/#price)。
* **Azure 存储空间**：在某个问题需要进一步调试时，将使用 Azure 存储帐户来存储诊断输出（[Azure 诊断](cloud-services-dotnet-diagnostics)的常见做法）。有关价格，请参阅 [Azure 存储空间定价](/home/features/storage/#price)。

## 后续步骤
[安装组件](sql-database-elastic-jobs-service-installation)，然后[创建一个登录名并将其添加到池中的每个数据库](sql-database-elastic-jobs-add-logins-to-dbs)。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->

[1]: ./media/sql-database-elastic-jobs-overview/elastic-jobs.png
<!--anchors-->

<!---HONumber=61-->