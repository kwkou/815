<properties
   pageTitle="SQL 数据仓库的管理工具 | Azure"
   description="SQL 数据仓库的管理工具简介"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="HappyNicolle"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="09/22/2015"
   wacn.date="01/20/2016"/>

# SQL 数据仓库的管理工具
本主题将探讨并比较用于管理 SQL 数据仓库的工具和选项，以便你可以挑选符合需求的适当工具。选择合适的工具取决于你管理的数据库数量、任务以及执行任务的频率。

## Visual Studio 中的 SQL Server Data Tools	
Visual Studio 中的 [SQL Server Data Tools][] (SSDT) 是在计算机上运行的客户端工具，你可以用它们来连接到、管理和开发云中的数据库。如果你是熟悉 Visual Studio 或其他集成开发环境 (IDE) 的应用程序开发人员，请尝试使用 Visual Studio 中的 SSDT。

使用 SSDT 包含的 SQL Server 资源管理器，可以针对 SQL 数据仓库数据库可视化、连接和运行脚本。若要快速连接到 SQL 数据仓库，只需在 Azure 门户中查看数据库的详细信息时，单击命令行中的“在 Visual Studio 中打开”按钮。

你可以下载支持 SQL 数据仓库的最新版的 [SQL Server Data Tools][] (SSDT)。

## 命令行工具
一个选项是使用 PowerShell 或 sqlcmd 命令行工具管理 SQL 数据仓库，并自动部署 Azure 资源。由于可为所需的作业编写脚本并自动执行此类作业，因此我们建议使用这些工具来管理大量的逻辑服务器，以及在生产环境中部署资源更改。

## 后续步骤
如要开始使用这些工具，请转到[连接][]主题。

<!--Image references-->

<!--Article references-->
[连接]: /documentation/articles/sql-data-warehouse-develop-connections

<!--MSDN references-->
[SQL Server Data Tools]: https://msdn.microsoft.com/zh-cn/library/mt204009.aspx

<!--Other web references-->
[Azure 门户]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_1207_2015-->
