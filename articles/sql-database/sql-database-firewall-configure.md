<properties
    pageTitle="SQL 数据库防火墙规则概述 | Azure"
    description="了解如何配置具有服务器级和数据库级防火墙规则的 SQL 数据库防火墙，以管理访问权限。"
    keywords="数据库防火墙"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor="cgronlun"
    tags="" />
<tags
    ms.assetid="ac57f84c-35c3-4975-9903-241c8059011e"
    ms.service="sql-database"
    ms.custom="authentication and authorization"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="02/09/2017"
    wacn.date="03/24/2017"
    ms.author="rickbyh" />  


# Azure SQL 数据库防火墙规则概述 

Azure SQL 数据库为 Azure 和其他基于 Internet 的应用程序提供关系数据库服务。为了保护你的数据，在你指定哪些计算机具有访问权限之前，防火墙将禁止所有对数据库服务器的访问。防火墙基于每个请求的起始 IP 地址授予数据库访问权限。

若要配置你的防火墙，请创建防火墙规则，以指定可接受的 IP 地址的范围。可以在服务器和数据库级上创建防火墙规则。

* **服务器级防火墙规则**：这些规则允许客户端访问整个 Azure SQL Server，即同一逻辑服务器内的所有数据库。这些规则存储在 **master** 数据库中。可以通过使用门户或 Transact-SQL 语句配置服务器级防火墙规则。若要使用 Azure 门户或 PowerShell 创建服务器级防火墙规则，你必须是订阅所有者或订阅参与者。若要使用 Transact-SQL 创建服务器级防火墙规则，你必须以服务器级主体登录用户或 Azure Active Directory 管理员身份连接到 SQL 数据库实例（这意味着，必须先由具有 Azure 级权限的用户创建服务器级防火墙规则）。
* **数据库级防火墙规则**：这些规则允许客户端访问 Azure SQL 数据库服务器内的单独数据库。可以为每个数据库创建这些规则，它们将存储在单独的数据库中。（可为 **master** 数据库创建数据库级防火墙规则。） 这些规则可以用来将访问限制为同一逻辑服务器内的某些（安全）数据库。只能通过使用 Transact-SQL 语句配置数据库级防火墙规则。

   > [AZURE.NOTE]
   >如需演示如何使用数据库级防火墙的教程，请参阅 [SQL 身份验证和授权](/documentation/articles/sql-database-control-access-sql-authentication-get-started/)。
   >

**建议：**Azure 建议尽量使用数据库级防火墙规则，以增强安全性并提高数据库的可移植性。如果你的多个数据库具有相同的访问要求，并且你不想花时间分别配置每个数据库，则使用管理员的服务器级防火墙规则。

> [AZURE.NOTE]
有关业务连续性上下文中的可移植数据库的信息，请参阅[灾难恢复的身份验证要求](/documentation/articles/sql-database-geo-replication-security-config/)。
>

## 防火墙概述
最初，防火墙会阻止对 Azure SQL Server 的所有 Transact-SQL 访问。若要开始使用 Azure SQL Server，必须转到 Azure 门户并指定允许访问 Azure SQL Server 的一个或多个服务器级防火墙规则。使用防火墙规则可以指定允许的 Internet 上的 IP 地址范围，以及 Azure 应用程序是否可以尝试连接到 Azure SQL Server。

若要有选择地授予对 Azure SQL Server 中的一个数据库的访问权限，必须针对所需的数据库创建数据库级规则。针对超出服务器级防火墙规则中指定的 IP 地址范围的数据库防火墙规则指定一个 IP 地址范围，并确保客户端的 IP 地址位于数据库级规则中指定的范围内。

来自 Internet 和 Azure 的连接尝试必须首先通过防火墙，然后才能访问 Azure SQL Server 或 SQL 数据库，如下图中所示：

   ![描述防火墙配置的示意图。][1]  


## 从 Internet 连接

在计算机尝试从 Internet 连接到数据库服务器时，防火墙将针对连接所请求的数据库，根据数据库级防火墙规则先检查该请求的发起 IP 地址：

* 如果该请求的 IP 地址位于数据库级防火墙规则中指定的某个范围内，则允许连接到包含该规则的 SQL 数据库。
* 如果该请求的 IP 地址不位于数据库级防火墙规则中指定的某个范围内，则检查服务器级防火墙规则。如果该请求的 IP 地址位于服务器级防火墙规则中指定的某个范围内，则允许进行连接。服务器级防火墙规则适用于 Azure SQL Server 上的所有 SQL 数据库。
* 如果该请求的 IP 地址不位于任何数据库级或服务器级防火墙规则中指定的范围内，则连接请求失败。

> [AZURE.NOTE] 若要从你的本地计算机访问 Azure SQL 数据库，请确保你的网络和本地计算机上的防火墙允许在 TCP 端口 1433 上的传出通信。


## 从 Azure 连接
若要允许来自 Azure 的应用程序连接到 Azure SQL Server，则必须启用 Azure 连接。在应用程序尝试从 Azure 连接到你的数据库服务器时，防火墙将验证是否允许 Azure 连接。如果防火墙设置的开始地址和结束地址都等于 0.0.0.0，则表示允许这些连接。如果不允许该连接尝试，则该请求将不会访问 Azure SQL 数据库服务器。

> [AZURE.IMPORTANT]
该选项将防火墙配置为允许来自 Azure 的所有连接，包括来自其他客户的订阅的连接。选择该选项时，请确保你的登录名和用户权限将访问权限限制为仅已授权用户使用。
> 

> [AZURE.NOTE]
有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库：外部与内部**部分
>  

## 创建第一个服务器级防火墙规则

第一个服务器级防火墙设置可以使用 [Azure 门户预览](https://portal.azure.cn/)进行创建，也可以使用 REST API 或 Azure PowerShell 通过编程方式创建。后续的服务器级防火墙规则可以使用这些方法和通过 Transact-SQL 创建和管理。为了提高性能，会暂时在数据库级别缓存服务器级防火墙规则。若要刷新缓存，请参阅 [DBCC FLUSHAUTHCACHE](https://msdn.microsoft.com/zh-cn/library/mt627793.aspx)。有关服务器级防火墙规则的详细信息，请参阅[如何：使用 Azure 门户配置 Azure SQL Server 防火墙](/documentation/articles/sql-database-configure-firewall-settings/)。

##<a name="creating-database-level-firewall-rules"></a> 创建数据库级防火墙规则

在你配置第一个服务器级防火墙之后，你可能希望限制对特定数据库的访问。如果你在数据库级防火墙规则中指定的 IP 地址范围超出了在服务器级防火墙规则中指定的范围，则客户端若要访问数据库，其 IP 地址必须处于数据库级范围内。对于每个数据库，最多可以有 128 个数据库级防火墙规则。可以通过 Transact-SQL 创建和管理 master 数据库和用户数据库的数据库级防火墙规则。有关配置数据库级防火墙规则的详细信息，请参阅 [sp\_set\_database\_firewall\_rule（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx)。

## 以编程方式管理防火墙规则
除了 Azure 门户外，防火墙规则还可使用 Transact-SQL、REST API 和 Azure PowerShell 通过编程方式进行管理。下表描述了每个方法允许使用的一组命令。

### Transact-SQL
| 目录视图或存储过程 | 级别 | 说明 |
| --- | --- | --- |
| [sys.firewall\_rules](https://msdn.microsoft.com/zh-cn/library/dn269980.aspx) | 服务器 | 显示当前的服务器级防火墙规则 |
| [sp\_set\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270017.aspx) | 服务器 | 创建或更新服务器级防火墙规则 |
| [sp\_delete\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270024.aspx) | 服务器 | 删除服务器级防火墙规则 |
| [sys.database\_firewall\_rules](https://msdn.microsoft.com/zh-cn/library/dn269982.aspx) | 数据库 | 显示当前的数据库级防火墙规则 |
| [sp\_set\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx) | 数据库 | 创建或更新数据库级防火墙规则 |
| [sp\_delete\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270030.aspx) | 数据库 | 删除数据库级防火墙规则 |

### REST API
| API | 级别 | 说明 |
| --- | --- | --- |
| [列出防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505715.aspx) | 服务器 | 显示当前的服务器级防火墙规则 |
| [创建防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505712.aspx) | 服务器 | 创建或更新服务器级防火墙规则 |
| [设置防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505707.aspx) | 服务器 | 更新现有服务器级防火墙规则的属性 |
| [删除防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505706.aspx) | 服务器 | 删除服务器级防火墙规则 |

### Azure PowerShell
| Cmdlet | 级别 | 说明 |
| --- | --- | --- |
| [Get-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546731.aspx) | 服务器 | 返回当前的服务器级防火墙规则 |
| [New-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546724.aspx) | 服务器 | 新建服务器级防火墙规则 |
| [Set-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546739.aspx) | 服务器 | 更新现有服务器级防火墙规则的属性 |
| [Remove-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546727.aspx) | 服务器 | 删除服务器级防火墙规则 |

> [AZURE.NOTE] 在防火墙设置的更改生效之前，可能最多有五分钟的延迟。

## 数据库防火墙故障排除

在对 Azure SQL 数据库服务的访问与你的期望不符时，请考虑以下几点：

- **本地防火墙配置：**在你的计算机可以访问 Azure SQL 数据库之前，可能需要在你的计算机上创建针对 TCP 端口 1433 的防火墙例外。如果要在 Azure 云边界内部建立连接，可能需要打开其他端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：外部与内部**部分。

- **网络地址转换 (NAT)：**由于 NAT 的原因，计算机用来连接到 Azure SQL 数据库的 IP 地址可能不同于计算机 IP 配置设置中显示的 IP 地址。若要查看计算机用于连接到 Azure 的 IP 地址，请登录门户并导航到托管数据库的服务器上的“配置”选项卡。在“允许的 IP 地址”部分下，显示了“当前客户端 IP 地址”。单击“添加”即可添加到“允许的 IP 地址”，允许此计算机访问服务器。

- **对允许列表的更改尚未生效：**对 Azure SQL 数据库防火墙配置所做的更改可能最多需要 5 分钟的延迟即可生效。

- **登录名未授权或使用了错误的密码：**如果某个登录名对 Azure SQL 数据库服务器没有权限或者使用的密码不正确，则与 Azure SQL 数据库服务器的连接将被拒绝。创建防火墙设置仅向客户端提供尝试连接到你的服务器的机会；每个客户端必须提供必需的安全凭据。有关准备登录名的详细信息，请参阅在 Azure SQL 数据库中管理数据库、登录名和用户。

- **动态 IP 地址：**如果你的 Internet 连接使用动态 IP 地址，并且你在通过防火墙时遇到问题，则可以尝试以下解决方法之一：

 - 向 Internet 服务提供商 (ISP) 询问分配给客户端计算机的将用来访问 Azure SQL 数据库服务器的 IP 地址范围，然后将该 IP 地址范围作为防火墙规则添加。

 - 改为获取你的客户端计算机的静态 IP 地址，然后将该 IP 地址作为防火墙规则添加。

## 后续步骤
有关如何创建服务器级和数据库级防火墙规则的文章，请参阅：

- [使用 Azure 门户预览配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)
- [使用 T-SQL 配置 Azure SQL 数据库服务器级和数据库级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-tsql/)
- [使用 PowerShell 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-powershell/)
- [使用 REST API 配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings-rest/)

有关创建数据库的教程，请参阅[你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助信息，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx)。

## 其他资源
- [保护数据库的安全](/documentation/articles/sql-database-security/)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!--Image references-->

[1]: ./media/sql-database-firewall-configure/sqldb-firewall-1.png

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->