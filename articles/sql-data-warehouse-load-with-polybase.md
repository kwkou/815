<properties
   pageTitle="SQL 数据仓库中的 PolyBase 教程 | Azure"
   description="了解什么是 PolyBase，以及如何将其用于数据仓库方案。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="jhubbard"
   editor="jrowlandjones"/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="11/04/2015"
   wacn.date="01/20/2016"/>


# 使用 PolyBase 加载数据
PolyBase 技术可让你完全使用 Transact-SQL 命令来查询和联接多个源的数据。

使用 PolyBase 时，可以通过执行以下步骤来查询存储在 Azure Blob 存储中的数据，再将数据载入 SQL 数据仓库数据库：

- 创建数据库主密钥和凭据。
- 创建 PolyBase 对象：外部数据源、外部文件格式和外部表。 
- 查询存储在 Azure Blob 存储中的数据。
- 将数据从 Azure Blob 存储载入 SQL 数据仓库。
- 将数据从 SQL 数据仓库导出到 Azure Blob 存储。


## 先决条件
若要逐步完成本教程，你需要：

- Azure 存储帐户
- 在 Azure Blob 存储中存储为分隔文本文件的数据

首先，你要创建当 PolyBase 连接到 Azure Blob 存储和查询数据时所需的对象。

> [AZURE.IMPORTANT]PolyBase 支持的 Azure 存储帐户类型为：
> 
> + 标准本地冗余存储 (Standard-LRS)
> + 标准异地冗余存储 (Standard-GRS)
> + 标准读取访问权限异地冗余存储 (Standard-RAGRS)
>
> PolyBase 不支持标准区域冗余存储 (Standard-ZRS) 和高级本地冗余存储 (Premium-LRS) 帐户类型。如果你要创建新的 Azure 存储帐户，请确保从定价层中选择 PolyBase 支持的存储帐户类型。

## 步骤 1：将凭据存储在数据库中
若要访问 Azure Blob 存储，需要创建数据库范围的凭据，其中存储 Azure 存储帐户的身份验证信息。遵循以下步骤存储数据库的凭据。

1. 连接到你的 SQL 数据仓库数据库。
2. 使用 [CREATE MASTER KEY (Transact-SQL)][] 创建数据库的主密钥。如果数据库已有主密钥，则不需要创建另一个主密钥。在下一个步骤中，将使用此密钥来加密凭据“机密”。

```
-- Create a E master key
CREATE MASTER KEY;
```

4. 检查你是否已有任何数据库凭据。为此，请使用 sys.database\_credentials 系统视图，而不要使用只显示服务器凭据的 sys.credentials。

``` 
    -- 检查是否存在数据库范围的凭据。SELECT * FROM sys.database\_credentials;
```
5. 使用 [CREATE CREDENTIAL (Transact-SQL)][] 为你想要访问的每个 Azure 存储帐户创建数据库范围的凭据。在此示例中，IDENTITY 是凭据的友好名称。它不影响对 Azure 存储空间的身份验证。SECRET 是 Azure 存储帐户密钥。

```
    -- 创建数据库范围凭据：
    CREATE DATABASE SCOPED CREDENTIAL ASBSecret
    WITH IDENTITY = 'joe' 
    , Secret = '<azure_storage_account_key>' 
    ; 
    ```

6. 如果需要删除某个数据库范围的凭据，请使用 [DROP CREDENTIAL (Transact-SQL)][]：

```
-- Dropping credential
DROP DATABASE SCOPED CREDENTIAL ASBSecret
;
```

## 步骤 2：创建外部数据源
外部数据源是存储 Azure Blob 存储数据位置及访问信息的数据库对象。使用 [CREATE EXTERNAL DATA SOURCE (Transact-SQL)][] 为你想要访问的每个 Azure 存储 Blob 定义外部数据源。

```
    -- Create an external data source for an Azure storage blob
    CREATE EXTERNAL DATA SOURCE azure_storage 
    WITH
    (
        TYPE = HADOOP,
        LOCATION ='wasbs://mycontainer@test.blob.core.chinacloudapi.cn',
        CREDENTIAL = ASBSecret
    )
    ;
```

如果需要删除外部表，请使用 [DROP EXTERNAL DATA SOURCE][]：

```
    -- Drop an external data source
    DROP EXTERNAL DATA SOURCE azure_storage
    ;
```

## 步骤 3：创建外部文件格式
外部文件格式是指定外部数据格式的数据库对象。PolyBase 可以处理压缩和未压缩的数据，包括分隔的文本、Hive RCFILE 和 HIVE ORC 格式。

使用 [CREATE EXTERNAL FILE FORMAT (Transact-SQL)][] 来创建外部文件格式。此示例指定文件中的数据为未压缩的文本，字段以竖线字符（“|”）分隔。

```
-- Create an external file format for a text-delimited file.
-- Data is uncompressed and fields are separated with the
-- pipe character.
CREATE EXTERNAL FILE FORMAT text_file_format 
WITH 
(   
    FORMAT_TYPE = DELIMITEDTEXT, 
    FORMAT_OPTIONS  
    (
        FIELD_TERMINATOR ='|',
        USE_TYPE_DEFAULT = TRUE
    )
)
;
```

如果需要删除某个外部文件格式，请使用 [DROP EXTERNAL FILE FORMAT]。

```
-- Dropping external file format
DROP EXTERNAL FILE FORMAT text_file_format
;
```

## 创建外部表

外部表定义类似于关系表定义。主要区别在于数据的位置和格式。

- 外部表定义以元数据的形式存储在 SQL 数据仓库数据库中。 
- 数据存储在数据源所指定的位置。

使用 [CREATE EXTERNAL TABLE (Transact-SQL)][] 来定义外部表。

LOCATION 选项指定从数据源根目录到数据的路径。在此示例中，数据位于 'wasbs://mycontainer@test.Blob.core.chinacloudapi.cn/path/Demo/'。同一个表的所有文件必须在 Azure Blob 存储中相同的逻辑文件夹下。

你也可以选择指定拒绝选项（REJECT\_TYPE、REJECT\_VALUE、REJECT\_SAMPLE\_VALUE），以确定 PolyBase 如何处理从外部数据源收到的脏记录。

```
-- Creating an external table for data in Azure blob storage.
CREATE EXTERNAL TABLE [ext].[CarSensor_Data] 
(
     [SensorKey]     int    NOT NULL,
     [CustomerKey]   int    NOT NULL,
     [GeographyKey]  int        NULL,
     [Speed]         float  NOT NULL,
     [YearMeasured]  int    NOT NULL,
)
WITH 
(
    LOCATION    = '/Demo/',
    DATA_SOURCE = azure_storage,
    FILE_FORMAT = text_file_format      
)
;
```

刚刚创建的对象存储在 SQL 数据仓库数据库中。可以在 SQL Server Data Tools (SSDT) 对象资源管理器中查看它们。

若要删除外部表，需要使用以下语法：

```
--Dropping external table
DROP EXTERNAL TABLE [ext].[CarSensor_Data]
;
```

> [AZURE.NOTE]删除外部表时，必须使用 `DROP EXTERNAL TABLE`。**不能**使用 `DROP TABLE`。

参考主题：[DROP EXTERNAL TABLE (Transact-SQL)][]。

另外值得注意的是，在 `sys.tables` 和 `sys.external_tables` 目录视图（后者更明确）中都会显示外部表。

## 轮换存储密钥

有时出于安全考虑，想要更改 Blob 存储的访问密钥。

执行此任务的最佳方式是遵循称为“轮换密钥”的过程。你可能已注意到，Blob 存储帐户有两个存储密钥。这样你便可以转换

轮换 Azure 存储帐户密钥的过程只需简单的三个步骤

1. 创建基于辅助存储访问密钥的第二个数据库范围凭据
2. 创建基于新凭据的第二个外部数据源
3. 删除并创建指向新外部数据源的外部表

当你将所有外部表迁移到新的外部数据源时，可以执行清理任务：
 
1. 删除第一个外部数据源
2. 删除基于主存储访问密钥的第一个数据库范围凭据
3. 登录 Azure 并重新生成主访问密钥供下一次使用

## 查询 Azure Blob 存储数据
针对外部表的查询只使用表名称，如关系表一样。


```
-- Query Azure storage resident data via external table. 
SELECT * FROM [ext].[CarSensor_Data]
;
```

> [AZURE.NOTE]对外部表的查询可能因“查询已中止 -- 从外部源读取时已达最大拒绝阈值”错误而失败。这表示外部数据包含*脏*记录。如果实际数据类型/列数目不匹配外部表的列定义，或数据不匹配指定的外部文件格式，则会将数据记录视为脏记录。若要解决此问题，请确保外部表和外部文件格式定义正确，并且外部数据符合这些定义。如果外部数据记录的子集是脏的，你可以通过使用 CREATE EXTERNAL TABLE DDL 中的拒绝选项，选择拒绝这些查询记录。


## 从 Azure Blob 存储加载数据
此示例将 Azure Blob 存储中的数据加载到 SQL 数据仓库数据库中。

直接存储数据可免除查询时的数据传输时间。配合列存储索引存储数据可让分析查询的查询性能提升 10 倍。

此示例使用 CREATE TABLE AS SELECT 语句来加载数据。新表继承查询中指定的列。它从外部表定义继承这些列的数据类型。

CREATE TABLE AS SELECT 是高性能 TRANSACT-SQL 语句，可将数据并行加载到 SQL 数据仓库的所有计算节点。它最初是针对分析平台系统中的大规模并行处理 (MPP) 引擎开发的，现已纳入 SQL 数据仓库。

```
	-- Load data from Azure blob storage to SQL Data Warehouse 	
	CREATE TABLE [dbo].[Customer_Speed]
	WITH 
	(   
	    CLUSTERED COLUMNSTORE INDEX
	,	DISTRIBUTION = HASH([CarSensor_Data].[CustomerKey])
	)
	AS 
	SELECT * 
	FROM   [ext].[CarSensor_Data]
	;
```

请参阅 [CREATE TABLE AS SELECT (Transact-SQL)][]。

## 基于新加载的数据创建统计信息

Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。有关统计信息的详细说明，请参阅开发主题组中的[统计信息][]主题。以下快速示例说明如何基于此示例中加载的表创建统计信息。

```
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## 将数据导出到 Azure Blob 存储
本部分说明如何将数据从 SQL 数据仓库导出到 Azure Blob 存储。此示例使用 CREATE EXTERNAL TABLE AS SELECT（高性能 TRANSACT-SQL 语句）将数据从所有计算节点并行导出。

以下示例使用 dbo.Weblogs 表中的列定义和数据从 dbo 创建外部表 Weblogs2014。外部表定义存储在 SQL 数据仓库中，SELECT 语句的结果将导出到数据源所指定的 Blob 容器下的“/archive/log2014/”目录中。将以指定的文本文件格式导出数据。

```
	CREATE EXTERNAL TABLE Weblogs2014 WITH
	(
	    LOCATION='/archive/log2014/',
	    DATA_SOURCE=azure_storage,
	    FILE_FORMAT=text_file_format
	)
	AS
	SELECT
	    Uri,
	    DateRequested
	FROM
	    dbo.Weblogs
	WHERE
	    1=1
	    AND DateRequested > '12/31/2013'
	    AND DateRequested < '01/01/2015';
```


## 满足 PolyBase UTF-8 要求
目前 PolyBase 支持加载 UTF-8 编码的数据文件。由于 UTF-8 使用与 ASCII 相同的字符编码，PolyBase 也支持加载 ASCII 编码的数据。但是，PolyBase 不支持其他字符编码，例如 UTF-16/Unicode 或扩展的 ASCII 字符。扩展的 ASCII 包括具有重音符号的字符，例如德语中常见的变音符号。

满足这项要求的最佳方法是重新编写为 UTF-8 编码。

有多种方法可实现此目的。以下是两种使用 Powershell 的方法：

### 适用于小文件的简单示例

以下是用于创建文件的一行简单 Powershell 脚本。
 
```
Get-Content <input_file_name> -Encoding Unicode | Set-Content <output_file_name> -Encoding utf8
```

但是，尽管将数据重新编码的方法非常简单，但绝非最有效的做法。以下 IO 流式处理示例要快得多，并可达到相同的效果。

### 适用于较大文件的 IO 流式处理示例

以下代码示例更为复杂，但在流式处理从源到目标的数据行时要有效得多。请对较大的文件应用此方法。

	#Static variables
	$ascii = [System.Text.Encoding]::ASCII
	$utf16le = [System.Text.Encoding]::Unicode
	$utf8 = [System.Text.Encoding]::UTF8
	$ansi = [System.Text.Encoding]::Default
	$append = $False
	
	#Set source file path and file name
	$src = [System.IO.Path]::Combine("C:\input_file_path","input_file_name.txt")
	
	#Set source file encoding (using list above)
	$src_enc = $ansi
	
	#Set target file path and file name
	$tgt = [System.IO.Path]::Combine("C:\output_file_path","output_file_name.txt")
	
	#Set target file encoding (using list above)
	$tgt_enc = $utf8
	
	$read = New-Object System.IO.StreamReader($src,$src_enc)
	$write = New-Object System.IO.StreamWriter($tgt,$append,$tgt_enc)
	
	while ($read.Peek() -ne -1)
	{
	    $line = $read.ReadLine();
	    $write.WriteLine($line);
	}
	$read.Close()
	$read.Dispose()
	$write.Close()
	$write.Dispose()


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[Load data with bcp]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[Load with PolyBase]: /documentation/articles/sql-data-warehouse-load-with-polybase/
[solution partners]: /documentation/articles/sql-data-warehouse-solution-partners/
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics/

<!--MSDN references-->
[supported source/sink]: https://msdn.microsoft.com/zh-cn/library/dn894007.aspx
[copy activity]: https://msdn.microsoft.com/zh-cn/library/dn835035.aspx
[SQL Server destination adapter]: https://msdn.microsoft.com/zh-cn/library/ms141095.aspx
[SSIS]: https://msdn.microsoft.com/zh-cn/library/ms141026.aspx


<!-- External Links -->
[CREATE EXTERNAL DATA SOURCE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/dn935022(v=sql.130).aspx
[CREATE EXTERNAL FILE FORMAT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/dn935026(v=sql.130).aspx
[CREATE EXTERNAL TABLE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/dn935021(v=sql.130).aspx

[DROP EXTERNAL DATA SOURCE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt146367.aspx
[DROP EXTERNAL FILE FORMAT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt146379.aspx
[DROP EXTERNAL TABLE (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt130698.aspx

[CREATE TABLE AS SELECT (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/mt204041.aspx
[CREATE MASTER KEY (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms174382.aspx
[CREATE CREDENTIAL (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms189522.aspx
[DROP CREDENTIAL (Transact-SQL)]: https://msdn.microsoft.com/zh-cn/library/ms189450.aspx

<!---HONumber=Mooncake_1207_2015-->
