<properties
	pageTitle="创建和导出 Azure SQL 数据库的 BACPAC"
	description="创建 Azure SQL 数据库的 BACPAC 并将其导出到 Azure 存储空间"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="09/05/2015"
	wacn.date="10/17/2015"/>


# 创建和导出 SQL 数据库的 BACPAC

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-export-powershell)

本文介绍如何通过 [Azure 预览门户](https://manage.widnowsazure.cn)手动导出 SQL 数据库的 BACPAC。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。有关详细信息，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)中的备份包 (.bacpac)。

> [AZURE.NOTE]Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。


BACPAC 导出到 Azure 存储 blob 容器中，你可以在操作成功完成后进行下载。

若要完成本文，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)文章中的步骤创建一个。
- 具有 blob 容器的 [Azure 存储帐户](/documentation/articles/storage-create-storage-account)可存储数据库备份。目前的存储帐户必须使用经典部署模型，因此在创建存储帐户时请选择“经典”。 


## 导出数据库

打开要导出为 .bacpac 文件的数据库的 SQL 数据库边栏选项卡：

1.	转到 [Azure 预览门户](https://manage.widnowsazure.cn)。
2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击要导出为 BACPAC 的数据库。
3.	在 SQL 数据库边栏选项卡中，单击“导出”以打开“导出数据库”边栏选项卡：

    ![导出按钮][1]

1.  单击“存储”并选择将在其中存储 BACPAC 的存储帐户和 blob 容器：

    ![导出数据库][2]

1.  为包含你要备份的数据库的 Azure SQL 服务器输入“服务器管理员登录名”和“密码”。
1.  单击“创建”以导出数据库。

单击“创建”会创建导出数据库请求并将其提交到服务。根据数据库的大小，导出操作可能需要一些时间才能完成。

## 监视导出操作的进度

2.	单击“浏览全部”。
3.	单击“SQL Server”。
2.	单击包含你刚导出的原始（源）数据库的服务器。
3.	在 SQL 服务器边栏选项卡中，单击“导入/导出历史记录”：

    ![导入导出历史记录][3] 
    ![导入导出历史记录][4]

## 确认 BACPAC 位于你的存储容器中

2.	单击“浏览全部”。
3.	单击“存储帐户（经典）”。
2.	单击你在其中存储 BACPAC 的存储帐户。
3.	单击“容器”，然后选择数据库所导出到的容器以了解备份详细信息（从这里可以下载和保存 BACPAC）。

    ![.bacpac 文件详细信息][5]


## 后续步骤

- [导入 Azure SQL 数据库](/documentation/articles/sql-database-import)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)
- [SQL 数据库文档](/documentation/services/sql-database/)


<!--Image references-->
[1]: ./media/sql-database-export/export.png
[2]: ./media/sql-database-export/export-blade.png
[3]: ./media/sql-database-export/export-history.png
[4]: ./media/sql-database-export/export-status.png
[5]: ./media/sql-database-export/bacpac-details.png

<!---HONumber=74-->