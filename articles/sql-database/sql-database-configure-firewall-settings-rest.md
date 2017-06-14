<properties
	pageTitle="使用 REST API 配置 Azure SQL 数据库服务器级别防火墙规则 | Azure"
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
	wacn.date="12/26/2016"
	ms.author="sstein"/>


#  使用 REST API 配置 Azure SQL 数据库服务器级防火墙规则


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-firewall-configure/)
- [Azure 门户](/documentation/articles/sql-database-configure-firewall-settings/)
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到数据库服务器，必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内建立连接，可能需要打开一些其他的 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分


## 通过 REST API 管理服务器级别防火墙规则
1. 通过 REST API 管理防火墙规则必须进行身份验证。有关详细信息，请参阅[使用 Azure Resource Manager API 进行授权的开发人员指南](/documentation/articles/resource-manager-api-authentication/)。
2. 可使用 REST API 创建、更新或删除服务器级别规则

	若要创建或更新服务器级别防火墙规则，请使用以下内容执行 PUT 方法：
 
		https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/firewallRules/{rule-name}?api-version={api-version}
	
	请求正文

		{
         "properties": { 
            "startIpAddress": "{start-ip-address}", 
            "endIpAddress": "{end-ip-address}
            }
        } 
 

	若要删除现有服务器级别防火墙规则，请使用以下内容执行 DELETE 方法：
	 
		https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/firewallRules/{rule-name}?api-version={api-version}


##<a name="manage-firewall-rules-using-the-service-management-rest-api"></a> 使用 REST API 管理防火墙规则

* [创建或更新防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/mt445501.aspx)
* [删除防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/mt445502.aspx)
* [获取防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/mt445503.aspx)
* [列出所有防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/mt604478.aspx)
 
## 后续步骤

有关如何使用 Transact-SQL 创建服务器级别和数据库级别防火墙规则的指导文章，请参阅[使用 T-SQL 配置 Azure SQL 数据库服务器级别和数据库级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/)。

有关如何使用其他方式创建服务器级别防火墙规则的指导文章，请参阅：

- [使用 Azure 门户配置 Azure SQL 数据库服务器级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)
- [使用 PowerShell 配置 Azure SQL 数据库服务器级别防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/)

有关数据库创建教程，请参阅[使用 Azure 门户在几分钟内创建 SQL 数据库](/documentation/articles/sql-database-get-started/)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助信息，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](/documentation/articles/sql-database-manage-logins/)。


## 其他资源

- [保护数据库的安全](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

 

<!---HONumber=Mooncake_Quality_Review_1215_2016-->