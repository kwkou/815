<properties
	pageTitle="使用 Azure 自动化管理 Azure SQL 数据库"
	description="了解如何使用 Azure 自动化服务来方便管理 Azure SQL 数据库。"
	services="sql-database, automation"
	documentationCenter=""
	authors="jodoglevy"
	manager="jeffreyg"
	editor="monicar"/>

<tags
	ms.service="sql-database"
	ms.date="05/03/2016"
	wacn.date="05/23/2016"/>



#使用 Azure 自动化管理 Azure SQL 数据库

本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure SQL 数据库的管理。


## 什么是 Azure 自动化？

[Azure 自动化](/home/features/automation)是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure 自动化可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT/开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure 自动化如何帮助管理 Azure SQL 数据库？

可以使用 [Azure PowerShell 工具](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)中提供的 Azure SQL 数据库 PowerShell cmdlet 在 Azure 自动化中管理 Azure SQL 数据库。Azure 自动化现成地提供了这些 Azure SQL 数据库 PowerShell cmdlet，因此，你可以在该服务中执行所有 SQL DB 管理任务。你还可以将 Azure 自动化中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

Azure 自动化还可以通过使用 PowerShell 发出 SQL 命令，来与 SQL 服务器直接通信。

[Azure 自动化 Runbook 库](https://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery)包含产品团队和社区提供的各种 Runbook，以帮助你开始自动管理 Azure SQL 数据库、其他 Azure 服务和第三方系统。库中 Runbook 的功能包括：

 * [对 SQL Server 数据库运行 SQL 查询](https://gallery.technet.microsoft.com/scriptcenter/How-to-use-a-SQL-Command-be77f9d2)
 * [按计划纵向缩放（向上或向下）Azure SQL 数据库](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-Database-e957354f)
 * [当数据库接近其最大大小时截断 SQL 表](https://gallery.technet.microsoft.com/scriptcenter/Azure-Automation-Your-SQL-30f8736b)
 * [当 Azure SQL 数据库中的表高度碎片化时为这些表编制索引](https://gallery.technet.microsoft.com/scriptcenter/Indexes-tables-in-an-Azure-73a2a8ea)

## 后续步骤

在了解 Azure 自动化 以及如何使用它来管理 Azure SQL 数据库的基础知识后，请使用以下链接了解有关 Azure 自动化的更多信息。

- [Azure 自动化概述](/documentation/articles/automation-intro/)
- [Azure 自动化学习路线图](https://azure.microsoft.com/documentation/learning-paths/automation)
- [Azure 自动化：你在云中的 SQL 代理](https://azure.microsoft.com/blog/2014/06/26/azure-automation-your-sql-agent-in-the-cloud) 
 

<!---HONumber=Mooncake_0307_2016-->