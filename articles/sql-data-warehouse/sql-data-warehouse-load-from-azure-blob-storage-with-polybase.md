<properties
   pageTitle="将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase) | Azure"
   description="了解如何使用 PolyBase 将数据从 Azure Blob 存储载入 SQL 数据仓库。将公共数据中的一些表载入 Contoso 零售数据仓库架构。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="ckarst"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="01/25/2017"
   ms.author="cakarst;barbkess;sonyama"/>  



# 将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)

> [AZURE.SELECTOR]
- [PolyBase](/documentation/articles/sql-data-warehouse-load-from-azure-blob-storage-with-polybase/)

使用 PolyBase 和 T-SQL 命令可将数据从 Azure Blob 存储载入 Azure SQL 数据仓库。

为简单起见，本教程会将两个表从公共 Azure 存储 Blob 载入 Contoso 零售数据仓库架构。若要加载完整的数据集，请运行 Microsoft SQL Server 示例存储库中的 [Load the full Contoso Retail Data Warehouse][Load the full Contoso Retail Data Warehouse]（加载完整的 Contoso 零售数据仓库）示例。

在本教程中你将：

1. 配置 PolyBase 以从 Azure Blob 存储加载数据
2. 将公共数据载入数据库
3. 完成加载后执行优化。

## 开始之前
若要运行本教程，需要一个已包含 SQL 数据仓库数据库的 Azure 帐户。如果没有此帐户，请参阅 [Create a SQL Data Warehouse][Create a SQL Data Warehouse]（创建 SQL 数据仓库）。

## 1\.配置数据源
PolyBase 使用 T-SQL 外部对象来定义外部数据的位置和属性。外部对象定义存储在 SQL 数据仓库中。数据本身存储在外部。

### 1\.1.创建凭据
如果要加载 Contoso 公共数据，请**跳过此步骤**。不需要以安全方式访问公共数据，因为它已经可供任何人访问。

如果你使用本教程作为加载自己数据的模板，请**不要跳过此步骤**。若要通过凭据访问数据，请使用以下脚本创建数据库范围的凭据，然后在定义数据源的位置时使用该凭据。



	-- A: Create a master key.
	-- Only necessary if one does not already exist.
	-- Required to encrypt the credential secret in the next step.

	CREATE MASTER KEY;


	-- B: Create a database scoped credential
	-- IDENTITY: Provide any string, it is not used for authentication to Azure storage.
	-- SECRET: Provide your Azure storage account key.


	CREATE DATABASE SCOPED CREDENTIAL AzureStorageCredential
	WITH
	    IDENTITY = 'user',
	    SECRET = '<azure_storage_account_key>'
	;


	-- C: Create an external data source
	-- TYPE: HADOOP - PolyBase uses Hadoop APIs to access data in Azure blob storage.
	-- LOCATION: Provide Azure storage account name and blob container name.
	-- CREDENTIAL: Provide the credential created in the previous step.

	CREATE EXTERNAL DATA SOURCE AzureStorage
	WITH (
	    TYPE = HADOOP,
	    LOCATION = 'wasbs://<blob_container_name>@<azure_storage_account_name>.blob.core.chinacloudapi.cn',
	    CREDENTIAL = AzureStorageCredential
	);


跳到步骤 2。

### 1\.2.创建外部数据源
使用 [CREATE EXTERNAL DATA SOURCE][CREATE EXTERNAL DATA SOURCE] 命令存储数据的位置以及数据的类型。


	CREATE EXTERNAL DATA SOURCE AzureStorage_west_public
	WITH 
	(  
	    TYPE = Hadoop 
	,   LOCATION = 'wasbs://contosoretaildw-tables@contosoretaildw.blob.core.chinacloudapi.cn/'
	); 


> [AZURE.IMPORTANT] 如果你选择公开 azure blob 存储容器，请记住，由于你是数据所有者，因此在数据离开数据中心时，需要支付数据传出费用。

## 2\.配置数据格式

数据存储在 Azure Blob 存储中的文本文件内，每个字段以分隔符隔开。运行 [CREATE EXTERNAL FILE FORMAT][] 命令，指定文本文件中数据的格式。Contoso 数据未压缩，以坚线分隔。


	CREATE EXTERNAL FILE FORMAT TextFileFormat 
	WITH 
	(   FORMAT_TYPE = DELIMITEDTEXT
	,	FORMAT_OPTIONS	(   FIELD_TERMINATOR = '|'
						,	STRING_DELIMITER = ''
						,	DATE_FORMAT		 = 'yyyy-MM-dd HH:mm:ss.fff'
						,	USE_TYPE_DEFAULT = FALSE 
						)
	);


## 3\.创建外部表
指定数据源和文件格式后，可以开始创建外部表。

### 3\.1.创建数据的架构。
若要创建一个位置用于存储数据库中的 Contoso 数据，请创建架构。


	CREATE SCHEMA [asb]
	GO


### 3\.2.创建外部表。
运行此脚本以创建 DimProduct 和 FactOnlineSales 外部表。在这里，我们只需定义列名和数据类型，然后将其绑定到 Azure blob 存储文件的位置和格式。定义存储在 SQL 数据仓库中，数据仍位于 Azure 存储 Blob 中。

**LOCATION** 参数是 Azure 存储 Blob 中根文件夹下的文件夹。每个表位于不同的文件夹中。



	--DimProduct
	CREATE EXTERNAL TABLE [asb].DimProduct (
		[ProductKey] [int] NOT NULL,
		[ProductLabel] [nvarchar](255) NULL,
		[ProductName] [nvarchar](500) NULL,
		[ProductDescription] [nvarchar](400) NULL,
		[ProductSubcategoryKey] [int] NULL,
		[Manufacturer] [nvarchar](50) NULL,
		[BrandName] [nvarchar](50) NULL,
		[ClassID] [nvarchar](10) NULL,
		[ClassName] [nvarchar](20) NULL,
		[StyleID] [nvarchar](10) NULL,
		[StyleName] [nvarchar](20) NULL,
		[ColorID] [nvarchar](10) NULL,
		[ColorName] [nvarchar](20) NOT NULL,
		[Size] [nvarchar](50) NULL,
		[SizeRange] [nvarchar](50) NULL,
		[SizeUnitMeasureID] [nvarchar](20) NULL,
		[Weight] [float] NULL,
		[WeightUnitMeasureID] [nvarchar](20) NULL,
		[UnitOfMeasureID] [nvarchar](10) NULL,
		[UnitOfMeasureName] [nvarchar](40) NULL,
		[StockTypeID] [nvarchar](10) NULL,
		[StockTypeName] [nvarchar](40) NULL,
		[UnitCost] [money] NULL,
		[UnitPrice] [money] NULL,
		[AvailableForSaleDate] [datetime] NULL,
		[StopSaleDate] [datetime] NULL,
		[Status] [nvarchar](7) NULL,
		[ImageURL] [nvarchar](150) NULL,
		[ProductURL] [nvarchar](150) NULL,
		[ETLLoadID] [int] NULL,
		[LoadDate] [datetime] NULL,
		[UpdateDate] [datetime] NULL
	)
	WITH
	(
	    LOCATION='/DimProduct/' 
	,   DATA_SOURCE = AzureStorage_west_public
	,   FILE_FORMAT = TextFileFormat
	,   REJECT_TYPE = VALUE
	,   REJECT_VALUE = 0
	)
	;
 
	--FactOnlineSales
	CREATE EXTERNAL TABLE [asb].FactOnlineSales 
	(
		[OnlineSalesKey] [int]  NOT NULL,
		[DateKey] [datetime] NOT NULL,
		[StoreKey] [int] NOT NULL,
		[ProductKey] [int] NOT NULL,
		[PromotionKey] [int] NOT NULL,
		[CurrencyKey] [int] NOT NULL,
		[CustomerKey] [int] NOT NULL,
		[SalesOrderNumber] [nvarchar](20) NOT NULL,
		[SalesOrderLineNumber] [int] NULL,
		[SalesQuantity] [int] NOT NULL,
		[SalesAmount] [money] NOT NULL,
		[ReturnQuantity] [int] NOT NULL,
		[ReturnAmount] [money] NULL,
		[DiscountQuantity] [int] NULL,
		[DiscountAmount] [money] NULL,
		[TotalCost] [money] NOT NULL,
		[UnitCost] [money] NULL,
		[UnitPrice] [money] NULL,
		[ETLLoadID] [int] NULL,
		[LoadDate] [datetime] NULL,
		[UpdateDate] [datetime] NULL
	)
	WITH
	(
	    LOCATION='/FactOnlineSales/' 
	,   DATA_SOURCE = AzureStorage_west_public
	,   FILE_FORMAT = TextFileFormat
	,   REJECT_TYPE = VALUE
	,   REJECT_VALUE = 0
	)
	;


## 4\.加载数据
可通过其他方式访问外部数据。可以直接从外部表查询数据、将数据载入新数据库表，或者将外部数据添加到现有数据库表。

### 4\.1.创建新架构
CTAS 可创建包含数据的新表。首先，请创建 contoso 数据的架构。


	CREATE SCHEMA [cso]
	GO


### 4\.2.将数据载入新表
若要从 Azure Blob 存储加载数据并将其保存到数据库中的某个表内，请使用 [CREATE TABLE AS SELECT (Transact-SQL)][CREATE TABLE AS SELECT (Transact-SQL)] 语句。使用 CTAS 加载可以利用你刚刚创建的强类型化外部表。若要将数据载入新表，请对每个表使用一个 [CTAS][CTAS] 语句。

CTAS 将创建新表，并在该表中填充 select 语句的结果。CTAS 将新表定义为包含与 select 语句结果相同的列和数据类型。如果你选择了外部表中的所有列，新表将是外部表中的列和数据类型的副本。

在此示例中，我们将以哈希分布表的形式创建维度表和事实表。



	SELECT GETDATE();
	GO

	CREATE TABLE [cso].[DimProduct]            WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[DimProduct]             OPTION (LABEL = 'CTAS : Load [cso].[DimProduct]             ');
	CREATE TABLE [cso].[FactOnlineSales]       WITH (DISTRIBUTION = HASH([ProductKey]  ) ) AS SELECT * FROM [asb].[FactOnlineSales]        OPTION (LABEL = 'CTAS : Load [cso].[FactOnlineSales]        ');


### 4\.3 跟踪加载进度
可以使用动态管理视图 (DMV) 跟踪加载操作的进度。


	-- To see all requests
	SELECT * FROM sys.dm_pdw_exec_requests;

	-- To see a particular request identified by its label
	SELECT * FROM sys.dm_pdw_exec_requests as r;
	WHERE r.label = 'CTAS : Load [cso].[DimProduct]             '
	      OR r.label = 'CTAS : Load [cso].[FactOnlineSales]        '
	;

	-- To track bytes and files
	SELECT
	    r.command,
	    s.request_id,
	    r.status,
	    count(distinct input_name) as nbr_files, 
	    sum(s.bytes_processed)/1024/1024 as gb_processed
	FROM
	    sys.dm_pdw_exec_requests r
	    inner join sys.dm_pdw_dms_external_work s
	        on r.request_id = s.request_id
	WHERE 
	    r.[label] = 'CTAS : Load [cso].[DimProduct]             '
	    OR r.[label] = 'CTAS : Load [cso].[FactOnlineSales]        '
	GROUP BY
	    r.command,
	    s.request_id,
	    r.status
	ORDER BY
	    nbr_files desc,
	    gb_processed desc;


## 5\.优化列存储压缩
默认情况下，SQL 数据仓库将表存储为聚集列存储索引。加载完成后，某些数据行可能未压缩到列存储中。发生这种情况的原因多种多样。若要了解详细信息，请参阅[管理列存储索引][manage columnstore indexes]。

若要在加载后优化查询性能和列存储压缩，请重新生成表，以强制列存储索引压缩所有行。


	SELECT GETDATE();
	GO

	ALTER INDEX ALL ON [cso].[DimProduct]               REBUILD;
	ALTER INDEX ALL ON [cso].[FactOnlineSales]          REBUILD;


有关维护列存储索引的详细信息，请参阅[管理列存储索引][manage columnstore indexes]一文。

## 6\.优化统计信息
最好是在加载之后马上创建单列统计信息。对于统计信息，可以使用多个选项。例如，如果针对每个列创建单列统计信息，则重新生成所有统计信息可能需要花费很长时间。如果你知道某些列不会在查询谓词中使用，可以不创建有关这些列的统计信息。

如果决定针对每个表的每个列创建单列统计信息，可以使用 [statistics][]（统计信息）一文中的存储过程代码示例 `prc_sqldw_create_stats`。

以下示例是创建统计信息的不错起点。它会针对维度表中的每个列以及事实表中的每个联接列创建单列统计信息。以后，你随时可以将单列或多列统计信息添加到其他事实表列。


	CREATE STATISTICS [stat_cso_DimProduct_AvailableForSaleDate] ON [cso].[DimProduct]([AvailableForSaleDate]);
	CREATE STATISTICS [stat_cso_DimProduct_BrandName] ON [cso].[DimProduct]([BrandName]);
	CREATE STATISTICS [stat_cso_DimProduct_ClassID] ON [cso].[DimProduct]([ClassID]);
	CREATE STATISTICS [stat_cso_DimProduct_ClassName] ON [cso].[DimProduct]([ClassName]);
	CREATE STATISTICS [stat_cso_DimProduct_ColorID] ON [cso].[DimProduct]([ColorID]);
	CREATE STATISTICS [stat_cso_DimProduct_ColorName] ON [cso].[DimProduct]([ColorName]);
	CREATE STATISTICS [stat_cso_DimProduct_ETLLoadID] ON [cso].[DimProduct]([ETLLoadID]);
	CREATE STATISTICS [stat_cso_DimProduct_ImageURL] ON [cso].[DimProduct]([ImageURL]);
	CREATE STATISTICS [stat_cso_DimProduct_LoadDate] ON [cso].[DimProduct]([LoadDate]);
	CREATE STATISTICS [stat_cso_DimProduct_Manufacturer] ON [cso].[DimProduct]([Manufacturer]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductDescription] ON [cso].[DimProduct]([ProductDescription]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductKey] ON [cso].[DimProduct]([ProductKey]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductLabel] ON [cso].[DimProduct]([ProductLabel]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductName] ON [cso].[DimProduct]([ProductName]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductSubcategoryKey] ON [cso].[DimProduct]([ProductSubcategoryKey]);
	CREATE STATISTICS [stat_cso_DimProduct_ProductURL] ON [cso].[DimProduct]([ProductURL]);
	CREATE STATISTICS [stat_cso_DimProduct_Size] ON [cso].[DimProduct]([Size]);
	CREATE STATISTICS [stat_cso_DimProduct_SizeRange] ON [cso].[DimProduct]([SizeRange]);
	CREATE STATISTICS [stat_cso_DimProduct_SizeUnitMeasureID] ON [cso].[DimProduct]([SizeUnitMeasureID]);
	CREATE STATISTICS [stat_cso_DimProduct_Status] ON [cso].[DimProduct]([Status]);
	CREATE STATISTICS [stat_cso_DimProduct_StockTypeID] ON [cso].[DimProduct]([StockTypeID]);
	CREATE STATISTICS [stat_cso_DimProduct_StockTypeName] ON [cso].[DimProduct]([StockTypeName]);
	CREATE STATISTICS [stat_cso_DimProduct_StopSaleDate] ON [cso].[DimProduct]([StopSaleDate]);
	CREATE STATISTICS [stat_cso_DimProduct_StyleID] ON [cso].[DimProduct]([StyleID]);
	CREATE STATISTICS [stat_cso_DimProduct_StyleName] ON [cso].[DimProduct]([StyleName]);
	CREATE STATISTICS [stat_cso_DimProduct_UnitCost] ON [cso].[DimProduct]([UnitCost]);
	CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureID] ON [cso].[DimProduct]([UnitOfMeasureID]);
	CREATE STATISTICS [stat_cso_DimProduct_UnitOfMeasureName] ON [cso].[DimProduct]([UnitOfMeasureName]);
	CREATE STATISTICS [stat_cso_DimProduct_UnitPrice] ON [cso].[DimProduct]([UnitPrice]);
	CREATE STATISTICS [stat_cso_DimProduct_UpdateDate] ON [cso].[DimProduct]([UpdateDate]);
	CREATE STATISTICS [stat_cso_DimProduct_Weight] ON [cso].[DimProduct]([Weight]);
	CREATE STATISTICS [stat_cso_DimProduct_WeightUnitMeasureID] ON [cso].[DimProduct]([WeightUnitMeasureID]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_CurrencyKey] ON [cso].[FactOnlineSales]([CurrencyKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_CustomerKey] ON [cso].[FactOnlineSales]([CustomerKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_DateKey] ON [cso].[FactOnlineSales]([DateKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_OnlineSalesKey] ON [cso].[FactOnlineSales]([OnlineSalesKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_ProductKey] ON [cso].[FactOnlineSales]([ProductKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_PromotionKey] ON [cso].[FactOnlineSales]([PromotionKey]);
	CREATE STATISTICS [stat_cso_FactOnlineSales_StoreKey] ON [cso].[FactOnlineSales]([StoreKey]);


## 大功告成！
你已成功地将公共数据载入 Azure SQL 数据仓库。干得不错！

现在，你可以使用如下所示的查询，开始查询表：


	SELECT  SUM(f.[SalesAmount]) AS [sales_by_brand_amount]
	,       p.[BrandName]
	FROM    [cso].[FactOnlineSales] AS f
	JOIN    [cso].[DimProduct]      AS p ON f.[ProductKey] = p.[ProductKey]
	GROUP BY p.[BrandName]


## 后续步骤
若要加载整个 Contoso 零售数据仓库数据，可以使用脚本。
有关更多开发技巧，请参阅 [SQL Data Warehouse development overview][SQL Data Warehouse development overview]（SQL 数据仓库开发概述）。

<!--Image references-->


<!--Article references-->
[Create a SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Load data into SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-overview-load/
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[manage columnstore indexes]: /documentation/articles/sql-data-warehouse-tables-index/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[label]: /documentation/articles/sql-data-warehouse-develop-label/

<!--MSDN references-->
[CREATE EXTERNAL DATA SOURCE]: https://msdn.microsoft.com/zh-cn/library/dn935022.aspx
[CREATE EXTERNAL FILE FORMAT]: https://msdn.microsoft.com/zh-cn/library/dn935026.aspx
[CREATE TABLE AS SELECT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt204041.aspx
[sys.dm_pdw_exec_requests]: https://msdn.microsoft.com/zh-cn/library/mt203887.aspx
[REBUILD]: https://msdn.microsoft.com/zh-cn/library/ms188388.aspx

<!--Other Web references-->

[Microsoft Download Center]: http://www.microsoft.com/download/details.aspx?id=36433
[Load the full Contoso Retail Data Warehouse]: https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/contoso-data-warehouse/readme.md

<!---HONumber=Mooncake_Quality_Review_0125_2017-->