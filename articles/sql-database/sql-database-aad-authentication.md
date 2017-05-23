<properties
    pageTitle="Azure Active Directory 身份验证 - Azure SQL（概述）| Azure"
    description="了解如何通过 SQL 数据库和 SQL 数据仓库使用 Azure Active Directory 进行身份验证"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor=""
    tags="" />
<tags
    ms.assetid="7e2508a1-347e-4f15-b060-d46602c5ce7e"
    ms.service="sql-database"
    ms.custom="security-access"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="03/23/2017"
    wacn.date="05/22/2017"
    ms.author="rickbyh"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="36614a71604550cb5d5346d7a645a3276e8ee024"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="use-azure-active-directory-authentication-for-authentication-with-sql-database-or-sql-data-warehouse"></a>通过 SQL 数据库或 SQL 数据仓库使用 Azure Active Directory 身份验证进行身份验证
Azure Active Directory 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Azure SQL 数据库和 [SQL 数据仓库](/documentation/articles/sql-data-warehouse-overview-what-is/)的一种机制。 通过 Azure AD 身份验证，可在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。 集中 ID 管理提供一个单一位置来管理数据库用户，并简化权限管理。 包括如下优点：

* 提供一个 SQL Server 身份验证的替代方法。
* 帮助阻止用户标识在数据库服务器之间激增。
* 允许在单一位置中轮换密码
* 客户可以使用外部 (AAD) 组管理数据库权限。
* 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
* Azure AD 身份验证使用包含的数据库用户以数据库级别对标识进行身份验证。
* Azure AD 支持对连接到 SQL 数据库的应用程序进行基于令牌的身份验证。
* Azure AD 身份验证支持对本地 Azure Active Directory 进行 ADFS（域联合）或本机用户/密码身份验证，而无需进行域同步。  
* Azure AD 支持从 SQL Server Management Studio 进行连接，后者使用 Active Directory 通用身份验证，其中包括多重身份验证 (MFA)。  MFA 包括利用一系列简单的验证选项进行的强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知。 有关详细信息，请参阅 [SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持](/documentation/articles/sql-database-ssms-mfa-authentication/)。

配置步骤包括配置和使用 Azure Active Directory 身份验证的以下过程。

1. 创建并填充 Azure AD。
2. 可选：关联或更改当前与 Azure 订阅关联的 Active Directory。
3. 为 Azure SQL Server 或 [Azure SQL 数据仓库](/home/features/sql-data-warehouse/)创建 Azure Active Directory 管理员。
4. 配置客户端计算机。
5. 在映射到 Azure AD 标识的数据库中创建包含的数据库用户。
6. 通过使用 Azure AD 标识连接到你的数据库。

> [AZURE.NOTE]
> 若要了解如何创建和填充 Azure AD，然后使用 Azure SQL 数据库和 SQL 数据仓库配置 Azure AD，请参阅[使用 Azure SQL 数据库配置 Azure AD](/documentation/articles/sql-database-aad-authentication-configure/)。
>

## <a name="trust-architecture"></a>信任体系结构
以下高级别关系图概述了将 Azure AD 身份验证用于 Azure SQL 数据库的解决方案体系结构。 相同的概念适用于 SQL 数据仓库。 若要支持 Azure AD 本机用户密码，只需考虑云部分和 Azure AD/Azure SQL 数据库。 若要支持联合身份验证（或 Windows 凭据的用户/密码），需要与 ADFS 块进行通信。 箭头表示通信路径。

![AAD 身份验证关系图][1]

下图表明允许客户端通过提交令牌连接到数据库的联合、信任和托管关系。 该令牌已由 Azure AD 进行身份验证且受数据库信任。 客户 1 可以代表具有本机用户的 Azure Active Directory 或具有联合用户的 Azure AD。 客户 2 代表包含已导入用户的可行解决方案；在本例中，来自联合 Azure Active Directory 且 ADFS 正与 Azure Active Directory 进行同步。 请务必了解，使用 Azure AD 身份验证访问数据库需要托管订阅与 Azure AD 相关联。 必须使用相同的订阅创建托管 Azure SQL 数据库或 SQL 数据仓库的 SQL Server。

![订阅关系][2]

## <a name="administrator-structure"></a>管理员结构
使用 Azure AD 身份验证时，SQL 数据库服务器会有两个管理员帐户：原始的 SQL Server 管理员和 Azure AD 管理员。 相同的概念适用于 SQL 数据仓库。 只有基于 Azure AD 帐户的管理员可以在用户数据库中创建第一个 Azure AD 包含的数据库用户。 Azure AD 管理员登录名可以是 Azure AD 用户，也可以是 Azure AD 组。 当管理员为组帐户时，可以由任何组成员使用，为 SQL Server 实例启用多个 Azure AD 管理员。 以管理员身份使用组帐户时，允许你无需更改 SQL 数据库中的用户或权限，便可集中添加和删除 Azure AD 中的组成员，从而提高可管理性。 无论何时都仅可配置一个 Azure AD 管理员（一个用户或组）。

![管理结构][3]

## <a name="permissions"></a>权限
若要新建用户，必须具有数据库中的 `ALTER ANY USER` 权限。 `ALTER ANY USER` 权限可以授予任何数据库用户。 `ALTER ANY USER` 权限还由服务器管理员帐户、具有该数据库的 `CONTROL ON DATABASE` 或 `ALTER ON DATABASE` 权限的数据库用户以及 `db_owner` 数据库角色的成员拥有。

若要在 Azure SQL 数据库或 SQL 数据仓库中创建一个包含的数据库用户，必须使用 Azure AD 标识连接到数据库。 若要创建第一个包含数据库用户，必须通过使用 Azure AD 管理员（也是数据库的所有者）连接到数据库。 在下面的步骤 4 和 5 中演示了此操作。 只有为 Azure SQL 数据库或 SQL 数据仓库服务器创建 Azure AD 管理员之后，才有可能进行任何 Azure AD 身份验证。 如果已从服务器删除 Azure Active Directory 管理员，先前在 SQL Server 内创建的现有 Azure Active Directory 用户便无法再使用其 Azure Active Directory 凭据连接到数据库。

##<a name="azure-ad-features-and-limitations"></a> Azure AD 功能和限制
可以在 Azure SQL Server 或 SQL 数据仓库中预配以下 Azure AD 成员：

* 本机成员：在托管域或客户域中的 Azure AD 中创建的成员。 有关详细信息，请参阅[将自己的域名添加到 Azure AD](/documentation/articles/active-directory-add-domain/)。
* 联合域成员：在联合域的 Azure AD 中创建的成员。 有关详细信息，请参阅 [Azure 现在支持与 Windows Server Active Directory 联合](https://azure.microsoft.com/zh-cn/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory)。
* 作为本机或联合域成员从其他 Azure AD 导入的成员。
* 以安全组形式创建的 Active Directory 组。

不支持 Microsoft 帐户（例如 outlook.com、hotmail.com、live.com）或其他来宾帐户（例如 gmail.com、yahoo.com）。 如果可以使用帐户和密码登录到 [https://login.live.com](https://login.live.com) ，则使用的是 Microsoft 帐户，Azure SQL 数据库或 Azure SQL 数据仓库的 Azure AD 身份验证不支持此类帐户。

## <a name="connecting-using-azure-ad-identities"></a>使用 Azure AD 标识进行连接

Azure Active Directory 身份验证支持使用 Azure AD 标识连接到数据库的以下方法：

* 使用集成的 Windows 身份验证
* 使用 Azure AD 主体名称和密码
* 使用应用程序令牌身份验证

### <a name="additional-considerations"></a>其他注意事项

* 为了增强可管理性，建议将一个专用 Azure AD 组预配为管理员。   
* 无论何时都仅可为 Azure SQL Server 或 Azure SQL 数据仓库配置一个 Azure AD 管理员（一个用户或组）。   
* 只有 SQL Server 的 Azure AD 管理员最初可以使用 Azure Active Directory 帐户连接到 Azure SQL Server 或 Azure SQL 数据仓库。 Active Directory 管理员可以配置后续的 Azure AD 数据库用户。   
* 我们建议将连接超时值设置为 30 秒。   
* SQL Server 2016 Management Studio 和 SQL Server Data Tools for Visual Studio 2015（版本 14.0.60311.1（2016 年 4 月）或更高版本）支持 Azure Active Directory 身份验证。 （**用于 SqlServer 的 .NET Framework 数据提供程序**（.NET Framework 4.6 或更高版本）支持 Azure AD 身份验证）。 因此，这些工具和数据层应用程序（DAC 和 .bacpac）的最新版本可以使用 Azure AD 身份验证。   
* 虽然 [ODBC 版本 13.1](https://www.microsoft.com/download/details.aspx?id=53339) 支持 Azure Active Directory 身份验证，但是 `bcp.exe` 无法使用 Azure Active Directory 身份验证进行连接，因为使用的是旧式 ODBC 提供程序。   
* `sqlcmd` 从版本 13.1 开始就支持 Azure Active Directory 身份验证，该版本可从 [下载中心](http://go.microsoft.com/fwlink/?LinkID=825643)下载。   
* SQL Server Data Tools for Visual Studio 2015 至少需要 2016 年 4 月版的 Data Tools（版本 14.0.60311.1）。 目前，Azure AD 用户不会显示在 SSDT 对象资源管理器中。 解决方法是在 [sys.database_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx) 中查看这些用户。   
* [Microsoft JDBC Driver 6.0 for SQL Server](https://www.microsoft.com/download/details.aspx?id=11774) 支持 Azure AD 身份验证。 另外，请参阅[设置连接属性](https://msdn.microsoft.com/zh-cn/library/ms378988.aspx)。   
* PolyBase 无法使用 Azure AD 身份验证进行身份验证。   
* Azure 门户预览的“导入数据库”和“导出数据库”边栏选项卡支持 SQL 数据库的 Azure AD 身份验证。 PowerShell 命令也支持使用 Azure AD 身份验证的导入和导出。   

## <a name="next-steps"></a>后续步骤
- 若要了解如何创建和填充 Azure AD，然后如何使用 Azure SQL 数据库配置 Azure AD，请参阅[使用 Azure SQL 数据库配置 Azure AD](/documentation/articles/sql-database-aad-authentication-configure/)。
- 有关 SQL 数据库中的访问和控制的概述，请参阅 [SQL 数据库访问和控制](/documentation/articles/sql-database-control-access/)。
- 有关 SQL 数据库中的登录名、用户和数据库角色的概述，请参阅[登录名、用户和数据库角色](/documentation/articles/sql-database-manage-logins/)。
- 有关数据库主体的详细信息，请参阅[主体](https://msdn.microsoft.com/zh-cn/library/ms181127.aspx)。
- 有关数据库角色的详细信息，请参阅[数据库角色](https://msdn.microsoft.com/zh-cn/library/ms189121.aspx)。
- 有关 SQL 数据库中的防火墙规则的详细信息，请参阅 [SQL 数据库防火墙规则](/documentation/articles/sql-database-firewall-configure/)。
- 有关使用 SQL Server 身份验证的教程，请参阅 [SQL 身份验证和授权](/documentation/articles/sql-database-control-access-sql-authentication-get-started/)。
- 有关如何使用 Azure Active Directory 身份验证的教程，请参阅 [Azure AD 身份验证和授权](/documentation/articles/sql-database-control-access-aad-authentication-get-started/)。

<!--Image references-->

[1]: ./media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: ./media/sql-database-aad-authentication/2subscription-relationship.png
[3]: ./media/sql-database-aad-authentication/3admin-structure.png
[4]: ./media/sql-database-aad-authentication/4select-subscription.png
[5]: ./media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: ./media/sql-database-aad-authentication/6edit-directory-select.png
[7]: ./media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: ./media/sql-database-aad-authentication/8choose-ad.png
[9]: ./media/sql-database-aad-authentication/9ad-settings.png
[10]: ./media/sql-database-aad-authentication/10choose-admin.png
[11]: ./media/sql-database-aad-authentication/11connect-using-int-auth.png
[12]: ./media/sql-database-aad-authentication/12connect-using-pw-auth.png
[13]: ./media/sql-database-aad-authentication/13connect-to-db.png
<!--Update_Description: wording update;add anchors to subtitle -->