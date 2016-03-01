<properties
   pageTitle="使用 PowerShell 创建 SQL 数据仓库 | Microsoft Azure"
   description="使用 PowerShell 创建 SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="10/20/2015"
   wacn.date="01/20/2016"/>

# 使用 PowerShell 创建 SQL 数据仓库

> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-data-warehouse-get-started-create-database-tsql)
- [PowerShell](/documentation/articles/sql-data-warehouse-get-started-provision-powershell)

> [AZURE.NOTE]若要将 Microsoft Azure Powershell 与 SQL 数据仓库配合使用，需要 1.0.2 或更高版本。可以在 Powershell 中运行 (Get-Module Azure).Version 来检查你的版本。

## 获取和运行 Azure PowerShell cmdlet
如果尚未安装 PowerShell，你可以：

1. 若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。
2. 若要运行该模块，请在开始窗口中键入 **Microsoft Azure PowerShell**。
3. 如果尚未将你的帐户添加到计算机，请运行以下 cmdlet。（有关详细信息，请参阅[如何安装和配置 Azure PowerShell][]）：

            Add-AzureRmAccount –EnvironmentName AzureChinaCloud

4. 还需要在 ARM 模式下运行 PowerShell。可以通过运行以下命令切换到此模式：

           Select-AzureRmSubscription -SubscriptionId <MySubscriptionID>

## 创建 SQL 数据仓库
确保已在你的帐户中成功安装 Powershell 之后，你可以运行以下命令来部署新的 SQL 数据仓库：

        New-AzureRmSqlDatabase -RequestedServiceObjectiveName "<Service Objective>" -DatabaseName "<Data Warehouse Name>" -ServerName "<Server Name>" -ResourceGroupName "<ResourceGroupName>" -Edition "DataWarehouse"

此 cmdlet 的必需参数如下：

 + **RequestedServiceObjectiveName**：请求的 DWU 数量，在“DWXXX”格式中，当前支持的值为：100、200、300、400、500、600、1000、1200、1500、2000。
 + **DatabaseName**：要创建的 SQL 数据仓库的名称。
 + **ServerName**：用于创建过程的服务器名称（必须是 V12）。
 + **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
 + **Edition**：必须将版本设置为“DataWarehouse”才能创建 SQL 数据仓库。 

## 后续步骤
完成预配 SQL 数据仓库之后，你可以[加载示例数据][]或了解如何[开发][]、[加载][]，或[迁移][]数据。

如果你有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看我们的 [Powershell][] 或 [REST API][] 文档。



<!--Image references-->

<!--Article references-->
[迁移]: /documentation/articles/sql-data-warehouse-overview-migrate/
[开发]: /documentation/articles/sql-data-warehouse-overview-develop/
[加载]: /documentation/articles/sql-data-warehouse-overview-load/
[加载示例数据]: /documentation/articles/sql-data-warehouse-get-started-manually-load-samples/
[Powershell]: /documentation/articles/sql-data-warehouse-reference-powershell-cmdlets/
[REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[firewall rules]: /documentation/articles/sql-database-configure-firewall-settings/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure

<!---HONumber=Mooncake_1207_2015-->
