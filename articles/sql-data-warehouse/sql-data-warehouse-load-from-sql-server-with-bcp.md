<!-- Temp remove statistics and table overview -->
<properties
   pageTitle="将数据从 SQL Server 载入 Azure SQL 数据仓库 (bcp) | Azure"
   description="对于少量的数据，可以使用 bcp 将数据从 SQL Server 导出到平面文件，然后将数据直接导入 Azure SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="12/12/2016"
   ms.author="lodipalm;barbkess;sonyama"/>


# 将数据从 SQL Server 载入 Azure SQL 数据仓库（平面文件）

> [AZURE.SELECTOR]
- [SSIS](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-integration-services/)
- [PolyBase](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-polybase/)
- [bcp](/documentation/articles/sql-data-warehouse-load-from-sql-server-with-bcp/)

对于小型数据集，可以使用 bcp 命令行实用工具从 SQL Server 导出数据，然后将其直接加载到 Azure SQL 数据仓库。

在本教程中，你将使用 bcp 来执行以下操作：

- 使用 bcp out 命令从 SQL Server 导出表（或创建简单的示例文件）
- 将表从平面文件导入 SQL 数据仓库。
- 针对加载的数据创建统计信息。



## 开始之前
### 先决条件
若要逐步完成本教程，你需要：

- 一个 SQL 数据仓库数据库。
- 已安装 bcp 命令行实用工具
- 已安装 sqlcmd 命令行实用工具

可以从 [Microsoft 下载中心][Microsoft Download Center]下载 bcp 和 sqlcmd 实用程序。

### 采用 ASCII 或 UTF-16 格式的数据
如果你使用自己的数据尝试学习本教程，则数据需要使用 ASCII 或 UTF-16 编码，因为 bcp 不支持 UTF-8。

PolyBase 支持 UTF-8，但尚不支持 UTF-16。请注意，如果你要结合使用 bcp 和 PolyBase，则从 SQL Server 导出数据后，需要将数据转换为 UTF-8。

## 1\.创建目标表
在 SQL 数据仓库中定义加载操作的目标表。该表中的列必须对应于数据文件每一行中的数据。

若要创建表，请打开命令提示符并使用 sqlcmd.exe 运行以下命令：


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



## 2\.创建源数据文件

打开记事本，将以下几行数据复制到新文本文件，然后将此文件保存到本地临时目录，路径为 C:\Temp\DimDate2.txt。此数据采用 ASCII 格式。


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


（可选）若要从 SQL Server 数据库导出自己的数据，请打开命令提示符并运行以下命令。将 TableName、ServerName、DatabaseName、Username 和 Password 替换为你自己的信息。


	bcp <TableName> out C:\Temp\DimDate2_export.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <Password> -q -c -t ','




## 3\.加载数据
若要加载数据，请打开命令提示符并运行以下命令，请注意将 Server Name、Database Name、Username 和 Password 替换为你自己的信息。


	bcp DimDate2 in C:\Temp\DimDate2.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <password> -q -c -t  ','


使用此命令来验证是否已正确加载数据


	sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "SELECT * FROM DimDate2 ORDER BY 1;"


结果应如下所示：

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

## 4\.创建统计信息
SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了获得最佳查询性能，在首次加载数据或者在数据发生重大更改之后，必须针对所有表的所有列创建统计信息。有关统计信息的详细说明，请参阅 [Statistics][Statistics]（统计信息）。

运行以下命令针对新加载的表创建统计信息。


	sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
	    create statistics [DateId] on [DimDate2] ([DateId]);
	    create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
	    create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
	"


## 5\.从 SQL 数据仓库导出数据
为了增加乐趣，你可以从 SQL 数据仓库中导出刚刚加载的数据。导出命令与从 SQL Server 导出所用的命令完全相同。

但是，结果会有所差异。由于数据存储在 SQL 数据仓库中的分散位置，因此，当你导出数据时，每个计算节点会将数据写入输出文件。输出文件中的数据顺序与输入文件中的数据顺序可能不同。

### 导出表并比较导出的结果
若要查看导出的数据，请打开命令提示符并使用自己的参数运行此命令。ServerName 是 Azure 逻辑 SQL Server 的名称。


	bcp DimDate2 out C:\Temp\DimDate2_export.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t ','

你可以通过打开新文件来验证是否已正确导出数据。文件中的数据应该与以下文本匹配，但可能以不同的顺序排序：


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


### 导出查询结果
可以使用 bcp 的 **queryout** 函数导出查询结果，而无需导出整个表。

## 后续步骤
有关加载数据的概述，请参阅[将数据载入 SQL 数据仓库][Load data into SQL Data Warehouse]。
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][SQL Data Warehouse development overview]。
有关在 SQL 数据仓库中创建表的详细信息，请参阅 [Table Overview][Table Overview]（表概述）或 [CREATE TABLE syntax][CREATE TABLE syntax]（CREATE TABLE 语法）。

<!--Image references-->


<!--Article references-->

[Load data into SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-overview-load/
[SQL Data Warehouse development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Table Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/

<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/zh-cn/library/ms162802.aspx
[CREATE TABLE syntax]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx

<!--Other Web references-->

[Microsoft Download Center]: https://www.microsoft.com/download/details.aspx?id=36433

<!---HONumber=Mooncake_1205_2016-->