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
   ms.date="03/30/2016"
   wacn.date="05/16/2016"/>

# 使用 PowerShell 创建 SQL 数据仓库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell)

## 获取和运行 Azure PowerShell cmdlet

> [AZURE.NOTE]  若要将 Azure Powershell 用于 SQL 数据仓库，应下载并安装带有 ARM cmdlet 的最新版本 Azure PowerShell。可以通过运行 `Get-Module -ListAvailable -Name Azure` 查看你的版本。本文基于  Azure PowerShell 版本 1.0.3 或更高版本。

如果你尚未安装 PowerShell，则需要下载它并对其进行配置。

1. 若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://aka.ms/webpi-azps)。有关此安装程序的详细信息，请参阅[如何安装和配置 Azure PowerShell][]。
2. 若要运行该模块，请在开始窗口中键入 **Windows PowerShell**。
3. 运行此 cmdlet 以登录到 Azure Resource Manager 中。

	```Powershell
	Login-AzureRmAccount–EnvironmentName AzureChinaCloud
	```

4. 选择要用于当前会话的订阅。

	```
	Get-AzureRmSubscription	-SubscriptionName "MySubscription" | Select-AzureRmSubscription
	```

## 创建 SQL 数据仓库数据库
若要部署 SQL 数据仓库，可使用 New-AzureRmSQLDatabase cmdlet。在运行该命令之前，请确保具备以下先决条件。

### 先决条件

- 用于托管数据库的 V12 Azure SQL Server
- 知道 SQL Server 的资源组名称。

### 部署命令

此命令将在 SQL 数据仓库中部署新数据库。

```
New-AzureRmSqlDatabase -RequestedServiceObjectiveName "<Service Objective>" -DatabaseName "<Data Warehouse Name>" -ServerName "<Server Name>" -ResourceGroupName "<ResourceGroupName>" -Edition "DataWarehouse"
```

此示例将一个名为“mynewsqldw1”且服务目标级别为“DW400”的新数据库部署到名为“mywesteuroperesgp1”的资源组中的名为“sqldwserver1”的服务器。

```
New-AzureRmSqlDatabase -RequestedServiceObjectiveName "DW400" -DatabaseName "mynewsqldw1" -ServerName "sqldwserver1" -ResourceGroupName "mywesteuroperesgp1" -Edition "DataWarehouse"
```

此 cmdlet 的必需参数如下：

 + **RequestedServiceObjectiveName**：请求的 DWU 数量，在“DWXXX”格式中，当前支持的值为：100、200、300、400、500、600、1000、1200、1500、2000。
 + **DatabaseName**：要创建的 SQL 数据仓库的名称。
 + **ServerName**：用于创建过程的服务器名称（必须是 V12）。
 + **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
 + **Edition**：必须将版本设置为“DataWarehouse”才能创建 SQL 数据仓库。

有关命令参考，请参阅 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/mt619339.aspx)

有关参数选项，请参阅[创建数据库（Azure SQL 数据仓库）](https://msdn.microsoft.com/zh-cn/library/mt204021.aspx)。

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][]或了解如何[开发][]、[加载][]，或[迁移][]数据。

如果你有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看我们的 [Powershell][] 或 [REST API][] 文档。



<!--Image references-->

<!--Article references-->
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-load-with-bcp/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[Powershell]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure

<!---HONumber=Mooncake_0509_2016-->