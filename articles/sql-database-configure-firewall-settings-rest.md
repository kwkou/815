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


# 如何：使用 REST API 配置 Azure SQL 数据库防火墙


> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到数据库服务器，则必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内部建立连接，可能需要打开其他一些 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分


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


## 使用服务管理 REST API 管理防火墙规则

* [创建防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505712.aspx)
* [删除防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505706.aspx)
* [获取防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505698.aspx)
* [列出防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505715.aspx)
 
## 后续步骤

有关创建数据库的教程，请参阅[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)。
有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅[以编程方式连接到 Azure SQL 数据库的指导原则](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。
若要了解如何导航到数据库，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins/)。

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

 

<!---HONumber=Mooncake_0530_2016-->
