<!-- Remove MSDN subscription benifits & azure portal sqlDW & load samples  -->
<properties
   pageTitle="使用 TSQL 创建 SQL 数据仓库 | Azure"
   description="了解如何使用 TSQL 创建 Azure SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""
   tags="azure-sql-data-warehouse"/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="10/31/2016"
   wacn.date="01/03/2017"
   ms.author="lodipalm;barbkess;sonyama"/>  


# 使用 Transact-SQL (TSQL) 创建 SQL 数据仓库数据库

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-data-warehouse-get-started-provision/)
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文说明如何使用 T-SQL 创建 SQL 数据仓库。

## 先决条件
若要开始，您需要：

- **Azure 帐户**：访问 [Azure 试用版][]以创建帐户。
- **Azure SQL Server**：有关详细信息，请参阅[使用 Azure 门户创建 Azure SQL 数据库逻辑服务器][]或[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器][]。
- **资源组**：可使用同一资源组作为 Azure SQL Server，或参阅[如何创建资源组][]。
- **执行 T-SQL 的环境**：可以使用 [Visual Studio][Installing Visual Studio and SSDT]、[sqlcmd][] 或 [SSMS][] 执行 T-SQL。

> [AZURE.NOTE] 创建 SQL 数据仓库可能会导致新的计费服务。有关定价的详细信息，请参阅 [SQL 数据仓库定价][]。

## 使用 Visual Studio 创建数据库
若不熟悉 Visual Studio，请参阅 [Query Azure SQL Data Warehouse (Visual Studio)][Query Azure SQL Data Warehouse (Visual Studio)]（查询 Azure SQL 数据仓库 (Visual Studio)）一文。若要开始操作，请在 Visual Studio 中打开 SQL Server 对象资源管理器，并连接到要托管 SQL 数据仓库数据库的服务器。连接后，可针对 **master** 数据库运行以下 SQL 命令来创建 SQL 数据仓库。此命令创建服务目标为 DW400 的数据库 MySqlDwDb，并允许此数据库增长到大小上限 10 TB。


	CREATE DATABASE MySqlDwDb COLLATE SQL_Latin1_General_CP1_CI_AS (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB);


## 使用 sqlcmd 创建数据库
也可以在命令提示符处运行以下命令，以使用 sqlcmd 运行相同的命令。


	sqlcmd -S <Server Name>.database.chinacloudapi.cn -I -U <User> -P <Password> -Q "CREATE DATABASE MySqlDwDb COLLATE SQL_Latin1_General_CP1_CI_AS(EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB)"


在不指定的情况下，默认排序规则是 COLLATE SQL\_Latin1\_General\_CP1\_CI\_AS。`MAXSIZE` 可介于 250 GB 和 240 TB 之间。`SERVICE_OBJECTIVE` 可以介于 DW100 与 DW2000 [DWU][DWU] 之间。有关所有有效值的列表，请参阅 [CREATE DATABASE][CREATE DATABASE] 的 MSDN 文档。MAXSIZE 和 SERVICE\_OBJECTIVE 可通过 [ALTER DATABASE][ALTER DATABASE] T-SQL 命令进行更改。数据库的排序规则在在创建后不能更改。更改 SERVICE\_OBJECTIVE 时应谨慎，因为更改 DWU 会导致服务重新启动而取消所有正在进行的查询。更改 MAXSIZE 不会重启服务，因为这只是简单的元数据操作。

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][load sample data]或了解如何[开发][develop]、[加载][load]，或[迁移][migrate]数据。

<!--Article references-->
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is/#data-warehouse-units
[how to create a SQL Data Warehouse from the Azure portal]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Query Azure SQL Data Warehouse (Visual Studio)]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[migrate]: /documentation/articles/sql-data-warehouse-overview-migrate/
[develop]: /documentation/articles/sql-data-warehouse-overview-develop/
[load]: /documentation/articles/sql-data-warehouse-overview-load/
[load sample data]: /documentation/articles/sql-data-warehouse-load-sample-databases/
[使用 Azure 门户预览创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started/#create-an-azure-sql-database-logical-server
[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started-powershell/#complete-azure-powershell-script-to-create-a-server-firewall-rule-and-database
[如何创建资源组]: /documentation/articles/resource-group-template-deploy-portal/#create-resource-group
[Installing Visual Studio and SSDT]: /documentation/articles/sql-data-warehouse-install-visual-studio/
[sqlcmd]: /documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/

<!--MSDN references--> 
[CREATE DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx
[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx
[SSMS]: https://msdn.microsoft.com/zh-cn/library/mt238290.aspx

<!--Other Web references-->
[SQL 数据仓库定价]: /pricing/details/sql-data-warehouse/
[Azure 试用版]: /pricing/1rmb-trial/
[MSDN Azure 信用额度]: /pricing/member-offers/

<!---HONumber=Mooncake_Quality_Review_1230_2016-->