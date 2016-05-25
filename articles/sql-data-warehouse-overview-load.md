   <properties
   pageTitle="将数据载入 SQL 数据仓库 | Azure"
   description="了解有关在 SQL 数据仓库中加载数据的常见方案"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/28/2016"
   wacn.date="04/18/2016"/>

# 将数据载入 SQL 数据仓库
SQL 数据仓库提供许多选项用于加载数据，包括：

- PolyBase
- Azure 数据工厂
- BCP 命令行实用程序
- SQL Server 集成服务 (SSIS)
- 第三方数据加载工具

尽管上述所有方法可以配合 SQL 数据仓库使用，但 PolyBase 能够以透明方式并行处理 Azure Blob 存储的负载，这使它成为加载数据最快的工具。查看有关[使用 PolyBase 加载数据][]的更多信息。此外，由于许多用户都在寻求从本地源初始加载数百 GB 到数十 TB 的数据，因此在以下部分中，我们将针对初始数据的加载提供一些指导。

## 从 SQL Server 初始加载到 SQL 数据仓库
从本地 SQL Server 实例加载到 SQL 数据仓库时，建议采用以下步骤：

1. **BCP** SQL Server 数据转为平面文件
2. 使用 **AZCopy** 或**导入/导出**（适用于大型数据集）将文件转移到 Azure
3. 将 PolyBase 配置为从你的存储帐户读取你的文件
4. 使用 **PolyBase** 创建新表并加载数据

在以下部分中，我们将深入探讨每个步骤并提供过程示例。

> [AZURE.NOTE]从 SQL Server 等系统移动数据之前，建议参阅文档 Web 应用上的[迁移架构][]和[迁移代码][]文章。

## 使用 BCP 导出文件

若要准备将文件转移到 Azure，必须将它们导出到平面文件。最佳做法是使用 BCP 命令行实用程序。如果你还没有此实用程序，可以与[适用于 SQL Server 的 Microsoft 命令行实用程序][]一起下载。示例 BCP 命令如下所示：

```
bcp "select top 10 * from <table>" queryout "<Directory><File>" -c -T -S <Server Name> -d <Database Name> -- Export Query
or
bcp <table> out "<Directory><File>" -c -T -S <Server Name> -d <Database Name> -- Export Table
```

要使吞吐量最大化，你可以尝试并行执行进程，方法是为不同的表或者是单个表中的不同分区运行多个并发的 BCP 命令。这样可以在运行 BCP 的服务器的多个内核之间分布 BCP 使用的 CPU 资源。如果你从 SQL DW 或 PDW 系统提取数据，那么需要在 BCP 命令中添加 -q 参数（带引号的标识符）。如果你的环境不使用 Active Directory，那么你可能还需要添加 -u 和 -P 来指定用户名和密码。

此外，在使用 PolyBase 加载时，请注意，PolyBase 尚不支持 UTF-16，所有文件必须采用 UTF-8。可以轻松地完成此操作，方法是在 BCP 命令中包含“-c”标志，或者，可以使用以下代码将平面文件从 UTF-16 转换为 UTF-8：

```
Get-Content <input_file_name> -Encoding Unicode | Set-Content <output_file_name> -Encoding utf8
```

成功将数据导出到文件后，可以开始将它们移到 Azure。完成此操作的方法是使用 AZCopy 或使用导入/导出服务，如以下部分所述。

## 使用 AZCopy 或导入/导出加载到 Azure
如果要移动 5-10 TB 范围或以上的数据，建议使用我们的磁盘传送服务[导入/导出][]来完成移动。但是，在我们的研究中，我们已经能够使用公共 Internet 配合 AZCopy，轻松地移动单个数字 TB 范围中的数据。此过程还可以加速或通过 ExpressRoute 扩展。

以下步骤详细说明如何使用 AZCopy 将数据从本地移入 Azure 存储帐户。如果你在同一区域没有 Azure 存储帐户，可以遵循 [Azure 存储空间文档][]创建一个。还可以从不同区域中的存储帐户加载数据，但在此情况下，性能不是最佳的。

> [AZURE.NOTE] 本文档假设你已安装的 AZCopy 命令行实用程序，而且能够使用 Powershell 运行它。如果不是这样，请遵循 [AZCopy 安装说明][]。

现在，已提供一组使用 BCP 创建的文件，因此 AzCopy 只需从 Azure powershell 或通过运行 powershell 脚本来运行。在更高的级别中，运行 AZCopy 所需的提示将采用以下格式：

```
AZCopy /Source:<File Location> /Dest:<Storage Container Location> /destkey:<Storage Key> /Pattern:<File Name> /NC:256
```

除了基本做法外，我们还建议采用以下最佳实践，使用 AZCopy 加载：


+ **并发连接**：除了增加同时运行的 AZCopy 操作数目外，还可以设置 /NC 参数来进一步并行处理 AZCopy 操作本身，这对目标打开多个并发连接。尽管此参数最高可以设置为 512，但是我们发现最佳的数据传输发生在 256，因此建议测试值的数目，以确定哪个数目最适合你的配置。

+ **Express Route**：如上所述，如果启用 Express Route，可以加速此过程。ExpressRoute 和步骤的概述可在 [ExpressRoute 文档][]中找到。

+ **文件夹结构**：若要更轻松地使用 PolyBase 进行传输，请确定每个表都映射到自己的文件夹。这样，稍后使用 PolyBase 加载时，将简化步骤并将其减到最少。换言之，如果表分区成多个文件，或者即使是文件夹中的子目录，也没有任何影响。


## 配置 PolyBase

既然数据位于 Azure 存储 Blob 中，我们就可以使用 PolyBase 将它导入 SQL 数据仓库实例。以下步骤仅适用于配置，而且对于每个 SQL 数据仓库实例、用户或存储帐户，其中许多步骤只需完成一次即可。[使用 PolyBase 加载数据][]文档中也详细描述了这些步骤。

1. **创建数据库主密钥。** 每个数据库只需完成这项操作一次。

2. **创建数据库范围的凭据。** 仅当你要创建新的凭据/用户时，才需要创建此操作，否则可以使用前面创建的凭据。

3. **创建外部文件格式。** 外部文件格式也可以重复使用。仅当你要上载新类型的文件时，才需要创建一个格式。

4. **创建外部数据源。** 指向存储帐户时，如果从同一个容器加载，则你可以使用外部数据源。对于 'LOCATION' 参数，请使用以下格式的位置：'wasbs://mycontainer@ test.blob.core.chinacloudapi.cn/path'。

```
-- Creating master key
CREATE MASTER KEY;

-- Creating a database scoped credential
CREATE DATABASE SCOPED CREDENTIAL <Credential Name>
WITH
    IDENTITY = '<User Name>'
,   Secret = '<Azure Storage Key>'
;

-- Creating external file format (delimited text file)
CREATE EXTERNAL FILE FORMAT text_file_format
WITH
(
    FORMAT_TYPE = DELIMITEDTEXT
,   FORMAT_OPTIONS  (
                        FIELD_TERMINATOR ='|'
                    )
);

--Creating an external data source
CREATE EXTERNAL DATA SOURCE azure_storage
WITH
(
    TYPE = HADOOP
,   LOCATION ='wasbs://<Container>@<Blob Path>'
,   CREDENTIAL = <Credential Name>
)
;
```

适当配置存储帐户后，你可以继续将数据载入 SQL 数据仓库。

## 使用 PolyBase 加载数据
配置 PolyBase 后，只需创建一个外部表，指向存储中的数据，然后将该数据映射到 SQL 数据仓库内的新表，即可将数据直接载入 SQL 数据仓库。可以使用以下两个简单命令来完成此操作。

1. 使用 'CREATE EXTERNAL TABLE' 命令来定义数据结构。若要确保快速且有效地捕获数据的状态，我们建议在 SSMS 中编写 SQL Server 表的脚本，然后根据外部表的差异手动调整。在 Azure 中创建外部表之后，它将继续指向相同的位置，即使更新了数据或添加了其他数据。  

```
-- Creating external table pointing to file stored in Azure Storage
CREATE EXTERNAL TABLE <External Table Name>
(
    <Column name>, <Column type>, <NULL/NOT NULL>
)
WITH
(   LOCATION='<Folder Path>'
,   DATA_SOURCE = <Data Source>
,   FILE_FORMAT = <File Format>      
);
```

2. 使用 'CREATE TABLE...AS SELECT' 语句加载数据。

```
CREATE TABLE <Table Name>
WITH
(
	CLUSTERED COLUMNSTORE INDEX,
	DISTRIBUTION = <HASH(<Column Name>)>/<ROUND_ROBIN>
)
AS
SELECT  *
FROM    <External Table Name>
;
```

请注意，你还可以使用更详细的 SELECT 语句，从表加载行的子部分。但是，由于 PolyBase 目前不会将额外的计算资源推送到存储帐户，因此，如果你使用 SELECT 语句加载子部分，速度不会比加载整个数据集更快。

除了 `CREATE TABLE...AS SELECT` 语句外，还可以使用 'INSERT...INTO' 语句，将数据从外部表加载到预先存在的表。

##  基于新加载的数据创建统计信息

Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。有关统计信息的详细说明，请参阅开发主题组中的[统计信息][]主题。以下快速示例说明如何基于此示例中加载的表创建统计信息。


```
create statistics [<name>] on [<Table Name>] ([<Column Name>]);
create statistics [<another name>] on [<Table Name>] ([<Another Column Name>]);
```

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[Load data with bcp]: /documentation/articles/sql-data-warehouse-load-with-bcp
[使用 PolyBase 加载数据]: /documentation/articles/sql-data-warehouse-load-with-polybase
[solution partners]: /documentation/articles/sql-data-warehouse-solution-partners
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop
[迁移架构]: /documentation/articles/sql-data-warehouse-migrate-schema
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code
[统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics

<!--MSDN references-->
[supported source/sink]: https://msdn.microsoft.com/zh-cn/library/dn894007.aspx
[copy activity]: https://msdn.microsoft.com/zh-cn/library/dn835035.aspx
[SQL Server destination adapter]: https://msdn.microsoft.com/zh-cn/library/ms141237.aspx
[SSIS]: https://msdn.microsoft.com/zh-cn/library/ms141026.aspx

<!--Other Web references-->
[AZCopy 安装说明]: /documentation/articles/storage-use-azcopy/
[适用于 SQL Server 的 Microsoft 命令行实用程序]: http://www.microsoft.com/zh-cn/download/details.aspx?id=36433
[导入/导出]: /documentation/articles/storage-import-export-service/
[Azure 存储空间文档]: /documentation/articles/storage-create-storage-account/
[ExpressRoute 文档]: /documentation/services/expressroute/

<!---HONumber=Mooncake_0411_2016-->