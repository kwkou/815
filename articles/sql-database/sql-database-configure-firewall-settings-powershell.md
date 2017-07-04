<properties
	pageTitle="使用 PowerShell 配置 Azure SQL 数据库服务器级别防火墙规则 | Azure"
	description="了解如何为访问 Azure SQL 数据库的 IP 地址配置防火墙。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="08/09/2016"
	wacn.date="12/26/2016"/>


# 使用 PowerShell 配置 Azure SQL 数据库服务器级别防火墙规则


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-firewall-configure/)
- [Azure 门户](/documentation/articles/sql-database-configure-firewall-settings/)
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到数据库服务器，必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内建立连接，可能需要打开一些其他 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的“SQL 数据库 V12：内部与外部”部分


[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 创建服务器防火墙规则

可使用 Azure PowerShell 创建、更新和删除服务器级别防火墙规则。

若要创建新的服务器级别防火墙规则，请执行 New-AzureRmSqlServerFirewallRule cmdlet。以下示例启用服务器 Contoso 上的某一 IP 地址范围。

    New-AzureRmSqlServerFirewallRule -ResourceGroupName 'resourcegroup1' -ServerName 'Contoso' -FirewallRuleName "ContosoFirewallRule" -StartIpAddress '192.168.1.1' -EndIpAddress '192.168.1.10'		

若要修改现有服务器级别防火墙规则，请执行 Set-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例更改名为 ContosoFirewallRule 的规则的可接受 IP 地址的范围。

    Set-AzureRmSqlServerFirewallRule -ResourceGroupName 'resourcegroup1' –StartIPAddress 192.168.1.4 –EndIPAddress 192.168.1.10 –RuleName 'ContosoFirewallRule' –ServerName 'Contoso'

若要删除现有服务器级别防火墙规则，请执行 Remove-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例删除名为 ContosoFirewallRule 的规则。

    Remove-AzureRmSqlServerFirewallRule –RuleName 'ContosoFirewallRule' –ServerName 'Contoso'


##<a name="manage-firewall-rules-using-powershell"></a> 使用 PowerShell 管理防火墙规则

还可使用 PowerShell 管理防火墙规则。有关详细信息，请参阅以下主题：

* [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603860.aspx)
* [Remove-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603588.aspx)
* [Set-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603789.aspx)
* [Get-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/mt603586.aspx)


## 后续步骤

若要详细了解如何使用 Transact-SQL 创建服务器级别和数据库级别防火墙规则，请参阅[使用 T-SQL 配置 Azure SQL 数据库服务器级别和数据库级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/)。

有关如何使用其他方式创建服务器级别防火墙规则的详细信息，请参阅：

- [使用 Azure 门户配置 Azure SQL 数据库服务器级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)
- [使用 REST API 配置 Azure SQL 数据库服务器级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/)

有关数据库创建教程，请参阅[使用 Azure 门户在几分钟内创建 SQL 数据库](/documentation/articles/sql-database-get-started/)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助信息，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](/documentation/articles/sql-database-manage-logins/)。


## 其他资源

- [保护数据库的安全](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)


<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

<!---HONumber=Mooncake_Quality_Review_1215_2016-->