<properties
	pageTitle="使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件"
	description="使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="08/15/2016"
	wacn.date="09/19/2016"


# 使用 Azure 门户预览将 Azure SQL 数据库存档到 BACPAC 文件

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/sql-database-export/)
- [PowerShell](/documentation/articles/sql-database-export-powershell/)

本文说明了如何使用 [Azure 门户预览](https://portal.azure.cn)将 Azure SQL 数据库存档到 BACPAC 文件（存于 Azure blob 存储中）。

需要创建 Azure SQL 数据库的存档时，可以将数据库架构和数据导出到 BACPAC 文件。BACPAC 文件只是一个扩展名为 BACPAC 的 ZIP 文件。之后可将 BACPAC 文件存储在 Azure blob 存储中或存储在本地位置的本地存储中，之后可以导入回 Azure SQL 数据库或 SQL Server 本地安装。

***注意事项***

- 为保证存档的事务处理方式一致，须确保导出期间未发生写入活动，或者正在从 Azure SQL 数据库的[事务处理方式一致性副本](/documentation/articles/sql-database-copy/)中导出。
- 存档到 Azure Blob 存储的 BACPAC 文件的大小上限为 200 GB。可使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示使用工具将更大的 BACPAC 文件存到恩地存储。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。
- 不支持使用 BACPAC 文件存档到 Azure 高级存储。
- 如果导出操作耗时超过 20 个小时，可能会取消操作。为提高导出过程中的性能，你可以进行如下操作：
 - 暂时提高服务级别。
 - 在导出期间终止所有读取和写入活动。
 - 对所有大型表格上的非 null 值使用[聚集索引](https://msdn.microsoft.com/zh-cn/library/ms190457.aspx)。如果不使用聚集索引，当时间超过 6-12 个小时时，导出可能会失败。原因是导出服务需要完成表格扫描才能尝试导出整个表格。确认表格是否已就导出进行优化的好方法是运行 **DBCC SHOW\_STATISTICS** 并确保 *RANGE\_HI\_KEY* 不是 null 且其值分布良好。相关详细信息，请参阅 [DBCC SHOW\_STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms174384.aspx)。


> [AZURE.NOTE] BACPAC 不能用于备份和还原操作。Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

若要完成本文，你需要以下各项：

- Azure 订阅。
- Azure SQL 数据库。
- [Azure 标准存储帐户](/documentation/articles/storage-create-storage-account/)，具有可在标准存储中存储 BACPAC 的 blob 容器。

## 导出数据库

打开要导出的数据库对应的 SQL 数据库边栏选项卡。

> [AZURE.IMPORTANT] 若要确保获得事务处理一致性 BACPAC 文件，应首先[创建数据库的副本](/documentation/articles/sql-database-copy/)，然后导出该数据库副本。

1.	转到 [Azure 门户预览](https://portal.azure.cn)。
2.	单击“SQL 数据库”。
3.	单击要存档的数据库。
4.	在 SQL 数据库边栏选项卡中，单击“导出”以打开“导出数据库”边栏选项卡：

    ![导出按钮][1]

5.  单击“存储”并选择将在其中存储 BACPAC 的存储帐户和 blob 容器：

    ![导出数据库][2]

6. 选择身份验证类型。
7.  为包含要导出的数据库的 Azure SQL Server 输入相应的身份验证凭据。
8.  单击“确定”将数据库存档。单击“确定”会创建一个导出数据库请求并将其提交给服务。导出所需要的时间取决于数据库的大小和复杂程度，以及你的服务级别。你将收到通知。

    ![导出通知][3]

## 监视导出操作的进度

1.	单击“SQL Server”。
2.	单击包含你刚存档的原始（源）数据库的服务器。
3.  向下滚动到操作。
4.	在 SQL 服务器边栏选项卡中，单击“导入/导出历史记录”：

    ![导入导出历史记录][4]

## 确认 BACPAC 位于你的存储容器中

1.	单击“存储帐户”。
2.	单击你在其中存储 BACPAC 存档的存储帐户。
3.	单击“容器”，然后选择数据库所导出到的容器以了解详细信息（从这里可以下载和保存 BACPAC）。

    ![.bacpac 文件详细信息][5]

## 后续步骤

- 若要了解如何将 BACPAC 导入到 Azure SQL 数据库，请参阅[将 BACPCAC 导入 Azure SQL 数据库](/documentation/articles/sql-database-import/)
- 若要了解如何将 BACPAC 导入到 SQL 数据库，请参阅[将 BACPCAC 导入 SQL 数据库](https://msdn.microsoft.com/zh-cn/library/hh710052.aspx)



<!--Image references-->
[1]: ./media/sql-database-export/export.png
[2]: ./media/sql-database-export/export-blade.png
[3]: ./media/sql-database-export/export-notification.png
[4]: ./media/sql-database-export/export-history.png
[5]: ./media/sql-database-export/bacpac-archive.png

<!---HONumber=Mooncake_0912_2016-->