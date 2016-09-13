<properties
	pageTitle="复制 Azure SQL 数据库 | Azure"
	description="创建 Azure SQL 数据库的副本"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="06/16/2016"
	wacn.date="07/11/2016"/>



# 复制 Azure SQL 数据库

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-copy/)
- [PowerShell](/documentation/articles/sql-database-copy-powershell/)
- [T-SQL](/documentation/articles/sql-database-copy-transact-sql/)

可以使用 Azure [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)来创建 SQL 数据库的副本。复制操作将复制事务日志的尾部，然后使用自动备份中的完整、差异和事务日志备份来创建自最终事务日志备份起，与源数据库保持事务一致的副本。

可以在相同或不同的服务器上创建数据库副本。数据库副本的服务层和性能级别（定价层）都与源数据库相同。在完成该复制后，副本将成为能够完全行使功能的独立数据库。登录名、用户和权限可单独进行管理。

将某个数据库复制到同一逻辑服务器时，可以在这两个数据库上使用相同的登录名。用于复制该数据库的安全主体将成为新数据库上的数据库所有者 (DBO)。所有数据库用户、其权限及安全标识符 (SID) 都复制到数据库副本中。

将数据库复制到不同的逻辑服务器时，新服务器上的安全主体将成为新数据库上的数据库所有者。充当包含用户的数据库用户可以在复制的数据库中使用。但是，将数据库复制到新服务器时，基于登录名的用户通常不起作用，因为登录名在新服务器上不存在，即使存在，它们的 SID 也可能不匹配。当新数据库在目标服务器上联机后，使用 [ALTER USER](https://msdn.microsoft.com/zh-cn/library/ms176060.aspx) 语句将新数据库中的用户重新映射到目标服务器上的登录名。若要解析孤立用户，请参阅 [Troubleshoot Orphaned Users（孤立用户疑难解答）](https://msdn.microsoft.com/zh-cn/library/ms175475.aspx)。


若要复制 SQL 数据库，需要做好以下准备：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 要复制的 SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)文章中的步骤创建一个。

## 后续步骤


- 若要使用 PowerShell 复制数据库，请参阅 [Copy an Azure SQL database using PowerShell（使用 PowerShell 复制 Azure SQL 数据库）](/documentation/articles/sql-database-copy-powershell/)。
- 若要使用 Transact-SQL 复制数据库，请参阅 [Copy an Azure SQL database using T-SQL（使用 Transact-SQL 复制 Azure SQL 数据库）](/documentation/articles/sql-database-copy-transact-sql/)。
- 若要了解如何在将数据库复制到其他逻辑服务器时管理用户和登录名，请参阅 [How to manage Azure SQL database security after disaster recovery（灾难恢复后如何管理 Azure SQL 数据库安全性）](/documentation/articles/sql-database-geo-replication-security-config/)。



## 其他资源

- [Manage logins（管理登录名）](/documentation/articles/sql-database-manage-logins/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)
- [将数据库导出到 BACPAC](/documentation/articles/sql-database-export-powershell/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases/)

<!---HONumber=Mooncake_0704_2016-->