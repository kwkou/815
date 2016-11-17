<properties
	pageTitle="配置 SQL 数据库服务器级防火墙规则 | Azure"
	description="了解如何配置防火墙以允许 IP 地址访问 Azure SQL 服务器。"
	services="sql-database"
	documentationCenter=""
	authors="BYHAM"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article" 
	ms.date="08/30/2016" 
	wacn.date="11/15/2016"
	ms.author="rickbyh;carlrab"/>  



# 使用 Azure 门户预览配置 Azure SQL 数据库服务器级防火墙规则


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-firewall-configure/)
- [Azure 门户预览](/documentation/articles/sql-database-configure-firewall-settings/)
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)

Azure SQL Server 通过防火墙规则允许连接到服务器和数据库。可在 Azure SQL Server 逻辑服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别的防火墙设置，从而有选择地允许对数据库的访问。本主题讨论服务器级防火墙规则。

> [AZURE.IMPORTANT] 若要允许来自 Azure 的应用程序连接到 Azure SQL Server，则必须启用 Azure 连接。若要了解防火墙规则的工作方式，请参阅[如何配置 Azure SQL 服务器防火墙 - 概述](/documentation/articles/sql-database-firewall-configure/)。如果要在 Azure 云边界内建立连接，可能需要打开一些其他 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分

**建议：**如果你的多个数据库具有相同的访问要求，并且你不想花时间分别设置每个数据库，则使用管理员的服务器级防火墙规则。Azure 建议尽量使用数据库级防火墙规则，以增强安全性并提高数据库的可移植性。

[AZURE.INCLUDE [创建 SQL 数据库数据库](../../includes/sql-database-create-new-server-firewall-portal.md)]

## 通过 Azure 门户预览管理现有的服务器级别防火墙规则

重复以下步骤来管理服务器级别的防火墙规则。

- 若要添加当前计算机，请单击“添加客户端 IP”。
- 若要添加其他 IP 地址，请键入“规则名称”、“起始 IP 地址”和“结束 IP 地址”。
- 若要修改现有规则，单击规则中的任意字段并修改。
- 若要删除现有规则，将鼠标悬停在该规则上直到 X 出现在行的末尾。单击“X”删除规则。

单击“保存”以保存更改。

## 后续步骤

有关如何使用 Transact-SQL 创建服务器级和数据库级防火墙规则的指导文章，请参阅[使用 T-SQL 配置 Azure SQL 数据库服务器级和数据库级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/)。

有关如何使用其他方式创建服务器级防火墙规则的指导文章，请参阅：

- [使用 PowerShell 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [使用 REST API 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/)

有关数据库创建教程，请参阅[使用 Azure 门户预览在几分钟内创建一个 SQL 数据库](/documentation/articles/sql-database-get-started/)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助，请参阅 [SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-libraries/)。若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](/documentation/articles/sql-database-manage-logins/)。


## 其他资源

- [保护你的数据库](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)


<!--Image references-->
[1]: ./media/sql-database-configure-firewall-settings/AzurePortalBrowseForFirewall.png
[2]: ./media/sql-database-configure-firewall-settings/AzurePortalFirewallSettings.png
<!--anchors-->


 

<!---HONumber=Mooncake_1010_2016-->