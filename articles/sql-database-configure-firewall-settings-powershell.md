<properties
	pageTitle="如何：配置 Azure SQL 数据库防火墙 | Azure"
	description="了解如何配置防火墙以允许 IP 地址访问 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="05/09/2016"
	wacn.date="06/14/2016"/>


# 如何：使用 PowerShell 配置 Azure SQL 数据库防火墙


> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到数据库服务器，则必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内部建立连接，可能需要打开其他一些 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分


[AZURE.INCLUDE [启动 PowerShell 会话](../includes/sql-database-powershell.md)]

## 创建服务器防火墙规则

可以使用 Azure PowerShell 创建、更新和删除服务器级别防火墙规则。

若要创建新的服务器级别防火墙规则，请执行 New-AzureRmSqlServerFirewallRule cmdlet。以下示例启用服务器 Contoso 上一系列 IP 地址。
 
    New-AzureRmSqlServerFirewallRule -ResourceGroupName 'resourcegroup1' -ServerName 'Contoso' -FirewallRuleName "ContosoFirewallRule" -StartIpAddress '192.168.1.1' -EndIpAddress '192.168.1.10'		
 
若要修改现有服务器级别防火墙规则，请执行 Set-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例更改名为 ContosoFirewallRule 的规则的可接受 IP 地址的范围。
 
    Set-AzureRmSqlServerFirewallRule -ResourceGroupName 'resourcegroup1' –StartIPAddress 192.168.1.4 –EndIPAddress 192.168.1.10 –RuleName 'ContosoFirewallRule' –ServerName 'Contoso'

若要删除现有服务器级别防火墙规则，请执行 Remove-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例删除名为 ContosoFirewallRule 的规则。

    Remove-AzureRmSqlServerFirewallRule –RuleName 'ContosoFirewallRule' –ServerName 'Contoso'


## 使用 PowerShell 管理防火墙规则

* [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603860.aspx)
* [Remove-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603588.aspx)
* [Set-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603789.aspx)
* [Get-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603586.aspx)
 
## 后续步骤

有关创建数据库的教程，请参阅[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)。
有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅[以编程方式连接到 Azure SQL 数据库的指导原则](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。
若要了解如何导航到数据库，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)。

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

 

<!---HONumber=Mooncake_0530_2016-->
