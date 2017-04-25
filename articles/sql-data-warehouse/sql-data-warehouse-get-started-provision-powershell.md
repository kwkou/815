<properties
    pageTitle="使用 PowerShell 创建 SQL 数据仓库 | Azure"
    description="使用 PowerShell 创建 SQL 数据仓库"
    services="sql-data-warehouse"
    documentationCenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="04/24/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f54d8f576cd4ca14ab36b274e54fde4bb6c56b80"
    ms.lasthandoff="04/14/2017" />

# <a name="create-sql-data-warehouse-using-powershell"></a>使用 PowerShell 创建 SQL 数据仓库
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-data-warehouse-get-started-provision/)
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文说明如何使用 PowerShell 创建 SQL 数据仓库。

## <a name="prerequisites"></a>先决条件
若要开始，您需要：

* **Azure 帐户**：访问 [Azure 免费试用版][Azure Free Trial] ，以创建帐户。
* **Azure SQL Server**：有关详细信息，请参阅[在 Azure 门户中创建 Azure SQL 数据库][Create an Azure SQL database in the Azure Portal]或[使用 PowerShell 创建 Azure SQL 数据库][Create an Azure SQL database with PowerShell]。
* **资源组**：可使用同一资源组作为 Azure SQL Server，或参阅[如何创建资源组](/documentation/articles/resource-group-portal/)。
* **PowerShell 1.0.3 或更高版本**：可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。  可通过 [Microsoft Web 平台安装程序][Microsoft Web Platform Installer]安装最新版本。  有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][How to install and configure Azure PowerShell]。
<!-- [MSDN Azure Credits] not supported in ACN--> 

> [AZURE.NOTE]
> 创建 SQL 数据仓库可能会导致新的计费服务。  有关定价的详细信息，请参阅 [SQL 数据仓库定价][SQL Data Warehouse pricing]。

## <a name="create-a-sql-data-warehouse"></a>创建 SQL 数据仓库
1. 打开 Windows PowerShell。
2. 运行此 cmdlet 以登录到 Azure Resource Manager 中。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

3. 选择要用于当前会话的订阅。

        Get-AzureRmSubscription    -SubscriptionName "MySubscription" | Select-AzureRmSubscription

4. 创建数据库。 此示例将在名为“mywesteuroperesgp1”的资源组中的名为“sqldwserver1”的服务器中创建一个名为“mynewsqldw”且服务目标级别为“DW400”的数据库。

        New-AzureRmSqlDatabase -RequestedServiceObjectiveName "DW400" -DatabaseName "mynewsqldw" -ServerName "sqldwserver1" -ResourceGroupName "mywesteuroperesgp1" -Edition "DataWarehouse" -CollationName "SQL_Latin1_General_CP1_CI_AS" -MaxSizeBytes 10995116277760

所需的参数有：

* **RequestedServiceObjectiveName**：请求的 [DWU][DWU] 的数量。  支持的值包括：DW100、DW200、DW300、DW400、DW500、DW600、DW1000、DW1200、DW1500、DW2000、DW3000 和 DW6000。
* **DatabaseName**：要创建的 SQL 数据仓库的名称。
* **ServerName**：用于创建过程的服务器名称（必须是 V12）。
* **ResourceGroupName**：要使用的资源组。  若要查找订阅中可用的资源，请使用 Get-AzureResource。
* **Edition**：必须为“DataWarehouse”才能创建 SQL 数据仓库。

可选参数有：

* **CollationName**：在不指定的情况下，默认排序规则是 SQL_Latin1_General_CP1_CI_AS。  在数据库上不能更改排序规则。
* **MaxSizeBytes**：数据库的默认最大大小为 10 GB。

有关参数选项的更多详细信息，请参阅 [New-AzureRmSqlDatabase][New-AzureRmSqlDatabase] 和[创建数据库（Azure SQL 数据仓库）][Create Database (Azure SQL Data Warehouse)]。

## <a name="next-steps"></a>后续步骤
完成 SQL 数据仓库预配之后，可能需要尝试[加载示例数据][loading sample data]或了解如何[开发][develop]、[加载][load]或[迁移][migrate]数据。

如果有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看我们有关如何使用 [PowerShell cmdlet 和 REST API][PowerShell cmdlets and REST APIs] 的文章。

<!--Image references-->

<!--Article references-->
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is/
[migrate]: /documentation/articles/sql-data-warehouse-overview-migrate/
[develop]: /documentation/articles/sql-data-warehouse-overview-develop/
[load]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[loading sample data]: /documentation/articles/sql-data-warehouse-load-sample-databases/
[PowerShell cmdlets and REST APIs]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/

[How to install and configure Azure PowerShell]: http://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[how to create a SQL Data Warehouse from the Azure Portal]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Create an Azure SQL database in the Azure Portal]: /documentation/articles/sql-database-get-started/
[Create an Azure SQL database with PowerShell]: /documentation/articles/sql-database-get-started-powershell/
[how to create a resource group]: /documentation/articles/resource-group-template-deploy-portal/#create-resource-group

<!--MSDN references--> 
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[Create Database (Azure SQL Data Warehouse)]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx

<!--Other Web references-->
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps
[SQL Data Warehouse pricing]: /pricing/details/sql-data-warehouse/
[Azure Free Trial]: /pricing/free-trial/?WT.mc_id=A261C142F
[MSDN Azure Credits]: /pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F

<!--Update_Description:update meta properties;wording update-->