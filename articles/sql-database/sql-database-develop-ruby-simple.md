<properties
    pageTitle="使用 Ruby 连接到 SQL 数据库 | Azure"
    description="提供可运行的用于连接到 Azure SQL 数据库的 Ruby 代码示例。"
    services="sql-database"
    documentationcenter=""
    author="ajlam"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="94fec528-58ba-4352-ba0d-25ae4b273e90"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="02/03/2017"
    wacn.date="03/24/2017"
    ms.author="andrela" />

# 使用 Ruby 连接到 SQL 数据库 

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题说明如何使用 Ruby 来连接和查询 Azure SQL 数据库。可以从 Windows、Ubuntu Linux 或 Mac 平台运行此示例。

## 步骤 1：配置开发环境
[使用 TinyTDS Ruby Driver for SQL Server 的先决条件](https://docs.microsoft.com/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development/)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。务必根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 4：运行示例代码
[“使用 Ruby 连接到 SQL”概念证明](https://docs.microsoft.com/sql/connect/ruby/step-3-proof-of-concept-connecting-to-sql-using-ruby/)

## 后续步骤

* 参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft Ruby Driver for SQL Server](https://docs.microsoft.com/sql/connect/ruby/ruby-driver-for-sql-server/) 的详细信息

## 其他资源 

* [多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库功能](/home/features/sql-database/)

<!---HONumber=Mooncake_0320_2017-->