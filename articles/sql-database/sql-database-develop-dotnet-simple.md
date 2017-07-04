<properties
    pageTitle="使用 .NET (C#) 连接到 SQL 数据库 | Azure"
    description="使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的新式应用程序。"
    services="sql-database"
    documentationcenter=""
    author="tobbox"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="7faca033-24b4-4f64-9301-b4de41e73dfd"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/03/2017"
    wacn.date="03/24/2017"
    ms.author="tobiast" />

# 使用 .NET (C#) 连接到 SQL 数据库

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

## 步骤 1：配置开发环境

[配置用于 ADO.NET 开发的开发环境](https://docs.microsoft.com/sql/connect/ado-net/step-1-configure-development-environment-for-ado-net-development/)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。务必根据指南创建 **AdventureWorks 数据库模板**。下面展示的示例仅适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-dotnet-20-portalshots](../../includes/sql-database-include-connection-string-dotnet-20-portalshots.md)]

## 步骤 4：运行示例代码

* [使用 ADO.NET 连接到 SQL 以进行概念证明](https://docs.microsoft.com/sql/connect/ado-net/step-3-proof-of-concept-connecting-to-sql-using-ado-net/)
* [使用 ADO.NET 弹性连接到 SQL](https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-to-sql-with-ado-net/)

## 后续步骤
* [创建具有身份验证和 SQL 数据库的 ASP.NET MVC 应用程序并将其部署到 Azure 应用服务](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/)
* 参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
* 有关 [用于 SQL Server 的 Microsoft ADO.Net Driver](https://docs.microsoft.com/sql/connect/ado-net/microsoft-ado-net-for-sql-server/) 的详细信息

## 其他资源 

* [多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库的功能](/home/features/sql-database/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->