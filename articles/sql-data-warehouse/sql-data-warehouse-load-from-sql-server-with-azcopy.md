<properties
   pageTitle="将数据从 SQL Server 载入 Azure SQL 数据仓库 (PolyBase) | Azure"
   description="使用 bcp 将数据从 SQL Server 导出到平面文件，使用 AZCopy 将数据导入 Azure Blob 存储，使用 PolyBase 将数据引入 Azure SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/30/2016"
   wacn.date="08/08/2016"/>


# 将数据从 SQL Server 载入 Azure SQL 数据仓库 (AZCopy)

使用 bcp 和 AZCopy 命令行实用工具将数据从 SQL Server 加载到 Azure Blob 存储。然后使用 PolyBase 或 Azure 数据工厂将数据载入 Azure SQL 数据仓库。


## 先决条件

若要逐步完成本教程，你需要：

- 一个 SQL 数据仓库数据库。
- 已安装 bcp 命令行实用工具
- 已安装 SQLCMD 命令行实用工具

>[AZURE.NOTE] 可以从 [Microsoft 下载中心][]下载 bcp 和 sqlcmd 实用工具。

## 将数据导入 SQL 数据仓库

在本教程中，你将在 Azure SQL 数据仓库中创建一个表，然后将数据导入该表。

### 步骤 1：在 Azure SQL 数据仓库中创建表

在命令提示符下，使用 sqlcmd 运行以下查询，以在实例上创建表：


    sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
        CREATE TABLE DimDate2
        (
            DateId INT NOT NULL,
            CalendarQuarter TINYINT NOT NULL,
            FiscalQuarter TINYINT NOT NULL
        )
        WITH
        (
            CLUSTERED COLUMNSTORE INDEX,
            DISTRIBUTION = ROUND_ROBIN
        );
    "


>[AZURE.NOTE] 有关在 SQL 数据仓库中创建表和 WITH 子句中可用选项的详细信息，请参阅 [CREATE TABLE syntax][]（CREATE TABLE 语法）。

### 步骤 2：创建源数据文件

打开记事本，将以下几行数据复制到新文本文件，然后将此文件保存到本地临时目录，路径为 C:\Temp\DimDate2.txt。


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


> [AZURE.NOTE] 请务必记得 bcp.exe 不支持 UTF-8 文件编码。使用 bcp.exe 时，请使用 ASCII 文件或 UTF-16 编码的文件。

### 步骤 3：连接并导入数据
在 bcp 中，可以使用以下命令来连接并导入数据（相应地替换其中的值）：


    bcp DimDate2 in C:\Temp\DimDate2.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t  ','


可以使用 sqlcmd 运行以下查询来验证是否已加载数据：


    sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "SELECT * FROM DimDate2 ORDER BY 1;"


应返回以下结果：

DateId |CalendarQuarter |FiscalQuarter
----------- |--------------- |-------------
20150101 |1 |3
20150201 |1 |3
20150301 |1 |3
20150401 |2 |4
20150501 |2 |4
20150601 |2 |4
20150701 |3 |1
20150801 |3 |1
20150801 |3 |1
20151001 |4 |2
20151101 |4 |2
20151201 |4 |2

### 步骤 4：基于新加载的数据创建统计信息

Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。有关统计信息的详细说明，请参阅开发主题组中的[统计信息][]主题。以下快速示例说明如何基于此示例中加载的表创建统计信息

在 sqlcmd 提示符下执行以下 CREATE STATISTICS 语句：


    sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
        create statistics [DateId] on [DimDate2] ([DateId]);
        create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
        create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
    "


## 从 SQL 数据仓库导出数据
在本教程中，你将从 Azure SQL 数据仓库中的表创建数据文件。我们将上面创建的数据导出到名为 DimDate2_export.txt 的新数据文件。

### 步骤 1：导出数据

在 bcp 实用程序中，可以使用以下命令来连接并导出数据（相应地替换其中的值）：


    bcp DimDate2 out C:\Temp\DimDate2_export.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t ','

你可以通过打开新文件来验证是否已正确导出数据。文件中的数据应与以下文本匹配：


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


>[AZURE.NOTE] 由于分布式系统的性质，数据顺序在不同 SQL 数据仓库数据库之间可能不同。另一种做法是使用 bcp 的 **queryout** 函数来编写查询提取，而不是导出整个表。

## 后续步骤
有关加载数据的概述，请参阅[将数据载入 SQL 数据仓库][]。
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][]。

<!--Image references-->

<!--Article references-->

[将数据载入 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-overview-load/
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[Table Design]: /documentation/articles/sql-data-warehouse-tables-overview/
[统计信息]: /documentation/articles/sql-data-warehouse-tables-statistics/

<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/zh-cn/library/ms162802.aspx
[CREATE TABLE syntax]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx

<!--Other Web references-->
[Microsoft 下载中心]: https://www.microsoft.com/download/details.aspx?id=36433

<!---HONumber=Mooncake_0801_2016-->