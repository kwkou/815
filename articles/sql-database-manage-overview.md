<properties 
	pageTitle="概述：SQL 数据库的管理工具" 
	description="比较管理 Azure SQL 数据库的工具和选项" 
	services="sql-database" 
	documentationCenter="" 
	authors="TigerMint" 
	manager="" 
	editor=""/>

<tags
	ms.service="sql-database" 
  	wacn.date="05/20/2015"
	ms.date="10/17/2015" />

# 概述：SQL 数据库的管理工具

本主题介绍并比较用于管理 SQL 数据库的工具和选项，以便你可以挑选适合作业、业务以及你自己的工具。选择合适的工具取决于你管理的数据库数量、任务以及执行任务的频率。



## Azure 门户


[Azure 门户](https://manage.windowsazure.cn)是一个基于 Web 的管理门户，你可以从中创建、更新和删除 Azure SQL 数据库及逻辑服务器并监视数据库资源。如果你刚开始使用 Azure、管理少量的数据库或需要快速执行某些操作，该工具是理想之选。

有关使用该门户的更多详细信息，请参阅[使用 Azure 门户管理 Azure 数据库](/documentation/articles/sql-database-manage-portal)。

## Visual Studio 中的 SQL Server Management Studio 和 SQL Server Data Tools


Visual Studio 中的 SQL Server Management Studio (SSMS) 和 SQL Server Data Tools (SSDT) 是在计算机上运行的客户端工具，你可以用它们来连接到、管理和开发云中的数据库。如果你是熟悉 Visual Studio 或其他集成开发环境 (IDE) 的应用程序开发人员，请尝试使用 Visual Studio 中的 SSDT。如果你需要使用 SSDT 中尚不支持的 SQL 功能，例如在混合环境中管理 SQL Server 数据库，请使用 SSMS。

有关使用 SSMS 和 SSDT 管理 Azure SQL 数据库的详细信息，请参阅[使用 SSMS 管理 SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms)


## 命令行工具

你可以使用诸如 PowerShell 之类的命令行工具管理 Azure SQL 数据库并自动执行 Azure 资源部署。Microsoft 建议在生产环境中使用此工具来管理大量的 Azure SQL 数据库和部署资源变化。

有关使用命令行工具管理 Azure SQL 数据库的详细信息，请[单击此处](/documentation/articles/sql-database-command-line-tools)

<!---HONumber=74-->