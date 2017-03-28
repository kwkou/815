<properties
    pageTitle="SqlPackage：从 BACPAC 文件导入到 Azure SQL 数据库 | Azure"
    description="本文说明如何使用 SqlPackage 命令行实用程序从 BACPAC 文件导入到 SQL 数据库。"
    keywords="Azure SQL 数据库, 数据库迁移, 导入数据库, 导入 BACPAC 文件, sqlpackage"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="424afa27-5f13-4ec3-98f6-99a511a6a2df"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="sqldb-migrate"
    ms.date="02/08/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab" />  


# 使用 SqlPackage 将数据库从 BACPAC 文件导入到 Azure SQL 数据库

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序从 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件导入到 SQL 数据库。此实用程序随 [SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 和[用于 Visual Studio 的 SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 的最新版本提供，也可直接从 Microsoft 下载中心下载 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) 的最新版本。

> [AZURE.NOTE]
>以下步骤假定用户已预配 SQL 数据库服务器，手头有连接信息，并且已验证源数据库兼容。
> 
> 

## 导入数据库
使用 [SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序从 BACPAC 文件导入兼容的 SQL Server 数据库（或 Azure SQL 数据库）。

> [AZURE.NOTE]
以下步骤假定用户已预配 Azure SQL 数据库服务器并且手头有连接信息。
>  

在包含最新版 sqlpackage.exe 命令行实用程序的目录中的命令提示符下，执行类似于以下示例命令的命令，将 BACPAC 文件导入充当 Premium P11 的 Azure SQL 数据库中。


	SqlPackage.exe /a:import /tcs:"Data Source=SERVER;Initial Catalog=DBNAME;User Id=USER;Password=PASSWORD" /sf:C:\db.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P11


## 后续步骤

* 如需 SQL Server 数据库完整迁移过程的介绍（包括性能建议），请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
* 如需有关 SQLPackage 的参考内容，请参阅 [SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx)。
* 若要下载 SQLPackage 的最新版本，请参阅 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876)。
* 如需 SQL Server 客户顾问团队编写的有关使用 BACPAC 文件进行迁移的博客，请参阅 [使用 BACPAC 文件从 SQL Server 迁移到 Azure SQL 数据库](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。

<!---HONumber=Mooncake_0320_2017-->