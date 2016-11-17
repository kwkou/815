<properties
   pageTitle="为 Azure TSQL 上的 SQL Server Stretch Database 启用透明数据加密 (TDE) | Azure"
   description="为 Azure TSQL 上的 SQL Server Stretch Database 启用透明数据加密 (TDE)"
   services="sql-server-stretch-database"
   documentationCenter=""
   authors="douglaslMS"
   manager=""
   editor=""/>

<tags
   ms.service="sql-server-stretch-database"
   ms.date="06/14/2016"
   wacn.date="07/25/2016"/>

# 为 Azure 上的 SQL Server Stretch Database 启用透明数据加密 (TDE)

透明数据加密 (TDE) 无需更改应用程序，即可对静止的数据库、关联的备份和事务日志执行实时加密和解密，帮助防止恶意活动的威胁。

TDE 使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。数据库加密密钥由内置服务器证书保护。内置服务器证书对每个 Azure 服务器都是唯一的。Microsoft 每隔 90 天自动轮换这些证书至少一次。有关 TDE 的一般描述，请参阅[透明数据加密 (TDE)]。

##启用加密

对于存储从启用延伸的 SQL Server 数据库迁移的数据的 Azure 数据库，若要启用 TDE，请执行以下操作：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到托管数据库的 Azure 服务器上的 master 数据库
2. 执行以下语句来加密数据库。

```sql
ALTER DATABASE [database_name] SET ENCRYPTION ON;
```

##禁用加密

对于存储从启用延伸的 SQL Server 数据库迁移的数据的 Azure 数据库，若要禁用 TDE，请执行以下操作：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 master 数据库
2. 执行以下语句来加密数据库。

```sql
ALTER DATABASE [database_name] SET ENCRYPTION OFF;
```

##验证加密

若要验证存储从启用延伸的 SQL Server 数据库迁移的数据的 Azure 数据库的加密状态，请执行以下操作：

1. 使用在 master 数据库中充当管理员或 **dbmanager** 角色成员的登录名，连接到 master 数据库或实例数据库
2. 执行以下语句来加密数据库。

```sql
SELECT
	[name],
	[is_encrypted]
FROM
	sys.databases;
```

结果 ```1``` 表示数据库已加密，```0``` 表示数据库未加密。


<!--Anchors-->
[透明数据加密 (TDE)]: https://msdn.microsoft.com/zh-cn/library/bb934049.aspx


<!--Image references-->

<!--Link references-->

<!---HONumber=Mooncake_0523_2016-->