<properties
	pageTitle="如何：配置防火墙设置 | Azure"
	description="了解如何配置防火墙以允许 IP 地址访问 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="03/16/2016"
	wacn.date="04/06/2016"/>


# 如何：使用 Azure 门户在 SQL 数据库上配置防火墙设置


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-configure-firewall-settings)
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest)


Azure SQL Server 通过防火墙规则允许连接到服务器和数据库。可在 Azure SQL Server 逻辑服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别的防火墙设置，从而有选择地允许对数据库的访问。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到 Azure SQL Server，则必须启用 Azure 连接。要了解防火墙规则的运作方式，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure)。如果要在 Azure 云边界内部建立连接，可能需要打开其他一些 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)中的 **SQL 数据库 V12：内部与外部**部分


### 通过 Azure 门户添加服务器级别的防火墙规则


[AZURE.INCLUDE [sql-database-include-ip-address-22-v12portal](../includes/sql-database-include-ip-address-22-v12portal.md)]


## 通过 Azure 门户管理现有的服务器级别防火墙规则

重复以下步骤来管理服务器级别的防火墙规则。

- 若要添加当前计算机，请单击“添加客户端 IP”。
- 若要添加其他 IP 地址，请键入“规则名称”、“起始 IP 地址”和“结束 IP 地址”。
- 若要修改现有规则，单击规则中的任意字段并修改。
- 若要删除现有规则，将鼠标悬停在该规则上直到 X 出现在行的末尾。单击“X”删除规则。

单击“保存”以保存更改。


## 后续步骤

服务器防火墙规则将影响 Azure SQL Server 上的所有 SQL 数据库。若要配置仅影响单一数据库的数据库级别的防火墙规则，请参阅 [sp\_set\_firewall\_rule（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn270017.aspx")。

有关创建数据库的教程，请参阅[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅[以编程方式连接到 Azure SQL 数据库的指导原则](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。若要了解如何导航到数据库，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins)。

<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->

<!---HONumber=Mooncake_0328_2016-->
