<properties 
	pageTitle="概述：SQL 数据库 的管理工具" 
	description="比较用于管理 Azure SQL 数据库 的工具和选项" 
	services="sql-database" 
	documentationCenter="" 
	authors="TigerMint" 
	manager="" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/15/2015"
	wacn.date="05/25/2015" 
	ms.author="vinsonyu"/>

# 概述：SQL 数据库 的管理工具

本主题将探讨和比较用于管理 Azure SQL 数据库 的工具与选项，以便你能够为作业、业务和自己选择适当的工具。如何选择适当的工具取决于你要管理的数据库数量、任务以及执行某个任务的频率。



## Azure 门户


[Azure 门户](http://manage.windowsazure.cn)是基于 Web 的管理门户，你可以在其中创建、更新和删除 Azure SQL 数据库 与逻辑服务器，以及监视数据库资源。如果你是 Azure 的新手、要管理的数据库较少或者需要快速完成某个任务，该门户可充当一个极好的工具。 

有关使用该门户的更深入信息，请参阅[使用 Azure 门户管理 SQL 数据库](/documentation/articles/sql-database-manage-portal)。

## Visual Studio 中的 SQL Server Management Studio 和 SQL Server Data Tools


Visual Studio 中的 SQL Server Management Studio (SSMS) 和 SQL Server Data Tools (SSDT) 是在计算机上运行的客户端工具，可让你连接、管理和开发云中的数据库。如果你是一名应用程序开发人员并熟悉 Visual Studio 或其他集成开发环境 (IDE)，请尝试使用 Visual Studio 中的 SSDT。如果你需要高级 SQL 功能，而 SSDT 尚不支持这种功能（例如，在混合环境中管理 SQL Server 数据库），请使用 SSMS。

有关使用 SSMS 和 SSDT 管理 Azure SQL 数据库 的详细信息，请参阅[使用 SSMS 管理 SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms)


## 命令行工具

你可以使用 PowerShell 等命令行工具管理 Azure SQL 数据库 和自动执行 Azure 资源部署。Microsoft 建议在生产环境中使用此工具管理大量 Azure SQL 数据库 和部署资源更改。 

有关使用命令行工具管理 Azure SQL 数据库 的详细信息，请[单击此处](/documentation/articles/sql-database-command-line-tools)

<!--HONumber=55-->