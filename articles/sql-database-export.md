<properties
	pageTitle="创建和导出 Azure SQL 数据库的 BACPAC"
	description="创建和导出 Azure SQL 数据库的 BACPAC"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="02/23/2016"
	wacn.date="04/06/2016"/>


# 创建和导出 Azure SQL 数据库的 BACPAC

**单一数据库**

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-export)
- [PowerShell](/documentation/articles/sql-database-export-powershell)

本文说明如何使用 [Azure 门户](https://manage.windowsazure.cn)导出 Azure SQL 数据库的 BACPAC。

[BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 是包含数据库架构和数据的 .bacpac 文件。BACPAC 的主要用例是将数据库从一个服务器移到另一个服务器、[将本地数据库迁移到云](/documentation/articles/sql-database-cloud-migrate)，以及采用开放格式对现有数据库进行存档。

> [AZURE.NOTE] BACPAC 不能用于备份和还原操作。Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。


BACPAC 导出到 Azure 存储 blob 容器中，你可以在操作成功完成后进行下载。

若要完成本文，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)文章中的步骤创建一个。
- 具有 blob 容器的 [Azure 存储帐户](/documentation/articles/storage-create-storage-account)可存储 BACPAC。


## 导出数据库

打开要导出的数据库对应的 SQL 数据库边栏选项卡。

> [AZURE.IMPORTANT] 若要确保获得事务处理一致性 BACPAC 文件，应首先[创建数据库的副本](/documentation/articles/sql-database-copy)，然后导出该数据库副本。

1.	转到 [Azure 门户](https://manage.windowsazure.cn)。
2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击要导出为 BACPAC 的数据库。
3.	在 SQL 数据库边栏选项卡中，单击“导出”以打开“导出数据库”边栏选项卡：

    ![导出按钮][1]

1.  单击“存储”并选择将在其中存储 BACPAC 的存储帐户和 blob 容器：

    ![导出数据库][2]

1.  为包含你要导出的数据库的 Azure SQL 服务器输入“服务器管理员登录名”和“密码”。
1.  单击“创建”以导出数据库。

单击“创建”会创建导出数据库请求并将其提交到服务。根据数据库的大小，导出操作可能需要一些时间才能完成。

## 监视导出操作的进度

2.	单击“浏览全部”。
3.	单击“SQL Server”。
2.	单击包含你刚导出的原始（源）数据库的服务器。
3.	在 SQL 服务器边栏选项卡中，单击“导入/导出历史记录”：


## 确认 BACPAC 位于你的存储容器中

2.	单击“浏览全部”。
3.	单击“存储帐户(经典)”。
2.	单击你在其中存储 BACPAC 的存储帐户。
3.	单击“容器”，然后选择数据库所导出到的容器以了解详细信息（从这里可以下载和保存 BACPAC）。


## 后续步骤

- [导入 Azure SQL 数据库](/documentation/articles/sql-database-import)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)
- [SQL 数据库文档](/documentation/services/sql-databases)


<!--Image references-->
[1]: ./media/sql-database-export/export.png
[2]: ./media/sql-database-export/export-blade.png
[3]: ./media/sql-database-export/export-history.png
[4]: ./media/sql-database-export/export-status.png
[5]: ./media/sql-database-export/bacpac-details.png

<!---HONumber=Mooncake_0328_2016-->
