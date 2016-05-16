<properties
	pageTitle="导入 BACPAC 文件以创建新的 Azure SQL 数据库 | Azure"
	description="通过导入现有的 BACPAC 文件创建新的 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="04/11/2016"
	wacn.date="05/16/2016"


# 导入 BACPAC 文件以创建新的 Azure SQL 数据库


**单一数据库**

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-import)
- [PowerShell](/documentation/articles/sql-database-import-powershell)
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage)

本文说明如何使用 [Azure 门户](https://manage.windowsazure.cn)通过 BACPAC 文件创建新的 Azure SQL 数据库。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。数据库是使用从 Azure 存储 blob 容器导入的 BACPAC 创建的。如果 Azure 存储空间中没有 .bacpac 文件，你可以按照[创建和导出 Azure SQL 数据库的 BACPAC](/documentation/articles/sql-database-export) 中的步骤创建一个。


> [AZURE.NOTE] Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。


要从 .bacpac 导入 SQL 数据库，你需要以下各项：

- Azure 订阅。 
- Azure SQL 数据库 V12 服务器。如果你没有 V12 服务器，可以遵循本文中的以下步骤创建一个：[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)。
- 要导入的数据库的 .bacpac 文件位于 [Azure 存储帐户（标准）](/documentation/articles/storage-create-storage-account)blob 容器中。

重要说明：从 Azure Blob 存储导入 BACPAC 时，请使用标准存储。不支持从高级存储导入 BACPAC。


## 选择将包含数据库的服务器

打开要导入的数据库的 SQL Server 边栏选项卡：

1.	转到 [Azure 门户](https://manage.windowsazure.cn)。
2.	单击“SQL Server”。
3.	单击要将数据库还原到的服务器。
4.	在 SQL Server 边栏选项卡中，单击“导入数据库”以打开“导入数据库”边栏选项卡：
1.  单击“存储”并选择存储帐户、blob 容器和 .bacpac 文件，然后单击“确定”。
1.  为新数据库选择定价层，然后单击“选择”。
1.  对于你将从 BACPAC 文件中创建的数据库，输入“数据库名称”。
2.  选择身份验证类型，然后向服务器提供身份验证信息。 
3.  单击“创建”以从 BACPAC 创建数据库。

单击“创建”会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

## 监视导入操作的进度

1.	单击“SQL Server”。
2.	单击要还原到的服务器。
3.	在 SQL Server 边栏选项卡中，在操作区域，单击“导入/导出历史记录”：


## 验证数据库位于服务器上

1.	单击“SQL 数据库”并验证新数据库处于“联机”状态。



## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms)



## 其他资源

- [SQL 数据库文档](/documentation/services/sql-databases)


<!--Image references-->
[1]: ./media/sql-database-import/import-database.png
[2]: ./media/sql-database-import/storage-options.png
[3]: ./media/sql-database-import/pricing-tier.png
[4]: ./media/sql-database-import/create.png
[5]: ./media/sql-database-import/import-history.png
[6]: ./media/sql-database-import/import-status.png

<!---HONumber=Mooncake_0509_2016-->
