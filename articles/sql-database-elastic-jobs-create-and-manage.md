<properties
	pageTitle="创建和管理弹性数据库作业"
	description="演练如何创建和管理弹性数据库作业。"
	services="sql-database"
	documentationCenter=""
	manager="jhubbard"
	authors="sidneyh"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="07/21/2015"
	wacn.date="09/15/2015"/>

# 使用门户创建和管理 SQL 数据库弹性作业（预览版）

> [AZURE.SELECTOR]
- [Azure portal](/documentation/articles/sql-database-elastic-jobs-create-and-manage)
- [PowerShell](/documentation/articles/sql-database-elastic-jobs-powershell)


使用**弹性数据库作业**，可以通过执行架构更改、凭据管理、引用数据更新、性能数据收集或租户（客户）遥测数据收集等管理操作，来轻松可靠地管理一组数据库。目前可以通过 Azure 门户和 PowerShell cmdlet 使用弹性数据库作业。但是，它在 Azure 门户中的功能被缩减，仅限针对[弹性数据库池（预览版）](/documentation/articles/sql-database-elastic-pool)中的所有数据库执行。若要访问其他功能并针对包含自定义集合或分片集（使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-scale-introduction)创建）的一组数据库执行脚本，请参阅[使用 PowerShell 创建和管理作业](/documentation/articles/sql-database-elastic-jobs-powershell)。有关作业的详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview)。

## 先决条件

* Azure 订阅。若要获取试用版，请参阅[试用](/pricing/1rmb-trial/)。
* 一个弹性数据库池。请参阅关于[弹性数据库池](/documentation/articles/sql-database-elastic-pool)
* 安装弹性数据库作业服务组件。请参阅[安装弹性数据库作业服务](/documentation/articles/sql-database-elastic-jobs-service-installation)。

## 创建作业

1. 使用 [Azure 门户](https://manage.windowsazure.cn)，从现有的弹性数据库作业池中单击“创建作业”。
2. 键入作业控制数据库（作业的元数据存储）的数据库管理员（在安装作业时创建）的用户名和密码。

	![为作业命名，键入或粘贴代码，然后单击“运行”][1]
2. 在“创建作业”边栏选项卡中，键入作业的名称。
3. 键入具有足够权限、用于连接到目标数据库的用户名和密码，使脚本执行成功。
4. 粘贴或键入 T-SQL 脚本。
5. 单击“保存”，然后单击“运行”。

	![创建并运行作业][5]

## 运行幂等作业

当你针对一组数据库运行某个脚本时，必须确保该脚本是幂等的。也就是说，该脚本必须能够运行多次，即使它在进入未完成状态之前已失败。例如，当脚本失败时，作业将自动重试直到成功（在限制次数内，因为重试逻辑最终会停止重试）。执行此操作的方法是使用一个“IF EXISTS”子句，并在创建新对象之前删除找到的任何实例。下面显示了一个示例：

	IF EXISTS (SELECT name FROM sys.indexes
            WHERE name = N'IX_ProductVendor_VendorID')
    DROP INDEX IX_ProductVendor_VendorID ON Purchasing.ProductVendor;
	GO
	CREATE INDEX IX_ProductVendor_VendorID
    ON Purchasing.ProductVendor (VendorID);

或者，在创建新实例之前使用“IF NOT EXISTS”子句：

	IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
	BEGIN
	 CREATE TABLE TestTable(
	  TestTableId INT PRIMARY KEY IDENTITY,
	  InsertionTime DATETIME2
	 );
	END
	GO

	INSERT INTO TestTable(InsertionTime) VALUES (sysutcdatetime());
	GO

然后，此脚本将更新以前创建的表。

	IF NOT EXISTS (SELECT columns.name FROM sys.columns INNER JOIN sys.tables on columns.object_id = tables.object_id WHERE tables.name = 'TestTable' AND columns.name = 'AdditionalInformation')
	BEGIN

	ALTER TABLE TestTable

	ADD AdditionalInformation NVARCHAR(400);
	END
	GO

	INSERT INTO TestTable(InsertionTime, AdditionalInformation) VALUES (sysutcdatetime(), 'test');
	GO


## 检查作业状态

作业开始后，你可以检查其进度。

1. 在弹性数据库池页中，单击“管理作业”。

	![单击“管理作业”。][2]

2. 单击作业的名称 (a)。“状态”可能是“已完成”或“失败”。 将显示作业的详细信息 (b) 及其创建和运行日期与时间。以下列表 (c) 显示了针对池中每个数据库运行的脚本的进度，同时还提供了该脚本的日期和时间详细信息。

	![检查已完成的作业][3]


## 检查失败的作业

如果作业失败，可以查找该作业的执行日志。单击失败作业的名称即可查看其详细信息。

![检查失败的作业][4]


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-create-and-manage/screen-1.png
[2]: ./media/sql-database-elastic-jobs-create-and-manage/click-manage-jobs.png
[3]: ./media/sql-database-elastic-jobs-create-and-manage/running-jobs.png
[4]: ./media/sql-database-elastic-jobs-create-and-manage/failed.png
[5]: ./media/sql-database-elastic-jobs-create-and-manage/screen-2.png

 

<!---HONumber=69-->