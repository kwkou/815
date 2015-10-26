<properties
	pageTitle="如何：配置防火墙设置（Azure SQL 数据库）"
	description="配置防火墙以允许 IP 地址访问 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="09/04/2015"
	wacn.date="10/17/2015"/>


# 如何：在 SQL 数据库上配置防火墙设置


 Windows Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

**重要提示**若要允许来自 Azure 的应用程序连接到数据库服务器，则必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure)。


## 服务器级别防火墙规则

可以通过 Windows Azure 管理门户、Transact-SQL、Azure PowerShell 或 REST API 创建和管理服务器级别防火墙规则。

### 通过新的 Azure 门户管理服务器级别防火墙规则


[AZURE.INCLUDE [sql-database-include-ip-address-22-v12portal](../includes/sql-database-include-ip-address-22-v12portal.md)]


## 通过管理门户管理服务器级别防火墙规则 

1. 在管理门户中，单击“SQL 数据库”。此处列出了所有数据库及其相应的服务器。
2. 单击页面顶部的“服务器”。
3. 单击想要管理其防火墙规则的服务器旁边的箭头。
4. 单击页面顶部的“配置”。

	*  若要添加当前计算机，单击“添加到允许的 IP 地址”。
	*  若要添加其他 IP 地址，请键入“规则名称”、“起始 IP 地址”和“结束 IP 地址”。
	*  若要修改现有规则，单击规则中的任意字段并修改。
	*  若要删除现有规则，将鼠标悬停在该规则上直到 X 出现在行的末尾。单击“X”删除规则。
5. 单击页面底部的“保存”以保存所做的更改。

## 通过 Transact-SQL 管理服务器级别防火墙规则

1. 通过管理门户或通过 SQL Server Management Studio 启动一个查询窗口。
2. 验证你是否已连接到 master 数据库。
3. 可以从查询窗口选择、创建、更新或删除服务器级别防火墙规则。
4. 若要创建或更新服务器级别防火墙规则，执行 sp\_set\_firewall 规则存储过程。以下示例启用服务器 Contoso 上一系列 IP 地址。<br/>首先，查看已经存在哪些规则。

		SELECT * FROM sys.database_firewall_rules ORDER BY name;

	其次，添加防火墙规则。

		EXECUTE sp_set_firewall_rule @name = N'ContosoFirewallRule',
			@start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'

	若要删除服务器级别防火墙规则，请执行 sp\_delete\_firewall\_rule 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。
 
		EXECUTE sp_delete_firewall_rule @name = N'ContosoFirewallRule'
 
## 通过 Azure PowerShell 管理服务器级别防火墙规则
1. 启动 Azure PowerShell。
2. 可以使用 Azure PowerShell 创建、更新和删除服务器级别防火墙规则。 

	若要创建新的服务器级别防火墙规则，请执行 New-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例启用服务器 Contoso 上一系列 IP 地址。
 
		New-AzureSqlDatabaseServerFirewallRule –StartIPAddress 192.168.1.1 –EndIPAddress 192.168.1.10 –RuleName ContosoFirewallRule –ServerName Contoso
 
	若要修改现有服务器级别防火墙规则，请执行 Set-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例更改名为 ContosoFirewallRule 的规则的可接受 IP 地址的范围。
 
		Set-AzureSqlDatabaseServerFirewallRule –StartIPAddress 192.168.1.4 –EndIPAddress 192.168.1.10 –RuleName ContosoFirewallRule –ServerName Contoso

	若要删除现有服务器级别防火墙规则，请执行 Remove-AzureSqlDatabaseServerFirewallRule cmdlet。以下示例删除名为 ContosoFirewallRule 的规则。

		Remove-AzureSqlDatabaseServerFirewallRule –RuleName ContosoFirewallRule –ServerName Contoso
 
## 通过 REST API 管理服务器级别防火墙规则
1. 通过 REST API 管理防火墙规则必须进行身份验证。有关信息，请参阅身份验证服务管理请求。
2. 可使用 REST API 创建、更新或删除服务器级别规则

	若要创建或更新服务器级别防火墙规则，请使用以下内容执行 POST 方法：
 
		https://management.core.chinacloudapi.cn:8443/{subscriptionId}/services/sqlservers/servers/Contoso/firewallrules
	
	请求正文

		<ServiceResource xmlns="http://schemas.microsoft.com/windowsazure">
		  <Name>ContosoFirewallRule</Name>
		  <StartIPAddress>192.168.1.4</StartIPAddress>
		  <EndIPAddress>192.168.1.10</EndIPAddress>
		</ServiceResource>
 

	若要删除现有服务器级别防火墙规则，请使用以下内容执行 DELETE 方法：
	 
		https://management.core.chinacloudapi.cn:8443/{subscriptionId}/services/sqlservers/servers/Contoso/firewallrules/ContosoFirewallRule
 
## 数据库级别防火墙规则

1. 为你的 IP 地址创建服务器级别防火墙后，通过管理门户或通过 SQL Server Management Studio 启动一个查询窗口。
2. 连接到你要为其创建数据库级别防火墙规则的数据库。

	若要创建新的或更新现有的数据库级别防火墙规则，请执行 sp\_set\_database\_firewall\_rule 存储过程。以下示例创建名为 ContosoFirewallRule 的新防火墙规则。
 
		EXEC sp_set_database_firewall_rule @name = N'ContosoFirewallRule', @start_ip_address = '192.168.1.11', @end_ip_address = '192.168.1.11'
 
	若要删除现有的数据库级别防火墙规则，请执行 sp\_delete\_database\_firewall\_rule 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。
 
		EXEC sp_delete_database_firewall_rule @name = N'ContosoFirewallRule'


## 使用服务管理 REST API 管理防火墙规则

* [创建防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505712.aspx)
* [删除防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505706.aspx)
* [获取防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505698.aspx)
* [列出防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505715.aspx)

## 使用 PowerShell 管理防火墙规则

* [New-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546724.aspx)
* [Remove-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546727.aspx)
* [Set-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546739.aspx)
* [Get-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546731.aspx)
 
## 后续步骤

有关创建数据库的教程，请参阅[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)。
有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅[以编程方式连接到 Azure SQL 数据库的指导原则](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。
若要了解如何导航到数据库，请参阅[在 Azure SQL 数据库中管理数据库和登录名](https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)。

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

<!---HONumber=74-->