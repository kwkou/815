<properties
	pageTitle="在 Windows上使用 PHP 连接到 SQL 数据库 | Azure"
	description="演示一个示例 PHP 程序，该程序可以从 Windows 客户端连接到 Azure SQL 数据库，并与客户端所需的软件组件建立链接。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="06/16/2016"
	wacn.date="07/11/2016"/>


# 在 Windows上使用 PHP 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题演示了如何从 Windows 运行的、以 PHP 编写的客户端应用程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[配置用于 PHP 开发的开发环境](https://msdn.microsoft.com/zh-cn/library/mt720663.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 步骤 3：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]


## 步骤 4：运行示例代码

* [使用 PHP 连接到 SQL 以进行概念认证](https://msdn.microsoft.com/zh-cn/library/mt720665.aspx)
* [使用 PHP 弹性连接到 SQL](https://msdn.microsoft.com/zh-cn/library/mt720667.aspx)


## 后续步骤

* 参阅 [SQL Database Development Overview（SQL 数据库开发概述）](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft PHP Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/dn865013.aspx) 的详细信息
* 有关 PHP 安装和用法的详细信息，请参阅[使用 PHP 访问 SQL Server 数据库](http://social.technet.microsoft.com/wiki/contents/articles/1258.accessing-sql-server-databases-from-php.aspx)。

## 其他资源 

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库功能](/home/features/sql-database/)

<!---HONumber=Mooncake_0704_2016-->