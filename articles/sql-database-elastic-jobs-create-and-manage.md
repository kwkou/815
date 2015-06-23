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
	ms.date="04/29/2015"
	wacn.date="06/23/2015"/>

# 创建和管理弹性数据库作业

**弹性数据库池**提供可预测的模型用于部署大量数据库。你可以设置每个数据库的最小数据吞吐量单位 (DTU)，而不会超过预算。使用**弹性数据库作业**可以非常轻松地管理这些数据库中的公共对象。该服务允许你以单个操作，针对池中的所有数据库运行 T-SQL 脚本。例如，可以对每个数据库设置策略，以便只允许具有适当凭据的用户查看敏感数据。

## 先决条件

* Azure 订阅。若要获取试用版，请参阅[试用](/pricing/1rmb-trial/)。
* 一个弹性数据库池。请参阅关于[弹性数据库池](sql-database-elastic-pool)
* 安装弹性数据库作业服务组件。请参阅[安装弹性数据库作业服务](vsql-database-elastic-jobs-service-installation)。

## 创建作业

1. 在弹性数据库作业池边栏选项卡，单击“创建作业”。
2. 键入数据库管理员（在安装时创建）的名称和密码。
2. 在“创建作业”边栏选项卡中，键入作业的名称。
3. 粘贴或键入 T-SQL 脚本。
4. 单击“保存”，然后单击“运行”。

	![为作业命名，键入或粘贴代码，然后单击“运行”][1]

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
[5]: ./media/sql-database-elastic-jobs-create-and-manage/provide-creds.png

<!---HONumber=61-->