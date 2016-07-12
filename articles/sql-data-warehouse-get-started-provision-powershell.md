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
   ms.date="04/20/2016"
   wacn.date="06/06/2016"/>

# 使用 PowerShell 创建 SQL 数据仓库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql/)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)

### 先决条件
在开始之前，请确保符合以下先决条件。

- 用于托管数据库的 V12 Azure SQL Server
- 知道 SQL Server 的资源组名称



> [AZURE.NOTE]  若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0.3 或更高版本。可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。可以通过 [Microsoft Web 平台安装程序][]安装最新版本。有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][]。

## 创建 SQL 数据仓库数据库
1. 打开 Windows PowerShell。
2. 运行此 cmdlet 以登录到 Azure Resource Manager 中。

	```
	Login-AzureRmAccount–EnvironmentName AzureChinaCloud
	```
	
3. 选择要用于当前会话的订阅。

	```
	Get-AzureRmSubscription	-SubscriptionName "MySubscription" | Select-AzureRmSubscription
	```

4.  创建数据库。此示例将在名为“mywesteuroperesgp1”的资源组中的名为“sqldwserver1”的服务器中创建一个名为“mynewsqldw”且服务目标级别为“DW400”的新数据库。**注意：创建新的 SQL 数据仓库数据库可能会导致新的计费费用。有关定价的更多详细信息，请参阅 [SQL 数据仓库定价][]。**

	```
	New-AzureRmSqlDatabase -RequestedServiceObjectiveName "DW400" -DatabaseName "mynewsqldw" -ServerName "sqldwserver1" -ResourceGroupName "mywesteuroperesgp1" -Edition "DataWarehouse"
	```

此 cmdlet 所需的参数：

- **RequestedServiceObjectiveName**：请求的 DWU 数量，采用“DWXXX”格式。DWU 表示 CPU 和内存分配。各 DWU 值表示这些资源的线性增加。目前支持的值为：100、200、300、400、500、600、1000、1200、1500、2000。
- **DatabaseName**：要创建的 SQL 数据仓库的名称。
- **ServerName**：用于创建过程的服务器名称（必须是 V12）。
- **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
- **Edition**：必须将版本设置为“DataWarehouse”才能创建 SQL 数据仓库。

有关参数选项的更多详细信息，请参阅[创建数据库（Azure SQL 数据仓库）][]。
有关命令参考，请参阅 [New-AzureRmSqlDatabase][]

## 后续步骤
完成 SQL 数据仓库预配之后，你可能想要尝试[加载示例数据][]或了解如何[开发][]、[加载][]或[迁移][]数据。

如果有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看我们有关如何使用 [PowerShell cmdlet 和 REST API][] 的文章。

<!--Image references-->

<!--Article references-->
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[load sample data]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[Powershell]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/

<!--MSDN references--> 
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[创建数据库（Azure SQL 数据仓库）]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx

<!--Other Web references-->
[Microsoft Web 平台安装程序]: https://aka.ms/webpi-azps
[SQL 数据仓库定价]: /home/features/sql-data-warehouse/pricing/

<!---HONumber=Mooncake_0530_2016-->