<properties
	pageTitle="SQL 数据库教程：安全性入门"
	description="了解如何创建用户帐户来访问和管理数据库。"
	keywords=""
	services="sql-database"
	documentationCenter=""
	authors="carlrabeler"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="05/03/2016"
	wacn.date="05/16/2016"/>

# SQL 数据库教程：使用 Azure 管理门户创建 SQL 数据库用户帐户以访问和管理数据库

在本教程中，你将学习如何使用 Azure 管理门户来：

- 使用服务器级主体登录名登录到 SQL 数据库
- 创建 SQL 数据库用户帐户
- 授予 SQL 数据库用户帐户在用户数据库中的 dbo 权限
- 使用非服务器级主体的用户帐户连接到 SQL 数据库 

[AZURE.INCLUDE [登录](../includes/azure-getting-started-portal-login.md)]

[AZURE.INCLUDE [创建 SQL 数据库逻辑服务器](../includes/sql-database-sql-server-management-studio-connect-server-principal.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../includes/sql-database-create-new-database-user.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../includes/sql-database-grant-database-user-dbo-permissions.md)]

[AZURE.INCLUDE [创建 SQL 数据库数据库](../includes/sql-database-sql-server-management-studio-connect-user.md)]

## 后续步骤
完成本 SQL 数据库教程、创建用户帐户并为该用户帐户授予 dbo 权限后，便可以详细了解 [SQL 数据库安全性](/documentation/articles/sql-database-manage-logins/)。


<!---HONumber=Mooncake_0503_2016-->
