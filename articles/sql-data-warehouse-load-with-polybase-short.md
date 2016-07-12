<properties
   pageTitle="使用 PolyBase 加载数据教程 | Azure"
   description="了解如何使用 PolyBase 将数据载入 SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahajs"
   manager="jhubbard"
   editor="sahajs"/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="11/04/2015"
   wacn.date="01/20/2016"/>


# 使用 PolyBase 加载数据

> [AZURE.SELECTOR]
- [PolyBase](/documentation/articles/sql-data-warehouse-load-with-polybase-short/)
- [BCP](/documentation/articles/sql-data-warehouse-load-with-bcp/)

本教程说明如何使用 PolyBase 将数据载入 Azure SQL 数据仓库。若要了解有关 PolyBase 的详细信息，请参阅 [SQL 数据仓库中的 PolyBase 教程][]。


## 先决条件
若要逐步完成本教程中，你需要：

- SQL 数据仓库数据库
- Azure 存储帐户


## 步骤 1：创建源数据文件
打开记事本并将以下数据行复制到一个新文件。将此文件保存到本地临时目录，路径为 C:\\Temp\\DimDate2.txt。

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```


## 步骤 2：将数据迁移到 Azure Blob 存储

- 下载[最新版本的 AzCopy][]。
- 打开一个命令窗口，然后导航到你的计算机上的 AzCopy 安装目录，AzCopy.exe 位于其中。默认情况下，在 64 位 Windows 计算机上，安装目录为 %ProgramFiles(x86)%\\Microsoft SDKs\\Azure\\AzCopy\\。
- 运行以下命令以上载该文件。为 /DestKey 指定 Azure 存储帐户密钥。

```
.\AzCopy.exe /Source:C:\Temp\ /Dest:https://pbdemostorage.blob.core.chinacloudapi.cn/datacontainer/datedimension/ /DestKey:<azure_storage_account_key> /Pattern:DimDate2.txt
```

有关 AzCopy 的详细信息，请参阅 [AzCopy 命令行实用程序入门][]。


## 步骤 3：创建外部表

接下来，需要在 SQL 数据仓库中创建外部表以引用 Azure Blob 存储中的数据。若要创建外部表，请使用以下步骤：

- [创建主密钥][]：加密数据库范围凭据的机密。
- [创建数据库范围的凭据]：指定 Azure 存储帐户的身份验证信息。
- [创建外部数据源]：指定 Azure Blob 存储的位置。
- [创建外部文件格式]：指定数据的布局。
- [创建外部表]：引用 Azure 存储空间数据。


```
-- A: Create Master Key
-- Required to encrypt the credential secret in the next step.
CREATE MASTER KEY;
-- B: Create Database Scoped Credential
-- Provide the Azure storage account key. The identity name does not affect authentication to Azure storage.
CREATE DATABASE SCOPED CREDENTIAL AzureStorageCredential 
WITH IDENTITY = 'user', 
SECRET = '<azure_storage_account_key>';
-- C: Create External Data Source
-- Specify location and credential to access your Azure blob storage.
CREATE EXTERNAL DATA SOURCE AzureStorage 
WITH (	
		TYPE = Hadoop, 
		LOCATION = 'wasbs://datacontainer@pbdemostorage.blob.core.chinacloudapi.cn',
		CREDENTIAL = AzureStorageCredential
); 
-- D: Create External File format 
-- Specify the layout of data stored in Azure storage blobs. 
CREATE EXTERNAL FILE FORMAT TextFile 
WITH (FORMAT_TYPE = DelimitedText, 
FORMAT_OPTIONS (FIELD_TERMINATOR = ','));


-- E: Create External Table
-- To reference data stored in Azure blob storage.
CREATE EXTERNAL TABLE dbo.DimDate2External (
	DateId INT NOT NULL, 
	CalendarQuarter TINYINT NOT NULL, 
	FiscalQuarter TINYINT NOT NULL
)
WITH (
		LOCATION='datedimension/', 
		DATA_SOURCE=AzureStorage, 
		FILE_FORMAT=TextFile
);
-- Run a query on external table to confirm that the Azure Storage data can be referenced.
SELECT count(*) FROM dbo.DimDate2External;
```



## 步骤 4：将数据载入 SQL 数据仓库

- 若要将数据载入新表，请运行 [CREATE TABLE AS SELECT (Transact-SQL)][] 语句。新表继承查询中指定的列。它从外部表定义继承这些列的数据类型。 
- 若要将数据载入现有表，请使用 INSERT...SELECT 语句。  


```
-- Load the data from Azure blob storage to SQL Data Warehouse
CREATE TABLE dbo.DimDate2
WITH 
(   
    CLUSTERED COLUMNSTORE INDEX,
    DISTRIBUTION = ROUND_ROBIN
)
AS 
SELECT * 
FROM   [dbo].[DimDate2External];
```


## 步骤 5：基于新加载的数据创建统计信息 

Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。有关统计信息的详细说明，请参阅开发主题组中的[统计信息][]主题。以下快速示例说明如何基于此示例中加载的表创建统计信息


```
create statistics [DateId] on [DimDate2] ([DateId]);
create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
```

<!--Article references-->
[SQL 数据仓库中的 PolyBase 教程]: /documentation/articles/sql-data-warehouse-load-with-polybase/


<!-- External Links -->
[最新版本的 AzCopy]: http://aka.ms/downloadazcopy
[AzCopy 命令行实用程序入门]: /documentation/articles/storage-use-azcopy/

[创建外部数据源]: https://msdn.microsoft.com/zh-cn/library/dn935022(v=sql.130).aspx
[创建外部文件格式]: https://msdn.microsoft.com/zh-cn/library/dn935026(v=sql.130).aspx
[创建外部表]: https://msdn.microsoft.com/zh-cn/library/dn935021(v=sql.130).aspx
[创建主密钥]: https://msdn.microsoft.com/zh-cn/library/ms174382.aspx
[创建数据库范围的凭据]: https://msdn.microsoft.com/zh-cn/library/mt270260.aspx
[CREATE TABLE AS SELECT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt204041.aspx


<!--Article references-->

[统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics/

<!---HONumber=Mooncake_1207_2015-->
