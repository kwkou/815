<properties
	pageTitle="如何：配置 SQL 数据库防火墙 | Azure"
	description="了解如何配置防火墙以允许 IP 地址访问 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="04/26/2016"
	wacn.date="06/14/2016"/>


# 如何：使用 TSQL 配置 Azure SQL 数据库防火墙


> [AZURE.SELECTOR]
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到数据库服务器，则必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内部建立连接，可能需要打开其他一些 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分


## 通过 Transact-SQL 管理服务器级别防火墙规则

只有服务器级别主体登录名或 Azure Active Directory 管理员才能使用 Transact-SQL 创建服务器级别防火墙规则。

1. 启动查询窗口，并使用 SQL Server Management Studio 连接到虚拟 master 数据库。
2. 可以从查询窗口选择、创建、更新或删除服务器级别防火墙规则。
3. 若要创建或更新服务器级别防火墙规则，执行 sp\_set\_firewall 规则存储过程。以下示例启用服务器 Contoso 上一系列 IP 地址。<br/>首先，查看已经存在哪些规则。

		SELECT * FROM sys.firewall_rules ORDER BY name;

	其次，添加防火墙规则。

		EXECUTE sp_set_firewall_rule @name = N'ContosoFirewallRule',
			@start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'

	若要删除服务器级别防火墙规则，请执行 sp\_delete\_firewall\_rule 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。
 
		EXECUTE sp_delete_firewall_rule @name = N'ContosoFirewallRule'
 
 有关这些存储过程的详细信息，请参阅 [sp\_set\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270017.aspx) 和 [sp\_delete\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270024.aspx)。

## 数据库级别防火墙规则

只有对数据库具备**控制**权限的数据库用户（例如数据库所有者）才能创建数据库级别防火墙规则。

1. 为你的 IP 地址创建服务器级别防火墙后，通过管理门户或通过 SQL Server Management Studio 启动一个查询窗口。
2. 连接到你要为其创建数据库级别防火墙规则的数据库。

	若要创建新的或更新现有的数据库级别防火墙规则，请执行 sp\_set\_database\_firewall\_rule 存储过程。以下示例创建名为 ContosoFirewallRule 的新防火墙规则。
 
		EXEC sp_set_database_firewall_rule @name = N'ContosoFirewallRule', 
		    @start_ip_address = '192.168.1.11', @end_ip_address = '192.168.1.11'
 
	若要删除现有的数据库级别防火墙规则，请执行 sp\_delete\_database\_firewall\_rule 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。
 
		EXEC sp_delete_database_firewall_rule @name = N'ContosoFirewallRule'

有关这些存储过程的详细信息，请参阅 [sp\_set\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx) 和 [sp\_delete\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270030.aspx)。

## 后续步骤

有关创建数据库的教程，请参阅[使用 Azure 管理门户在几分钟内创建一个 SQL 数据库](/documentation/articles/sql-database-get-started/)。
有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。
若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](/documentation/articles/sql-database-manage-logins/)。


<!---HONumber=Mooncake_0530_2016-->
