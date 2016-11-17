<properties
   pageTitle="SQL 数据仓库中的透明数据加密 (TDE) | Azure"
   description="SQL 数据仓库 (T-SQL) 中的透明数据加密 (TDE)"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.workload="data-management"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="09/24/2016"
   wacn.date="10/31/2016"
   ms.author="rortloff;barbkess;sonyama"/>  


# 透明数据加密 (TDE) 入门


> [AZURE.SELECTOR]
- [安全性概述](/documentation/articles/sql-data-warehouse-overview-manage-security/)
- [身份验证](/documentation/articles/sql-data-warehouse-authentication/)
- [加密（门户）](/documentation/articles/sql-data-warehouse-encryption-tde/)
- [加密 (T-SQL)](/documentation/articles/sql-data-warehouse-encryption-tde-tsql/)

## 所需的权限

若要启用透明数据加密 (TDE)，用户必须是管理员或 dbmanager 角色的成员。

## 启用加密

执行以下步骤，对 SQL 数据仓库启用 TDE：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到托管数据库的服务器上的 *master* 数据库
2. 执行以下语句来加密数据库。


	    ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;


## 禁用加密

执行以下步骤，对 SQL 数据仓库禁用 TDE：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库
2. 执行以下语句来加密数据库。


	    ALTER DATABASE [AdventureWorks] SET ENCRYPTION OFF;

> [AZURE.NOTE] 在更改 TDE 设置之前，必须恢复暂停的 SQL 数据仓库。

## 验证加密

若要验证 SQL 数据仓库的加密状态，请遵循以下步骤：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库或实例数据库
2. 执行以下语句来加密数据库。


	    SELECT [name], [is\_encrypted] FROM sys.databases;


结果 ```1``` 表示数据库已加密，```0``` 表示数据库未加密。

## 加密 DMV  

- [sys.databases][]
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys][]


<!--Anchors-->

[Transparent Data Encryption (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx
[sys.databases]: http://msdn.microsoft.com/zh-cn/library/ms178534.aspx
[sys.dm\_pdw\_nodes\_database\_encryption\_keys]: https://msdn.microsoft.com/zh-cn/library/mt203922.aspx

<!--Image references-->

<!--Link references-->

<!---HONumber=Mooncake_1024_2016-->