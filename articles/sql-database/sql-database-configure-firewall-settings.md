<properties
    pageTitle="配置 SQL 数据库服务器级防火墙规则 | Azure"
    description="了解如何配置防火墙以允许 IP 地址访问 Azure SQL 服务器。"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="c3b206b5-af6e-41af-8306-db12ecfc1b5d"
    ms.service="sql-database"
    ms.custom="authentication and authorization"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="get-started-article"
    ms.date="11/28/2016"
    wacn.date="01/20/2017"
    ms.author="rickbyh;carlrab" />  



# 使用 Azure 门户创建和管理 Azure SQL 数据库服务器级防火墙规则


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-firewall-configure/)
- [Azure 门户](/documentation/articles/sql-database-configure-firewall-settings/)
- [TSQL](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [PowerShell](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [REST API](/documentation/articles/sql-database-configure-firewall-settings-rest/)


服务器级防火墙规则可让管理员通过指定的 IP 地址或 IP 地址范围访问 SQL 数据库服务器。如果多个数据库具有相同的访问要求，并且不想要花费时间分别配置每个数据库，则也可以针对用户使用服务器级防火墙规则。Microsoft 建议尽量使用数据库级防火墙规则，以增强安全性并提高数据库的可移植性。有关 SQL 数据库防火墙的概述，请参阅 [SQL 数据库防火墙规则概述](/documentation/articles/sql-database-firewall-configure/)。

> [AZURE.NOTE]
有关业务连续性上下文中的可移植数据库的信息，请参阅[灾难恢复的身份验证要求](/documentation/articles/sql-database-geo-replication-security-config/)。
>

[AZURE.INCLUDE [创建 SQL 数据库防火墙规则](../../includes/sql-database-create-new-server-firewall-portal.md)]

## 通过 Azure 门户管理现有的服务器级别防火墙规则

重复以下步骤来管理服务器级别的防火墙规则。

- 若要添加当前计算机，请单击“添加客户端 IP”。
- 若要添加其他 IP 地址，请键入“规则名称”、“起始 IP 地址”和“结束 IP 地址”。
- 若要修改现有规则，单击规则中的任意字段并修改。
- 若要删除现有规则，将鼠标悬停在该规则上直到“X”出现在行的末尾。单击“X”删除规则。

单击“保存”以保存更改。

## 后续步骤

- 有关入门教程，请参阅 [SQL 数据库教程：创建服务器、服务器级防火墙规则、示例数据库、数据库级防火墙规则并连接到 SQL Server](/documentation/articles/sql-database-get-started/)。
- 有关安全入门教程，请参阅[安全入门](/documentation/articles/sql-database-get-started-security/)
- 有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助信息，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。
- 若要了解如何创建其他可连接到数据库的用户，请参阅 [SQL 数据库身份验证和授权：授予访问权限](https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)。

## 其他资源
- [保护数据库的安全](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!---HONumber=Mooncake_0116_2017-->
<!--update: wording update; update several link references-->