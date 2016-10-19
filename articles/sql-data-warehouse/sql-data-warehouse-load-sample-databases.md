<properties
   pageTitle="将示例数据载入 SQL 数据仓库 | Azure"
   description="将示例数据载入 SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="08/16/2016"
   wacn.date="10/17/2016"/>  


#将示例数据载入 SQL 数据仓库

请遵循以下简单步骤，加载并查询 Adventure Works 示例数据库。这些脚本首先使用 sqlcmd 运行 SQL，这会创建表和视图。创建了表之后，脚本会使用 bcp 加载数据。如果还没有安装 sqlcmd和 bcp，请单击以下链接[安装 bcp][] 并[安装 sqlcmd][]。

##加载示例数据

1. 下载[适用于 SQL 数据仓库的 Adventure Works 示例脚本][] zip 文件。

2. 将下载的 zip 中的文件解压缩到本地计算机上的目录。

3. 编辑解压缩的 aw\_create.bat 文件，并设置位于文件顶部的以下变量。切勿在“=”和参数之间留有空格。下面是编辑后的内容示例。

        server=mylogicalserver.database.chinacloudapi.cn
        user=mydwuser
        password=Mydwpassw0rd
        database=mydwdatabase
    

4. 从 Windows 命令提示符运行编辑过的 aw\_create.bat。确保你所在的目录是保存了所编辑 aw\_create.bat 版本的位置。
此脚本将...
	* 删除所有 Adventure Works 表或所有已在你数据库中的视图
	* 创建 Adventure Works 表和视图
	* 使用 bcp 加载每个 Adventure Works 表
	* 验证每个 Adventure Works 表的行计数
	* 收集每个 Adventure Works 表的所有列中的统计信息


##查询示例数据

将一些示例数据载入 SQL 数据仓库后，你可以快速运行几个查询。若要运行查询，请使用 Visual Studio 和 SSDT 连接到 Azure SQL DW 中新建的 Adventure Works 数据库，如 [使用 Visual Studio 进行查询][] 文档中所述。

用于获取所有员工信息的简单 select 语句示例：


    SELECT * FROM DimEmployee;

下面是一个更复杂的查询示例，它使用构造（例如 GROUP BY）来查看每天所有销售活动的总金额：


    SELECT OrderDateKey, SUM(SalesAmount) AS TotalSales
    FROM FactInternetSales
    GROUP BY OrderDateKey
    ORDER BY OrderDateKey;


用于筛选出特定日期之前的订单的 SELECT 与 WHERE 子句示例：

    SELECT OrderDateKey, SUM(SalesAmount) AS TotalSales
    FROM FactInternetSales
    WHERE OrderDateKey > '20020801'
    GROUP BY OrderDateKey
    ORDER BY OrderDateKey;

SQL 数据仓库几乎支持 SQL Server 所能支持的所有 T-SQL 构造。[迁移代码][]文档中描述了两者的所有差别。

## 后续步骤
现在你可以配合示例数据尝试一些查询，接下来请了解如何[开发][]、[加载][]或[迁移][]到 SQL 数据仓库。

<!--Image references-->


<!--Article references-->
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate
[开发]: /documentation/articles/sql-data-warehouse-overview-develop
[加载]: /documentation/articles/sql-data-warehouse-overview-load
[连接]: /documentation/articles/sql-data-warehouse-query-visual-studio
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code
[安装 bcp]: /documentation/articles/sql-data-warehouse-load-with-bcp
[安装 sqlcmd]: /documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd

<!--Other Web references-->

[适用于 SQL 数据仓库的 Adventure Works 示例脚本]: https://migrhoststorage.blob.core.windows.net/sqldwsample/AdventureWorksSQLDW2012.zip

<!---HONumber=Mooncake_1010_2016-->