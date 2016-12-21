<properties
    pageTitle="复制 Azure SQL 数据库 | Azure"
    description="创建 Azure SQL 数据库的副本"
    services="sql-database"
    documentationcenter=""
    author="anosov1960"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="5aaf6bcd-3839-49b5-8c77-cbdf786e359b"
    ms.service="sql-database"
    ms.devlang="NA"
    ms.date="10/24/2016"
    wacn.date="12/19/2016"
    ms.author="sstein; sashan"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />

#<a name="copy-your-sql-database"></a> 复制 Azure SQL 数据库

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-copy/)
- [Azure 门户预览](/documentation/articles/sql-database-copy/)
- [PowerShell](/documentation/articles/sql-database-copy-powershell/)
- [T-SQL](/documentation/articles/sql-database-copy-transact-sql/)


可以使用 Azure [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)创建 SQL 数据库的副本。数据库副本使用相同技术作为异地复制功能。但与异地复制不同的是，它会在种子设定阶段完成后终止复制链接。因此，提出复制请求时，副本数据库是源数据库的快照。可以在相同或不同的服务器上创建数据库副本。默认情况下，数据库副本的服务层和性能级别（定价层）均与源数据库相同。使用 API 时，可以选择同一服务层（版本）内的其他性能级别。在完成复制后，副本将成为能够完全行使功能的独立数据库。此时，可以将其升级或降级为任何版本。登录名、用户和权限可单独进行管理。

将某个数据库复制到同一逻辑服务器时，可以在这两个数据库上使用相同的登录名。用于复制该数据库的安全主体将成为新数据库上的数据库所有者 (DBO)。所有数据库用户、其权限及安全标识符 (SID) 都将复制到数据库副本中。

将数据库复制到不同的逻辑服务器时，新服务器上的安全主体将成为新数据库上的数据库所有者。如果使用[包含的数据库用户](/documentation/articles/sql-database-manage-logins/)进行数据访问，请确保主要和辅助数据库始终具有相同的用户凭据，以便在复制完成后能够立即使用相同的凭据进行访问。如果使用 [Azure Active Directory](/documentation/articles/active-directory-whatis/)，完全无需管理副本中的凭据。但是，将数据库复制到新服务器时，基于登录名的用户通常不起作用，因为登录名在新服务器上不存在。若要了解如何在将数据库复制到其他逻辑服务器时管理登录名，请参阅[灾难恢复后如何管理 Azure SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/)。

若要复制 SQL 数据库，需要做好以下准备：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 要复制的 SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)一文中的步骤创建一个。

## 后续步骤
- 若要使用 Azure 门户预览复制数据库，请参阅 [使用 Azure 门户预览复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-portal/)。
- 若要使用 PowerShell 复制数据库，请参阅 [使用 PowerShell 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-powershell/)。
- 若要使用 Transact-SQL 复制数据库，请参阅 [使用 Transact-SQL 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-transact-sql/)。
- 若要了解如何在将数据库复制到其他逻辑服务器时管理用户和登录名，请参阅 [灾难恢复后如何管理 Azure SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/)。

## 其他资源

- [Manage logins（管理登录名）](/documentation/articles/sql-database-manage-logins/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)
- [将数据库导出到 BACPAC](/documentation/articles/sql-database-export/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases/)

<!---HONumber=Mooncake_1212_2016-->