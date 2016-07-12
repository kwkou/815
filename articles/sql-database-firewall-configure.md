<properties
   pageTitle="配置 SQL 数据库防火墙 | Azure"
   description="了解如何配置使用服务器级和数据库级防火墙规则的 SQL 数据库防火墙以管理访问权限。"
   keywords="数据库防火墙"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor="cgronlun"
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="02/18/2016"
   wacn.date="06/14/2016"/>

# 如何配置 Azure SQL 数据库防火墙

Azure SQL 数据库为 Azure 和其他基于 Internet 的应用程序提供关系数据库服务。为了帮助保护你的数据，在你指定哪些计算机具有访问权限之前，SQL 数据库防火墙将禁止所有对 SQL 数据库服务器的访问。防火墙基于每个请求的起始 IP 地址授予数据库的访问权限。

若要配置你的数据库防火墙，请创建防火墙规则，以指定可接受的 IP 地址的范围。可以在服务器和数据库级别上创建防火墙规则。

- **服务器级防火墙规则**：这些规则允许客户端访问整个 Azure SQL 数据库服务器，即同一逻辑服务器内的所有数据库。这些规则存储在 **master** 数据库中。
- **数据库级防火墙规则**：这些规则允许客户端访问 Azure SQL 数据库服务器内的单个数据库。按每个数据库创建这些规则，并且存储在单个数据库（包括 **master**）中。这些规则可以用来将访问限制为同一逻辑服务器内的某些（安全）数据库。

**建议：**Azure 建议尽量使用数据库级防火墙规则，以提高数据库的可移植性。如果你的多个数据库具有相同的访问要求，并且你不想花时间分别设置每个数据库，则使用服务器级防火墙规则。

**关于联合：**联合的当前实现将随 Web 和企业服务层一起停用。请考虑部署自定义分片解决方案，以最大限度地提高可缩放性、灵活性和性能。有关自定义分片的详细信息，请参阅[向外扩展 Azure SQL 数据库](https://msdn.microsoft.com/zh-cn/library/dn495641.aspx)。

> [AZURE.NOTE] 如果在 Azure SQL 数据库中创建一个数据库联合（其中根数据库包含数据库级防火墙规则），则不将这些规则复制到联合成员数据库。如果需要为联合成员创建数据库级防火墙规则，你必须重新创建联合成员的规则。但是，如果你使用 ALTER FEDERATION … SPLIT 语句将包含数据库级防火墙规则的联合成员拆分为新的联合成员，新目标成员将具有与源联合成员相同的数据库级防火墙规则。有关联合的详细信息，请参阅 [Azure SQL 数据库中的联合](https://msdn.microsoft.com/zh-cn/library/hh597452.aspx)。

## SQL 数据库防火墙概述

最初，防火墙会阻止对 Azure SQL 数据库服务器的所有访问。为了开始使用你的 Azure SQL 数据库服务器，你必须转到 Azure 门户并且指定使你可以访问 Azure SQL 数据库服务器的一个或多个服务器级防火墙规则。使用防火墙规则可以指定允许的 Internet 上的 IP 地址范围，以及 Azure 应用程序是否可以尝试连接到你的 Azure SQL 数据库服务器。

但是，如果要有选择地授予对 Azure SQL 数据库服务器中某个数据库的访问权限，必须使用一个 IP 地址范围（它超过服务器级防火墙规则中指定的 IP 地址范围）为所需的数据库创建数据库级规则，并确保客户端的 IP 地址位于数据库级规则中指定的范围内。

来自 Internet 和 Azure 的连接尝试必须首先穿过防火墙，然后才能访问你的 Azure SQL 数据库服务器或数据库，如下图中所示。

   ![描绘防火墙配置的示意图。][1]

## 从 Internet 连接

在计算机尝试从 Internet 连接到你的数据库服务器时，防火墙将根据整个服务器级和（如果必需）数据库级防火墙规则检查该请求的发起 IP 地址。

- 如果该请求的 IP 地址位于服务器级防火墙规则中指定的某个范围内，则将连接权限授予 Azure SQL 数据库服务器。

- 如果该请求的 IP 地址不位于服务器级防火墙规则中指定的某个范围内，则检查数据库级防火墙规则。如果该请求的 IP 地址位于数据库级防火墙规则中指定的某个范围内，则仅将连接权限授予匹配数据库级规则的数据库。

- 如果该请求的 IP 地址不位于任何服务器级或数据库级防火墙规则中指定的范围内，则连接请求失败。

> [AZURE.NOTE] 若要从你的本地计算机访问 Azure SQL 数据库，请确保你的网络和本地计算机上的防火墙允许在 TCP 端口 1433 上的传出通信。


## 从 Azure 连接

在应用程序尝试从 Azure 连接到你的数据库服务器时，防火墙将验证是否允许 Azure 连接。如果防火墙设置的开始地址和结束地址都等于 0.0.0.0，则表示允许这些连接。如果不允许该连接尝试，则该请求将不会访问 Azure SQL 数据库服务器。

你可以从 Azure 启用连接：

- 在“[管理门户](https://manage.windowsazure.cn)”中，从服务器上的“配置”选项卡，在“允许的服务”部分下，单击“Azure 服务”对应的“是”。

## 创建第一个服务器级防火墙规则

第一个服务器级防火墙设置可以使用[ Azure 门户](https://manage.windowsazure.cn)进行创建，也可以使用 REST API 或 Azure PowerShell 通过编程方式创建。后续的服务器级防火墙规则可以使用这些方法或通过Transact-SQL来创建和管理。

## 创建数据库级防火墙规则

在你配置第一个服务器级防火墙之后，你可能希望限制对特定数据库的访问。如果你在数据库级防火墙规则中指定的 IP 地址范围超出了在服务器级防火墙规则中指定的范围，则客户端若要访问数据库，其 IP 地址必须处于数据库级别范围内。对于每个数据库，最多可以有 128 个数据库级防火墙规则。可以通过 Transact-SQL 创建和管理主数据库和用户数据库的数据库级防火墙规则。

## 以编程方式管理防火墙规则

除了 Azure 门户外，防火墙规则还可使用 Transact-SQL、REST API 和 Azure PowerShell 通过编程方式进行管理。下表描述了每个方法允许使用的一组命令。


### Transact-SQL

| 目录视图或存储过程 | 级别 | 说明 |
|--------------------------------------------------------------------------------------------|-----------|------------------------------------------------------|
| [sys.firewall\_rules](https://msdn.microsoft.com/zh-cn/library/dn269980.aspx) | 服务器 | 显示当前的服务器级防火墙规则 |
| [sp\_set\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270017.aspx) | 服务器 | 创建或更新服务器级防火墙规则 |
| [sp\_delete\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270024.aspx) | 服务器 | 删除服务器级防火墙规则 |
| [sys.database\_firewall\_rules](https://msdn.microsoft.com/zh-cn/library/dn269982.aspx) | 数据库 | 显示当前的数据库级防火墙规则 |
| [sp\_set\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx) | 数据库 | 创建或更新数据库级防火墙规则 |
| [sp\_delete\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270030.aspx) | 数据库 | 删除数据库级防火墙规则 |

### REST API

| API | 级别 | 说明 |
|----------------------|--------|------------------------------------------------------------------|
| [列出防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505715.aspx) | 服务器 | 显示当前的服务器级防火墙规则 |
| [创建防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505712.aspx) | 服务器 | 创建或更新服务器级防火墙规则 |
| [设置防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505707.aspx) | 服务器 | 更新现有服务器级防火墙规则的属性 |
| [删除防火墙规则](https://msdn.microsoft.com/zh-cn/library/azure/dn505706.aspx) | 服务器 | 删除服务器级防火墙规则 |


### Azure PowerShell

| Cmdlet | 级别 | 说明 |
|-----------------------------------------------------------------------------------------------------|--------|------------------------------------------------------------------|
| [Get-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546731.aspx) | 服务器 | 返回当前的服务器级防火墙规则 |
| [New-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546724.aspx) | 服务器 | 创建新的服务器级防火墙规则 |
| [Set-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546739.aspx) | 服务器 | 更新现有服务器级防火墙规则的属性 |
| [Remove-AzureSqlDatabaseServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/dn546727.aspx) | 服务器 | 删除服务器级防火墙规则 |

> [AZURE.NOTE] 在防火墙设置的更改生效之前，可能最多有五分钟的延迟。

## 数据库防火墙故障排除

在对 Azure SQL 数据库服务的访问与你的期望不符时，请考虑以下几点：

- **本地防火墙配置：**在你的计算机可以访问 Azure SQL 数据库之前，可能需要在你的计算机上创建针对 TCP 端口 1433 的防火墙例外。如果要在 Azure 云边界内部建立连接，可能需要打开其他端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)中的 **SQL 数据库 V12：内部与外部**部分。

- **网络地址转换 (NAT)：**由于 NAT 的原因，计算机用来连接到 Azure SQL 数据库的 IP 地址可能不同于计算机 IP 配置设置中显示的 IP 地址。若要查看你的计算机用于连接到 Azure 的 IP 地址，请登录门户并导航到承载你的数据库的服务器上的“配置”选项卡。在“允许的 IP 地址”部分下，显示了“当前客户端 IP 地址”。单击“添加到允许的 IP 地址”以允许此计算机访问服务器。

- **对允许列表的更改尚未生效：**对 Azure SQL 数据库防火墙配置所做的更改可能最多需要 5 分钟的延迟即可生效。

- **登录名未授权或使用了错误的密码：**如果某个登录名对 Azure SQL 数据库服务器没有权限或者使用的密码不正确，则与 Azure SQL 数据库服务器的连接将被拒绝。创建防火墙设置仅向客户端提供尝试连接到你的服务器的机会；每个客户端必须提供必需的安全凭据。有关准备登录名的详细信息，请参阅在 Azure SQL 数据库中管理数据库、登录名和用户。

- **动态 IP 地址：**如果你的 Internet 连接使用动态 IP 寻址，并且你在通过防火墙时遇到问题，则可以尝试以下解决方法之一：

 - 向你的 Internet 服务提供商 (ISP) 询问分配给你的客户端计算机的将用来访问 Azure SQL 数据库服务器的 IP 地址范围，然后将该 IP 地址范围作为防火墙规则添加。

 - 改为获取你的客户端计算机的静态 IP 地址，然后将该 IP 地址作为防火墙规则添加。

## 另请参阅

[SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!--Image references-->
[1]: ./media/sql-database-firewall-configure/sqldb-firewall-1.png

<!---HONumber=Mooncake_0606_2016-->
