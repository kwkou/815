<properties
    pageTitle="多重身份验证 - Azure SQL | Azure"
    description="将多重身份验证与 SSMS 共同用于 SQL 数据库和 SQL 数据仓库。"
    services="sql-database"
    documentationcenter=""
    author="BYHAM"
    manager="jhubbard"
    editor=""
    tags="" />
<tags
    ms.assetid="fbd6e644-0520-439c-8304-2e4fb6d6eb91"
    ms.service="sql-database"
    ms.custom="security-access"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="04/07/2017"
    wacn.date="05/22/2017"
    ms.author="rickbyh"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="72e9f54d79be380cdde6e4388d91ea65b579d01c"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="universal-authentication-with-sql-database-and-sql-data-warehouse-ssms-support-for-mfa"></a>使用 SQL 数据库和 SQL 数据仓库进行通用身份验证（MFA 的 SSMS 支持）
Azure SQL 数据库和 Azure SQL 数据仓库支持使用 Active Directory 通用身份验证，从 SQL Server Management Studio (SSMS) 进行连接。 

- Active Directory 通用身份验证支持两种非交互式身份验证方法（Active Directory 密码身份验证和 Active Directory 集成身份验证）。 非交互式 Active Directory 密码和 Active Directory 集成身份验证方法，可在许多不同的应用程序（ADO.NET、JDBC、ODBC 等）中使用。 这两种方法绝对不会产生弹出式对话框。

- Active Directory 通用身份验证是同时支持 *Azure 多重身份验证* (MFA) 的交互式方法。 Azure MFA 可帮助保护对数据和应用程序的访问，同时满足用户对简单登录过程的需求。 它利用一系列简单的验证选项提供强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知，用户可以根据自己的偏好选择所用的方法。 配合使用 Azure AD 和交互式 MFA 时会出现用于验证的弹出式对话框。

有关多重身份验证的说明，请参阅[多重身份验证](/documentation/services/multi-factor-authentication/)。
有关配置步骤，请参阅[配置 SQL Server Management Studio 的 Azure SQL 数据库多重身份验证](/documentation/articles/sql-database-ssms-mfa-authentication-configure/)。

### <a name="azure-ad-domain-name-or-tenant-id-parameter"></a>Azure AD 域名称或租户 ID 参数   

从 [SSMS 版本 17](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 开始，从其他 Azure Active Directory 导入到当前 Active Directory 的用户在连接时可提供 Azure AD 域名或租户 ID。 这使 **Active Directory 通用身份验证**可以识别正确的身份验证机构。 此选项也是支持 outlook.com、hotmail.com 或 live.com 等 Microsoft 账户 (MSA) 的必需选项。 所有要使用通用身份验证进行身份验证的用户必须输入其 Azure AD 域名或租户 ID。 此参数表示 Azure 服务器当前链接的 Azure AD 域名/租户ID。 例如，如果 Azure Server 与 Azure AD 域 `contosotest.onmicrosoft.com`（其中用户 `joe@contosodev.onmicrosoft.com` 托管为从 Azure AD 域 `contosodev.onmicrosoft.com` 导入的用户）相关联，则需用于对此用户进行身份验证的域名为 `contosotest.onmicrosoft.com`。 如果用户是链接到 Azure 服务器的 Azure AD 的本机用户，并且不是 MSA 帐户，则无需提供域名或租户 ID。 若要输入参数（从 SSMS 版本 17 开始），请在“连接到数据库”对话框中，选择“Active Directory 通用身份验证”，依次单击“选项”、“连接属性”选项卡，检查“AD 域名或租户 ID”框，并提供身份验证机构，如域名 (**contosotest.onmicrosoft.com**) 或租户 ID 的 GUID。

## <a name="universal-authentication-limitations-for-sql-database-and-sql-data-warehouse"></a>SQL 数据库和 SQL 数据仓库的 Active Directory 通用身份验证限制
* SSMS 是目前唯一通过 Active Directory 通用身份验证针对 MFA 启用的工具。
* 使用通用身份验证登录到某个 SSMS 实例的 Azure Active Directory 帐户只能有一个。 若要以另一个 Azure AD 帐户登录，则必须使用另一个 SSMS 实例。 （此限制仅限于 Active Directory 通用身份验证；如果使用 Active Directory 密码验证、Active Directory 集成身份验证或 SQL Server 身份验证，可以登录到不同的服务器）。
* 对于对象资源管理器、查询编辑器和查询存储可视化效果，SSMS 支持 Active Directory 通用身份验证。
* DacFx 和架构设计器均不支持通用身份验证。
* 除了必须使用支持的 SSMS 版本，Active Directory 通用身份验证没有其他软件需求。


## <a name="next-steps"></a>后续步骤

* 有关配置步骤，请参阅[配置 SQL Server Management Studio 的 Azure SQL 数据库多重身份验证](/documentation/articles/sql-database-ssms-mfa-authentication-configure/)。
* 向其他人员授予数据库访问权限：[SQL 数据库身份验证和授权：授予访问权限](/documentation/articles/sql-database-manage-logins/)  
确保其他人员可以通过防火墙进行连接：[使用 Azure 门户配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)
<!--Update_Description: simplify content-->