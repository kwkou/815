<!-- Remove MSDN subscription benifits & azure portal sqlDW & load samples  -->
<properties
    pageTitle="使用 TSQL 创建 SQL 数据仓库 | Azure"
    description="了解如何使用 TSQL 创建 Azure SQL 数据仓库"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    tags="azure-sql-data-warehouse"
    translationtype="Human Translation" />
<tags
    ms.assetid="a4e2e68e-aa9c-4dd3-abb0-f7df997d237a"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="04/24/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="ba65e3b4a76405fdad5e09e5622fee6abcec0639"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-sql-data-warehouse-database-by-using-transact-sql-tsql"></a>使用 Transact-SQL (TSQL) 创建 SQL 数据仓库数据库
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-data-warehouse-get-started-provision/)
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文说明如何使用 T-SQL 创建 SQL 数据仓库。

## <a name="prerequisites"></a>先决条件
若要开始，您需要：

* **Azure 帐户**：访问 [Azure 免费试用版][Azure Free Trial]或者 [MSDN Azure 信用额度][MSDN Azure Credits]，以创建帐户。
* **Azure SQL 服务器**：有关详细信息，请参阅[使用 Azure 门户创建 Azure SQL 数据库逻辑服务器][使用 Azure 门户创建 Azure SQL 数据库逻辑服务器]或[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器][使用 PowerShell 创建 Azure SQL 数据库逻辑服务器]。
* **资源组**：可使用同一资源组作为 Azure SQL Server，或参阅[如何创建资源组][how to create a resource group]。
* **执行 T-SQL 的环境**：可以使用 [Visual Studio][Installing Visual Studio and SSDT]、[sqlcmd][sqlcmd] 或 [SSMS][SSMS] 执行 T-SQL。

> [AZURE.NOTE]
> 创建 SQL 数据仓库可能会导致新的计费服务。  有关定价的详细信息，请参阅 [SQL 数据仓库定价][SQL Data Warehouse pricing]。
>
>

## <a name="create-a-database-with-visual-studio"></a>使用 Visual Studio 创建数据库
如果不熟悉 Visual Studio，请参阅文章[查询 Azure SQL 数据仓库 (Visual Studio)][Query Azure SQL Data Warehouse (Visual Studio)]。  若要开始操作，请在 Visual Studio 中打开 SQL Server 对象资源管理器，并连接到要托管 SQL 数据仓库数据库的服务器。  连接后，可针对 **master** 数据库运行以下 SQL 命令来创建 SQL 数据仓库。  此命令创建服务目标为 DW400 的数据库 MySqlDwDb，并允许此数据库增长到大小上限 10 TB。

    CREATE DATABASE MySqlDwDb COLLATE SQL_Latin1_General_CP1_CI_AS (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB);

## <a name="create-a-database-with-sqlcmd"></a>使用 sqlcmd 创建数据库
也可以在命令提示符处运行以下命令，以使用 sqlcmd 运行相同的命令。

    sqlcmd -S <Server Name>.database.windows.net -I -U <User> -P <Password> -Q "CREATE DATABASE MySqlDwDb COLLATE SQL_Latin1_General_CP1_CI_AS (EDITION='datawarehouse', SERVICE_OBJECTIVE = 'DW400', MAXSIZE= 10240 GB)"

在不指定的情况下，默认排序规则是 COLLATE SQL_Latin1_General_CP1_CI_AS。  `MAXSIZE` 可介于 250 GB 和 240 TB 之间。  `SERVICE_OBJECTIVE` 可以介于 DW100 与 DW2000 [DWU][DWU] 之间。  有关所有有效值的列表，请参阅 [CREATE DATABASE][CREATE DATABASE] 的 MSDN 文档。  MAXSIZE 和 SERVICE_OBJECTIVE 可通过 [ALTER DATABASE][ALTER DATABASE] T-SQL 命令进行更改。  数据库的排序规则在创建后不能更改。   更改 SERVICE_OBJECTIVE 时应谨慎，因为更改 DWU 会导致服务重新启动，这会取消所有正在进行的查询。  更改 MAXSIZE 不会重启服务，因为这只是简单的元数据操作。

## <a name="next-steps"></a>后续步骤
完成预配 SQL 数据仓库后，可以[加载示例数据][load sample data]或了解如何[开发][develop]、[加载][load]或[迁移][migrate]数据。

<!--Article references-->
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is/
[how to create a SQL Data Warehouse from the Azure portal]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Query Azure SQL Data Warehouse (Visual Studio)]: /documentation/articles/sql-data-warehouse-query-visual-studio/
[migrate]: /documentation/articles/sql-data-warehouse-overview-migrate/
[develop]: /documentation/articles/sql-data-warehouse-overview-develop/
[load]: /documentation/articles/sql-data-warehouse-overview-load/
[load sample data]: /documentation/articles/sql-data-warehouse-load-sample-databases/
[Create an Azure SQL database with the Azure Portal]: /documentation/articles/sql-database-get-started/#create-an-azure-sql-database-logical-server
[Create an Azure SQL database with PowerShell]: /documentation/articles/sql-database-create-and-configure-database-powershell
[how to create a resource group]: /documentation/articles/resource-group-template-deploy-portal/#create-resource-group
[Installing Visual Studio and SSDT]: /documentation/articles/sql-data-warehouse-install-visual-studio/
[sqlcmd]: /documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/

<!--MSDN references--> 
[CREATE DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx
[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx
[SSMS]: https://msdn.microsoft.com/zh-cn/library/mt238290.aspx

<!--Other Web references-->
[SQL Data Warehouse pricing]: /pricing/details/sql-data-warehouse/
[Azure Free Trial]: /pricing/1rmb-trial/
[MSDN Azure Credits]: /pricing/member-offers/

<!--Update_Description:update meta properties;wording update-->