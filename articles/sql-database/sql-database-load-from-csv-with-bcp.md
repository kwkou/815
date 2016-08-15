<properties
   pageTitle="将数据从 CSV 文件载入 Azure SQL 数据库 (bcp) | Azure"
   description="对于较小的数据，请使用 bcp 将数据导入到 Azure SQL 数据库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="06/30/2016"
   wacn.date="08/15/2016"/>


# 将数据从 CSV 载入 Azure SQL 数据仓库（平面文件）

你可以使用 bcp 命令行实用程序将数据从 CSV 文件导入 Azure SQL 数据库。

## 开始之前

### 先决条件

若要逐步完成本教程，你需要：

- Azure SQL 数据库逻辑服务器和数据库
- 已安装 bcp 命令行实用工具
- 已安装 sqlcmd 命令行实用工具

可以从 [Microsoft 下载中心][]下载 bcp 和 sqlcmd 实用程序。

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
	    ;
	"



## 2\.创建源数据文件

打开记事本，将以下几行数据复制到新文本文件，然后将此文件保存到本地临时目录，路径为 C:\\Temp\\DimDate2.txt。此数据采用 ASCII 格式。


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


## 后续步骤

若要迁移 SQL Server 数据库，请参阅 [SQL Server 数据库迁移](/documentation/articles/sql-database-cloud-migrate/)。

<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/zh-cn/library/ms162802.aspx
[CREATE TABLE syntax]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx

<!--Other Web references-->
[Microsoft 下载中心]: https://www.microsoft.com/download/details.aspx?id=36433

<!---HONumber=Mooncake_0808_2016-->