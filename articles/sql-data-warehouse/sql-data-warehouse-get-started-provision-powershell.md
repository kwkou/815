<properties
   pageTitle="使用 PowerShell 创建 SQL 数据仓库 | Azure"
   description="使用 PowerShell 创建 SQL 数据仓库"
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
   ms.date="08/25/2016"
   wacn.date="10/17/2016"/>  


# 使用 PowerShell 创建 SQL 数据仓库

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-data-warehouse-get-started-provision/)
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

本文说明如何使用 PowerShell 创建 SQL 数据仓库。

## 先决条件

若要开始，您需要：

- **Azure 帐户**：访问 [Azure 试用版][]，以创建帐户。
- **Azure SQL Server**：有关详细信息，请参阅[使用 Azure 门户创建 Azure SQL 数据库逻辑服务器][]或[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器][]。
- **资源组**：可使用同一资源组作为 Azure SQL Server，或参阅[如何创建资源组][]。
- **PowerShell 1.0.3 或更高版本**：可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。可以通过 [Microsoft Web 平台安装程序][]安装最新版本。有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][]。

> [AZURE.NOTE] 创建 SQL 数据仓库可能会导致新的计费服务。有关定价的详细信息，请参阅 [SQL 数据仓库定价][]。

## 创建 SQL 数据仓库

1. 打开 Windows PowerShell。
2. 运行此 cmdlet 以登录到 Azure Resource Manager 中。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	
3. 选择要用于当前会话的订阅。


		Get-AzureRmSubscription	-SubscriptionName "MySubscription" | Select-AzureRmSubscription


4.  创建数据库。此示例将在名为“mywesteuroperesgp1”的资源组中的名为“sqldwserver1”的服务器中创建一个名为“mynewsqldw”且服务目标级别为“DW400”的新数据库。**注意：创建新的 SQL 数据仓库数据库可能会导致新的计费费用。有关定价的详细信息，请参阅 [SQL 数据仓库定价][]。**


		New-AzureRmSqlDatabase -RequestedServiceObjectiveName "DW400" -DatabaseName "mynewsqldw" -ServerName "sqldwserver1" -ResourceGroupName "mywesteuroperesgp1" -Edition "DataWarehouse"


所需的参数包括：

- **RequestedServiceObjectiveName**：请求的 [DWU][] 的数量。支持的值包括：DW100、DW200、DW300、DW400、DW500、DW600、DW1000、DW1200、DW1500、DW2000、DW3000 和 DW6000。
- **DatabaseName**：要创建的 SQL 数据仓库的名称。
- **ServerName**：用于创建过程的服务器名称（必须是 V12）。
- **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
- **Edition**：必须为“DataWarehouse”才能创建 SQL 数据仓库。

可选参数包括：

- **CollationName**：在不指定的情况下，默认排序规则是 SQL\_Latin1\_General\_CP1\_CI\_AS。在数据库上不能更改排序规则。
- **MaxSizeBytes**：数据库的默认最大大小为 10 GB。


有关参数选项的更多详细信息，请参阅 [New-AzureRmSqlDatabase][] 和[创建数据库（Azure SQL 数据仓库）][]。

## 后续步骤

完成 SQL 数据仓库预配之后，可能需要尝试[加载示例数据][]或了解如何[开发][]、[加载][]或[迁移][]数据。

如果有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看有关如何使用 [PowerShell cmdlet 和 REST API][] 的文章。

<!--Image references-->


<!--Article references-->
[DWU]: /documentation/articles/sql-data-warehouse-overview-what-is#data-warehouse-units
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[PowerShell cmdlet 和 REST API]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/

[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/
[how to create a SQL Data Warehouse from the Azure Portal]: /documentation/articles/sql-data-warehouse-get-started-provision/
[使用 Azure 门户预览创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started#create-an-azure-sql-database-logical-server/
[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器]: /documentation/articles/sql-database-get-started-powershell#database-setup-create-a-resource-group-server-and-firewall-rule/
[如何创建资源组]: /documentation/articles/resource-group-portal/

<!--MSDN references--> 
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[创建数据库（Azure SQL 数据仓库）]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx

<!--Other Web references-->
[Microsoft Web 平台安装程序]: https://aka.ms/webpi-azps
[SQL 数据仓库定价]: /pricing/details/sql-data-warehouse/
[Azure 试用版]: /pricing/free-trial/?WT.mc_id=A261C142F
[MSDN Azure 信用额度]: /pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F

<!---HONumber=Mooncake_1010_2016-->