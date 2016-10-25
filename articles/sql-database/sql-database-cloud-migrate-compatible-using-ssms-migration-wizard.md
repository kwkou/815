<properties
   pageTitle="使用“将数据库部署到 Azure 数据库”向导将 SQL Server 数据库迁移到 SQL 数据库"
   description="Azure SQL 数据库, 数据库迁移, Azure 数据库向导"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="sqldb-migrate"
   ms.date="08/24/2016"
   wacn.date="10/17/2016"
   ms.author="carlrab"/>  


# 使用“将数据库部署到 Azure 数据库”向导将 SQL Server 数据库迁移到 SQL 数据库


> [AZURE.SELECTOR]
- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/)
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/)

SQL Server Management Studio 中的“将数据库部署到 Azure 数据库”向导可将[兼容的 SQL Server 数据库](/documentation/articles/sql-database-cloud-migrate/)直接迁移到 Azure SQL 数据库服务器。

## 使用“将数据库部署到 Azure 数据库”向导

> [AZURE.NOTE] 以下步骤假定用户有预配的 SQL 数据库服务器。

1. 确认你安装了最新版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

    > [AZURE.IMPORTANT] 建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。

2. 打开 Management Studio 并连接到要在对象资源管理器中迁移的 SQL Server 数据库。
3. 右键单击对象资源管理器中的数据库，指向“任务”，然后单击“将数据库部署到 Azure SQL 数据库...”

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard01.png)  


4.	在部署向导中，单击“下一步”，然后单击“连接”以配置与 SQL 数据库服务器的连接。

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard002.png)  


5. 在“连接到服务器”对话框中，输入用于连接到 SQL 数据库服务器的连接信息。

	![通过“任务”菜单部署到 Azure](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard00.png)  


5.	为 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件提供以下内容，该文件是此向导在迁移过程中创建的：

 - **新数据库名称**
 - **Azure SQL 数据库的版本**（[服务层](/documentation/articles/sql-database-service-tiers/)）
 - **数据库最大大小**
 - **服务目标**（性能级别）
 - **临时文件名**

    ![导出设置](./media/sql-database-cloud-migrate/MigrateUsingDeploymentWizard02.png)  


6.	完成该向导。根据数据库的大小和复杂性，部署可能需要花费几分钟到几小时。如果此向导检测到兼容性问题，错误将显示到屏幕上，并且迁移将不会继续。有关如何修复数据库兼容性问题的指导，请转到[修复数据库兼容性问题](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)。

7.	使用对象资源管理器连接到 Azure SQL 数据库服务器中的已迁移数据库。
8.	使用 Azure 门户查看数据库及其属性。

## 后续步骤

- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)

## 其他资源

- [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/)
- [Transact-SQL 部分支持或不支持的函数](/documentation/articles/sql-database-transact-sql-information/)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_1010_2016-->