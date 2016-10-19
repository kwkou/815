<properties
	pageTitle="SQL 数据库教程：安全性入门"
	description="了解如何创建用户帐户来访问和管理数据库。"
	keywords=""
	services="sql-database"
	documentationCenter=""
	authors="CarlRabeler"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="08/17/2016"
	wacn.date="10/17/2016"
	ms.author="carlrab"/>  


# SQL 数据库教程：创建 SQL 数据库用户帐户用于访问和管理数据库


> [AZURE.SELECTOR]
- [入门教程](/documentation/articles/sql-database-get-started-security/)
- [授予访问权限](/documentation/articles/sql-database-manage-logins/)

本教程介绍如何使用 SQL Server Management Studio (SSMS) 来执行以下操作：

- 使用服务器级主体登录名登录到 SQL 数据库。
- 创建 SQL 数据库用户帐户。
- 向 SQL 数据库用户授予 [db\_owner 权限](https://msdn.microsoft.com/zh-cn/library/ms189121.aspx#Anchor_0)。
- 使用非服务器级主体的用户帐户连接到 SQL 数据库。

[AZURE.INCLUDE [登录](../../includes/azure-getting-started-portal-login.md)]


[AZURE.INCLUDE [创建 SQL 数据库逻辑服务器](../../includes/sql-database-sql-server-management-studio-connect-server-principal.md)]


[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-database-user.md)]


[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-grant-database-user-dbo-permissions.md)]


[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-sql-server-management-studio-connect-user.md)]


## 后续步骤
完成本 SQL 数据库教程、创建用户帐户并为该帐户授予 dbo 权限后，便可详细了解 [SQL 数据库安全](/documentation/articles/sql-database-manage-logins/)。

<!---HONumber=Mooncake_1010_2016-->