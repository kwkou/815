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
   ms.date="07/11/2016"
   wacn.date="08/08/2016"/>

# 使用 Transact-SQL (TSQL) 创建 SQL 数据仓库数据库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文将演示如何使用 Transact-SQL (T-SQL) 创建 SQL 数据仓库数据库。

## 先决条件
在开始之前，请确保符合以下先决条件。

- **Azure 帐户**：若要创建帐户，请参阅 [Azure 试用版][] <!-- 或 [MSDN Azure 信用额度][] -->。
- **V12 Azure SQL Server**：请参阅 <!-- [使用 Azure 门户创建 Azure SQL 数据库逻辑服务器][]或 --> [使用 PowerShell 创建 Azure SQL 数据库逻辑服务器][]。
- **资源组名称**：若要创建新的资源组，可使用同一资源组作为 V12 Azure SQL Server，或参阅[资源组][]。
- **带 SQL Server Data Tools 的 Visual Studio**：有关安装说明，请参阅[安装 Visual Studio 和 SSDT][]。

> [AZURE.NOTE] 创建新的 SQL 数据仓库可能会导致新的计费服务。有关定价的更多详细信息，请参阅 [SQL Data Warehouse pricing][]（SQL 数据仓库定价）。

## 使用 Visual Studio 创建数据库

若不熟悉 Visual Studio，请参阅 [Connect to SQL Data Warehouse with Visual Studio][]（使用 Visual Studio 连接到 SQL 数据仓库）一文。若要开始操作，请在 Visual Studio 中打开 SQL Server 对象资源管理器，并连接到要托管 SQL 数据仓库数据库的服务器。连接后，可针对 **master** 数据库运行以下 SQL 命令来创建 SQL 数据仓库。此命令会创建服务目标为 DW400 的数据库 MySqlDwDb，并允许此数据库增长到大小上限 10 TB。


	CREATE DATABASE MySqlDwDb (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB);


## 使用 sqlcmd 创建数据库

或者，你可以在命令提示符处运行以下命令，以使用 sqlcmd 运行相同的命令。


	sqlcmd -S <Server Name>.database.chinacloudapi.cn -I -U <User> -P <Password> -Q "CREATE DATABASE MySqlDwDb (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB)"


`MAXSIZE` 可介于 250 GB 和 240 TB 之间。`SERVICE_OBJECTIVE` 可以介于 DW100 与 DW2000 [DWU][] 之间。有关所有有效值的列表，请参阅 [CREATE DATABASE][] 的 MSDN 文档。MAXSIZE 和 SERVICE\_OBJECTIVE 还可通过 [ALTER DATABASE][] T-SQL 命令进行更改。更改 SERVICE\_OBJECTIVE 时应谨慎，因为这会导致服务重新启动而取消所有正在进行的查询。更改 MAXSIZE 不会重启服务，因为这只是简单的元数据操作。

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][]或了解如何[开发][]、[加载][]或[迁移][]数据。

<!--Article references-->
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is#data-warehouse-units
[how to create a SQL Data Warehouse from the Azure portal]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[Connect to SQL Data Warehouse with Visual Studio]: /documentation/articles/sql-data-warehouse-get-started-connect/
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-overview-load/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[使用 Azure 门户创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started/#create-an-azure-sql-database-logical-server
[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started-powershell/#database-setup-create-a-resource-group-server-and-firewall-rule
[资源组]: /documentation/articles/resource-group-portal/
[安装 Visual Studio 和 SSDT]: /documentation/articles/sql-data-warehouse-install-visual-studio/


<!--MSDN references--> 
[CREATE DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx
[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx

<!--Other Web references-->
[SQL Data Warehouse pricing]: /pricing/details/sql-data-warehouse/
[Azure 试用版]: /pricing/1rmb-trial/
[MSDN Azure 信用额度]: /pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F

<!---HONumber=Mooncake_0801_2016-->