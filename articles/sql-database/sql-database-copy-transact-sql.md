<properties 
    pageTitle="使用 Transact-SQL 复制 Azure SQL 数据库 | Azure" 
    description="使用 Transact-SQL 创建 Azure SQL 数据库的副本" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>  


<tags
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="09/19/2016"
	wacn.date="11/16/2016"
	ms.author="sstein"
	ms.workload="data-management"
	ms.topic="article"
	ms.tgt_pltfrm="NA"/>  



# 使用 Transact-SQL 复制 Azure SQL 数据库


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-copy)
- [T-SQL](/documentation/articles/sql-database-copy-transact-sql/)
- [PowerShell](/documentation/articles/sql-database-copy-powershell)
- [T-SQL](/documentation/articles/sql-database-copy-transact-sql)


以下步骤说明如何使用 Transact-SQL 将 SQL 数据库复制到同一服务器或其他服务器。复制数据库的操作将使用 [CREATE DATABASE](https://msdn.microsoft.com/zh-cn/library/ms176061.aspx) 语句。

若要完成本文中的步骤，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)一文中的步骤创建一个。
- [SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/zh-cn/library/ms174173.aspx)。如果你没有 SSMS，或无法使用本文中所述的功能，请[下载最新版本](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。


##<a name="copy-your-sql-database"></a> 复制 SQL 数据库

使用服务器级别主体登录名或创建了你要复制的数据库的登录名登录到 master 数据库。非服务器级主体的登录名必须是 dbmanager 角色的成员才能复制数据库。有关登录名和链接到服务器的详细信息，请参阅[管理登录名](/documentation/articles/sql-database-manage-logins/)。

使用 [CREATE DATABASE](https://msdn.microsoft.com/zh-cn/library/ms176061.aspx) 语句开始复制源数据库。执行此语句将启动数据库复制过程。因为复制数据库是一个异步过程，所以，CREATE DATABASE 语句将在数据库完成复制前返回。


### 将 SQL 数据库复制到同一台服务器

使用服务器级别主体登录名或创建了你要复制的数据库的登录名登录到 master 数据库。非服务器级主体的登录名必须是 dbmanager 角色的成员才能复制数据库。

此命令将 Database1 复制到同一服务器上名为 Database2 的新数据库。根据数据库的大小，复制操作可能需要一些时间才能完成。

    -- Execute on the master database.
    -- Start copying.
    CREATE DATABASE Database1_copy AS COPY OF Database1;

### 将 SQL 数据库复制到不同的服务器

登录到目标服务器（即要创建新数据库的 Azure SQL 数据库服务器）的 master 数据库。所用登录名的名称和密码应该与源 Azure SQL 数据库服务器上源数据库的数据库所有者 (DBO) 的名称和密码相同。目标服务器上的登录名也必须是 dbmanager 角色的成员或服务器级主体登录名。

此命令将 server1 上的 Database1 复制到 server2 上名为 Database2 的新数据库。根据数据库的大小，复制操作可能需要一些时间才能完成。


    -- Execute on the master database of the target server (server2)
    -- Start copying from Server1 to Server2
    CREATE DATABASE Database1_copy AS COPY OF server1.Database1;
    

## 监视复制操作的进度

通过查询 sys.databases 和 sys.dm\_database\_copies 视图来监视复制过程。在复制过程中，新数据库的 sys.databases 视图的 state\_desc 列将设置为 COPYING。


- 如果复制失败，新数据库的 sys.databases 视图的 state\_desc 列将设置为 SUSPECT。在这种情况下，对新数据库执行 DROP 语句并稍后重试。
- 如果复制成功，新数据库的 sys.databases 视图的 state\_desc 列将设置为 ONLINE。在这种情况下，复制已完成并且新数据库是一个常规数据库，可独立于源数据库进行更改。

> [AZURE.NOTE] - 如果决定在复制过程中取消复制，请对新数据库执行 [DROP DATABASE](https://msdn.microsoft.com/zh-cn/library/ms178613.aspx) 语句。此外，对源数据库执行 DROP DATABASE 语句也将取消复制过程。


##<a name="resolve-logins-after-the-copy-operation-completes"></a> 在复制操作完成后解析登录名

当新数据库在目标服务器上联机后，使用 [ALTER USER](https://msdn.microsoft.com/zh-cn/library/ms176060.aspx) 语句将新数据库中的用户重新映射到目标服务器上的登录名。若要解析孤立用户，请参阅[孤立用户疑难解答](https://msdn.microsoft.com/zh-cn/library/ms175475.aspx)。另请参阅[灾难恢复后如何管理 Azure SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/)。

新数据库中的所有用户都保持他们在源数据库中已有的权限。启动数据库复制过程的用户将成为新数据库的数据库所有者，并且会为该用户分配一个新的安全标识符 (SID)。复制成功之后，重新映射其他用户之前，只有启动复制的登录名，即数据库所有者 (DBO)，才能登录到新数据库。


## 后续步骤

- 有关复制 Azure SQL 数据库的概述，请参阅[复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/)。
- 如果你决定在复制过程中取消复制，请对新数据库执行 [DROP DATABASE](https://msdn.microsoft.com/zh-cn/library/ms178613.aspx) 语句。此外，对源数据库执行 DROP DATABASE 语句也将取消复制过程。
- 若要使用 PowerShell 复制数据库，请参阅[使用 PowerShell 复制 Azure SQL 数据库](/documentation/articles/sql-database-copy-powershell/)。
- 若要了解如何在将数据库复制到其他逻辑服务器时管理用户和登录名，请参阅[灾难恢复后如何管理 Azure SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/)。



## 其他资源

- [管理登录名](/documentation/articles/sql-database-manage-logins/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)
- [将数据库导出到 BACPAC](/documentation/articles/sql-database-export-powershell/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases/)

<!---HONumber=Mooncake_1024_2016-->