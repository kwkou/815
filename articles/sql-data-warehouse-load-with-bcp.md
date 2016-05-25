<properties
   pageTitle="使用 bcp 将数据载入 SQL 数据仓库 | Azure"
   description="了解什么是 bcp，以及如何将它用于数据仓库方案。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="TwoUnder"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>


# 使用 bcp 加载数据

> [AZURE.SELECTOR]
- [BCP](/documentation/articles/sql-data-warehouse-load-with-bcp)


**[bcp][]** 是一个命令行批量加载实用程序，可让你在 SQL Server、数据文件和 SQL 数据仓库之间复制数据。使用 bcp 实用程序可将大量的行导入 SQL 数据仓库表，或将 SQL Server 表中的数据导出到数据文件。除非与 queryout 选项一起使用，否则 bcp 不需要 Transact-SQL 方面的知识。

bcp 是将较小数据集移入和移出 SQL 数据仓库数据库的快速轻松方式。通过 bcp 加载/提取数据时，建议的确切数据量取决于 Azure 数据中心的网络连接。通常可以加载和提取维度表，但加载或提取相当大的事实表则可能需要很长的时间。

使用 bcp 可以：

- 使用简单的命令行实用程序将数据载入 SQL 数据仓库。
- 使用简单的命令行实用程序从 SQL 数据仓库提取数据。

本教程将说明如何：

- 使用 bcp in 命令将数据导入表中
- 使用 bcp out 命令从表中导出数据

## 先决条件

若要逐步完成本教程，你需要：

- 一个 SQL 数据仓库数据库。
- 已安装 bcp 命令行实用程序
- 已安装 SQLCMD 命令行实用程序

>[AZURE.NOTE] 可以从 [Microsoft 下载中心][]下载 bcp 和 sqlcmd 实用程序。

## 将数据导入 SQL 数据仓库

在本教程中，你将在 Azure SQL 数据仓库中创建一个表，然后将数据导入该表。

### 步骤 1：在 Azure SQL 数据仓库中创建表

从命令提示符下，使用以下命令连接到你的实例（相应地替换其中的值）：

```
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I
```
连接后，在 sqlcmd 提示符下复制以下表脚本，然后按 Enter 键：

```
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
GO
```
>[AZURE.NOTE] 有关 WITH 子句中可用选项的详细信息，请参阅“开发”主题组中的[表设计][]。

### 步骤 2：创建源数据文件

打开记事本并将以下数据行复制到一个新文件。

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

将此文件保存到本地临时目录，路径为 C:\\Temp\\DimDate2.txt。

> [AZURE.NOTE] 请务必记得 bcp.exe 不支持 UTF-8 文件编码。使用 bcp.exe 时，请对文件使用 ASCII 编码文件或 UTF-16 编码。

### 步骤 3：连接并导入数据
在 bcp 中，可以使用以下命令来连接并导入数据（相应地替换其中的值）：

```
bcp DimDate2 in C:\Temp\DimDate2.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t  ','
```

像前面一样使用 sqlcmd 来建立连接，然后执行以下 TSQL 命令以验证是否加载了数据：

```
SELECT * FROM DimDate2 ORDER BY 1;
GO
```

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

```
create statistics [DateId] on [DimDate2] ([DateId]);
create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
GO
```

## 从 SQL 数据仓库导出数据
在本教程中，你将从 Azure SQL 数据仓库中的表创建数据文件。我们将上面创建的数据导出到名为 DimDate2\_export.txt 的新数据文件。

### 步骤 1：导出数据

在 bcp 实用程序中，可以使用以下命令来连接并导出数据（相应地替换其中的值）：

```
bcp DimDate2 out C:\Temp\DimDate2_export.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t ','
```
你可以通过打开新文件来验证是否已正确导出数据。文件中的数据应与以下文本匹配：

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

>[AZURE.NOTE] 由于分布式系统的性质，数据顺序在不同 SQL 数据仓库数据库之间可能不同。你可以选择性地使用 queryout 参数来指定要运行的 Transact-SQL 查询。

## 后续步骤
有关加载数据的概述，请参阅[将数据载入 SQL 数据仓库][]。
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][]。

<!--Image references-->

<!--Article references-->

[将数据载入 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-overview-load
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-develop
[表设计]: /documentation/articles/sql-data-warehouse-develop-table-design
[统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics


<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/zh-cn/library/ms162802.aspx


<!--Other Web references-->
[Microsoft 下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?id=36433

<!---HONumber=Mooncake_0307_2016-->