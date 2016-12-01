<properties
	pageTitle="使用 .NET (C#) 连接到 SQL 数据库 | Azure"
	description="使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的现代应用程序。"
	services="sql-database"
	documentationCenter=""
	authors="tobbox"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="drivers"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="06/16/2016"
	wacn.date="07/11/2016"
	ms.author="tobiast"/>

# 使用 .NET (C#) 连接到 SQL 数据库

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

## 步骤 1：配置开发环境

[配置用于 ADO.NET 开发的开发环境](https://msdn.microsoft.com/zh-cn/library/mt718321.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-dotnet-20-portalshots](../../includes/sql-database-include-connection-string-dotnet-20-portalshots.md)]

## 步骤 4：运行示例代码

* [使用 ADO.NET 连接到 SQL 以进行概念证明](https://msdn.microsoft.com/zh-cn/library/mt718320.aspx)
* [使用 ADO.NET 弹性连接到 SQL](https://msdn.microsoft.com/zh-cn/library/mt703195.aspx)

## 后续步骤

* [创建具有身份验证和 SQL 数据库的 ASP.NET MVC 应用程序并将其部署到 Azure Wev 应用](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)
* [使用现有的 SQL 数据库和移动服务 .NET 后端生成服务](/documentation/articles/mobile-services-dotnet-backend-use-existing-sql-database/)
* 参阅 [SQL Database Development Overview（SQL 数据库开发概述）](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft ADO.Net Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/mt657768.aspx) 的详细信息

## 其他资源 

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库功能](/home/features/sql-database/)






<!---HONumber=Mooncake_0704_2016-->