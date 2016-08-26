<!-- Remove Azure portal tde -->
<properties
   pageTitle="SQL 数据仓库透明数据加密 (TDE) TSQL 入门 | Azure"
   description="SQL 数据仓库透明数据加密 (TDE) TSQL 入门"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/07/2016"
   wacn.date="07/04/2016"/>

# 透明数据加密 (TDE) 入门
> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-encryption-tde-tsql/)
<!-- - [Azure 门户](/documentation/articles/sql-data-warehouse-encryption-tde/) -->

Azure SQL 数据仓库透明数据加密 (TDE) 无需更改应用程序，即可对静止的数据库、关联的备份和事务日志执行实时加密和解密，帮助防止恶意活动的威胁。

TDE 使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。在 SQL 数据库中，数据库加密密钥由内置服务器证书保护。内置服务器证书对每个 SQL 数据库服务器都是唯一的。Microsoft 每隔 90 天自动轮换这些证书至少一次。SQL 数据仓库使用的加密算法为 AES-256。有关 TDE 的一般描述，请参阅[透明数据加密 (TDE)]。

##启用加密

若要为 SQL 数据仓库启用 TDE，请遵循以下步骤：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到托管数据库的服务器上的 *master* 数据库
2. 执行以下语句来加密数据库。


	    ALTER DATABASE [AdventureWorks] SET ENCRYPTION ON;


##禁用加密

若要为 SQL 数据仓库禁用 TDE，请遵循以下步骤：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库
2. 执行以下语句来加密数据库。


	    ALTER DATABASE [AdventureWorks] SET ENCRYPTION OFF;


##验证加密

若要验证 SQL 数据仓库的加密状态，请遵循以下步骤：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 *master* 数据库或实例数据库
2. 执行以下语句来加密数据库。


	    SELECT [name], [is\_encrypted] FROM sys.databases;


结果 ```1``` 表示数据库已加密，```0``` 表示数据库未加密。

##加密 DMV  

- [sys.databases][] 
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys](https://msdn.microsoft.com/zh-cn/library/mt203922.aspx)


<!--Anchors-->
[透明数据加密 (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx
[sys.databases]: http://msdn.microsoft.com/zh-cn/library/ms178534.aspx
[sys.dm\_pdw\_nodes\_database\_encryption\_keys]: https://msdn.microsoft.com/zh-cn/library/mt203922.aspx

<!--Image references-->

<!--Link references-->

<!---HONumber=Mooncake_0627_2016-->