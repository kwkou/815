<properties
   pageTitle="使用 Powershell 创建 SQL 数据仓库 | Azure"
   description="使用 PowerShell 创建 SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="01/11/2016"
   wacn.date="02/26/2016"/>

# 使用 PowerShell 创建 SQL 数据仓库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell)

> [AZURE.NOTE]若要将 Azure Powershell 与 SQL 数据仓库配合使用，需要 1.0.2 或更高版本。可以在 Powershell 中运行 (Get-Module Azure).Version 来检查你的版本。

## 获取和运行 Azure PowerShell cmdlet
如果你尚未安装 PowerShell，则需要下载它并对其进行配置。

1. 若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。
2. 若要运行该模块，请在开始窗口中键入 **Azure PowerShell**。
3. 如果尚未将你的帐户添加到计算机，请运行以下 cmdlet。（有关详细信息，请参阅[如何安装和配置 Azure PowerShell][]）：

```
Add-AzureAccount
```

4. 选择要使用的订阅。此示例将获取订阅名称的列表。然后，它将订阅名称设置为“MySubscription”。 

```
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName "MySubscription"
```
   
## 创建 SQL 数据仓库
为你的帐户配置 PowerShell 后，可以运行以下命令以在 SQL 数据仓库中部署新数据库。

```
New-AzureSqlDatabase -RequestedServiceObjectiveName "<Service Objective>" -DatabaseName "<Data Warehouse Name>" -ServerName "<Server Name>" -ResourceGroupName "<ResourceGroupName>" -Edition "DataWarehouse"
```

此 cmdlet 的必需参数如下：

 + **RequestedServiceObjectiveName**：请求的 DWU 数量，在“DWXXX”格式中，当前支持的值为：100、200、300、400、500、600、1000、1200、1500、2000。
 + **DatabaseName**：要创建的 SQL 数据仓库的名称。
 + **ServerName**：用于创建过程的服务器名称（必须是 V12）。
 + **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
 + **Edition**：必须将版本设置为“DataWarehouse”才能创建 SQL 数据仓库。 

有关命令参考，请参阅 [New-AzureSqlDatabase](https://msdn.microsoft.com/zh-cn/library/mt619339.aspx)

有关参数选项，请参阅[创建数据库（Azure SQL 数据仓库）](https://msdn.microsoft.com/zh-cn/library/mt204021.aspx)。

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][]或了解如何[开发][]、[加载][] 或[迁移][]。

如果你有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看我们的 [Powershell][] 或 [REST API][] 文档。



<!--Image references-->

<!--Article references-->
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[Powershell]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure

<!---HONumber=Mooncake_0215_2016-->