<properties
    pageTitle="Azure SQL 数据库访问控制 | Azure"
    description="了解如何控制 Azure SQL 数据库的访问权限。"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor=""
    tags="" />
<tags
    ms.assetid="8e71b04c-bc38-4153-8f83-f2b14faa31d9"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="10/18/2016"
    wacn.date="01/25/2017"
    ms.author="rickbyh" />  


# Azure SQL 数据库访问控制
为了确保安全性，SQL 数据库会进行访问控制，即：使用防火墙规则来限制通过 IP 地址进行的连接，使用身份验证机制来要求用户证明其身份，并使用授权机制来限制用户执行特定操作和访问特定数据。

> [AZURE.IMPORTANT]
有关 SQL 数据库安全功能的概述，请参阅 [SQL 安全概述](/documentation/articles/sql-database-security-overview/)。有关使用 SQL Server 身份验证的教程，请参阅 [SQL 数据库教程：SQL Server 身份验证、登录名和用户帐户、数据库角色、权限、服务器级防火墙规则和数据库级防火墙规则](/documentation/articles/sql-database-control-access-sql-authentication-get-started/)。有关使用 Azure Active Directory 身份验证的教程，请参阅 [SQL 数据库教程：AAD 身份验证、登录名和用户帐户、数据库角色、权限、服务器级防火墙规则和数据库级防火墙规则](/documentation/articles/sql-database-control-access-aad-authentication-get-started/)。

>

## 防火墙和防火墙规则
Azure SQL 数据库为 Azure 和其他基于 Internet 的应用程序提供关系数据库服务。为了保护你的数据，在你指定哪些计算机具有访问权限之前，防火墙将禁止所有对数据库服务器的访问。防火墙基于每个请求的起始 IP 地址授予数据库访问权限。有关详细信息，请参阅 [Azure SQL 数据库防火墙规则概述](/documentation/articles/sql-database-firewall-configure/)。

只能通过 TCP 端口 1433 使用 Azure SQL 数据库服务。若要从计算机访问 SQL 数据库，请确保客户端计算机防火墙允许 TCP 端口 1433 上的传出 TCP 通信。如果其他应用程序不需要，则阻止 TCP 端口 1433 上的入站连接。

在连接过程中，来自 Azure 虚拟机的连接将重定向到每个辅助角色特有的不同 IP 地址和端口。该端口号在 11000 到 11999 的范围内。有关 TCP 端口的详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)。

## 身份验证

SQL 数据库支持两种类型的身份验证：

* **SQL 身份验证**，使用用户名和密码。在为数据库创建逻辑服务器时，你已指定一个包含用户名和密码的“服务器管理员”登录名。通过这些凭据，你可以使用数据库所有者（即“dbo”）的身份通过服务器上任何数据库的身份验证。
* **Azure Active Directory 身份验证**，使用 Azure Active Directory 管理的标识，并支持托管域和集成域。[尽可能](https://msdn.microsoft.com/zh-cn/library/ms144284.aspx)使用 Active Directory 身份验证（集成安全性）。如果想要使用 Azure Active Directory 身份验证，则必须创建名为“Azure AD 管理员”的另一个服务器管理员，用于管理 Azure AD 用户和组。此管理员还能执行普通服务器管理员可以执行的所有操作。有关如何创建 Azure AD 管理员以启用 Azure Active Directory 身份验证的演练，请参阅[通过使用 Azure Active Directory 身份验证连接到 SQL 数据库](/documentation/articles/sql-database-aad-authentication/)。

数据库引擎将关闭空闲超过 30 分钟的连接。该连接必须重新登录才可供使用。连续与 SQL 数据库建立活动连接需要至少每隔 10 小时重新授权一次（由数据库引擎执行授权）。数据库引擎将尝试使用最初提交的密码重新授权，且不需要用户输入。出于性能原因，在 SQL 数据库中重置密码时，将不会对连接重新进行身份验证，即使该连接由于连接池而重置。这一点与本地 SQL Server 的行为不同。如果在最初授权连接后密码发生更改，则必须终止该连接，并使用新密码建立新连接。具有 KILL DATABASE CONNECTION 权限的用户可以使用 [KILL](https://msdn.microsoft.com/zh-cn/library/ms173730.aspx) 命令显式终止与 SQL 数据库的连接。

用户帐户可以在 master 数据库中创建，可以在服务器的所有数据库中授予权限，也可在数据库中自行创建（称为“包含用户”）。有关创建和管理登录名的信息，请参阅[管理登录名](/documentation/articles/sql-database-manage-logins/)。若要提高可移植性和可伸缩性，请使用包含数据库用户。有关包含用户的详细信息，请参阅[包含数据库用户 - 使数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)、[CREATE USER (Transact-SQL)](https://technet.microsoft.com/zh-cn/library/ms173463.aspx) 和[包含数据库](https://technet.microsoft.com/zh-cn/library/ff929071.aspx)。

作为最佳实践，应用程序应使用不同的帐户进行身份验证 – 这样，就可以限制授予应用程序的权限，并在应用程序代码容易受到 SQL 注入攻击的情况下降低恶意活动的风险。建议的方法是创建[包含数据库用户](https://msdn.microsoft.com/zh-cn/library/ff929188)，使应用程序能够直接向数据库进行身份验证。可以通过执行以下 T-SQL 命令，在以服务器管理员身份登录连接到用户数据库时，创建使用 SQL 身份验证的包含数据库用户：

## 授权

授权是指允许用户在 Azure SQL 数据库中执行哪些操作，这由用户帐户的数据库[角色成员身份](https://msdn.microsoft.com/zh-cn/library/ms189121)和[对象级权限](https://msdn.microsoft.com/zh-cn/library/ms191291.aspx)来控制。作为最佳实践，你应向用户授予所需的最低权限。用于连接的服务器管理员帐户是 db\_owner 所有者的成员，该帐户有权在数据库中执行任何操作。请保存此帐户，以便部署架构升级并执行其他管理操作。权限受到更多限制的“ApplicationUser”帐户可让用户使用应用程序所需的最低权限从应用程序连接到数据库。有关详细信息，请参阅[管理登录名](/documentation/articles/sql-database-manage-logins/)。

通常，只有管理员需要 master 数据库的访问权限。对每个用户数据库的例程访问应通过数据库用户进行，这些用户创建于每个数据库且不包含管理员。使用包含数据库用户时，不需要在 master 数据库中创建登录名。有关详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

此外，这些功能还可以用于限制或提升权限。

* [模拟](https://msdn.microsoft.com/zh-cn/library/vstudio/bb669087)和[模块签名](https://msdn.microsoft.com/zh-cn/library/bb669102)可用于安全地暂时提升权限。
* [行级安全性](https://msdn.microsoft.com/zh-cn/library/dn765131)可用于限制用户可访问的行。
* [数据屏蔽](/documentation/articles/sql-database-dynamic-data-masking-get-started/)可用于限制敏感数据的公开。
* [存储过程](https://msdn.microsoft.com/zh-cn/library/ms190782)可用于限制可对数据库执行的操作。

## 后续步骤

- 有关所有 SQL 数据库安全功能的概述，请参阅 [SQL 安全概述](/documentation/articles/sql-database-security-overview/)。
- 有关防火墙规则的详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。
- 若要了解用户和登录名，请参阅[管理登录名](/documentation/articles/sql-database-manage-logins/)。
- 有关 SQL 数据库中数据保护功能的用法介绍，请参阅[数据保护和安全性](/documentation/articles/sql-database-protect-data/)。
- 有关主动监视的介绍，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started/)和 [SQL 数据库威胁检测入门](/documentation/articles/sql-database-threat-detection-get-started/)。
- 有关使用 SQL Server 身份验证的教程，请参阅 [SQL 数据库教程：SQL Server 身份验证、登录名和用户帐户、数据库角色、权限、服务器级防火墙规则和数据库级防火墙规则](/documentation/articles/sql-database-control-access-sql-authentication-get-started/)。
- 有关使用 Azure Active Directory 身份验证的教程，请参阅 [SQL 数据库教程：AAD 身份验证、登录名和用户帐户、数据库角色、权限、服务器级防火墙规则和数据库级防火墙规则](/documentation/articles/sql-database-control-access-aad-authentication-get-started/)。

<!---HONumber=Mooncake_0120_2017-->