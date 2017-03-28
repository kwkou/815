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
    ms.custom="authentication and authorization"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="01/23/2017"
    wacn.date="03/24/2017"
    ms.author="rickbyh" />

# SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持
现在，Azure SQL 数据库和 Azure SQL 数据仓库支持使用 *Active Directory 通用身份验证*，从 SQL Server Management Studio (SSMS) 进行连接。Active Directory 通用身份验证是支持 *Azure 多重身份验证* (MFA) 的交互式工作流。Azure MFA 可帮助保护对数据和应用程序的访问，同时满足用户对简单登录的需求。它利用一系列简单的验证选项提供强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知，用户可以根据自己的偏好选择所用的方法。

* 有关多重身份验证的说明，请参阅[多重身份验证](/documentation/articles/multi-factor-authentication/)。
* 有关配置步骤，请参阅[为 SQL Server Management Studio 配置 Azure SQL 数据库多重身份验证](/documentation/articles/sql-database-ssms-mfa-authentication-configure/)。

## 多重选项

SSMS 现在支持：

* 搭配 Azure AD 使用交互式 MFA，可能会进行弹出式对话框验证。
* 非交互式 Active Directory 密码和 Active Directory 集成身份验证方法，可在许多不同的应用程序（ADO.NET、JDBC、ODBC 等）中使用。这两种方法绝对不会产生弹出式对话框。

将用户帐户配置为使用 MFA 时，交互式身份验证工作流会通过弹出式对话框、使用智能卡等方式，要求额外的用户交互。将用户帐户配置为使用 MFA 时，用户必须选择要连接的 Azure 通用身份验证。如果用户帐户不需要 MFA，用户仍然可以使用另外两个 Azure Active Directory 身份验证选项。

## SQL 数据库和 SQL 数据仓库的 Active Directory 通用身份验证限制
* SSMS 是目前唯一通过 Active Directory 通用身份验证针对 MFA 启用的工具。
* 使用通用身份验证登录到某个 SSMS 实例的 Azure Active Directory 帐户只能有一个。若要以另一个 Azure AD 帐户登录，则必须使用另一个 SSMS 实例。（此限制仅限于 Active Directory 通用身份验证；如果使用 Active Directory 密码验证、Active Directory 集成身份验证或 SQL Server 身份验证，可以登录到不同的服务器）。
* 对于对象资源管理器、查询编辑器和查询存储可视化效果，SSMS 支持 Active Directory 通用身份验证。
* DacFx 和架构设计器均不支持通用身份验证。
* Active Directory 通用身份验证不支持 MSA 帐户。
* 对于从其他 Active Directory 导入到当前 Active Directory 的用户，SSMS 不支持 Azure Active Directory 通用身份验证。不支持这些用户，因为需要使用租户 ID 验证帐户，但没有任何机制提供这种验证。
* 除了必须使用支持的 SSMS 版本，Active Directory 通用身份验证没有其他软件需求。



## 后续步骤

* 有关配置步骤，请参阅[为 SQL Server Management Studio 配置 Azure SQL 数据库多重身份验证](/documentation/articles/sql-database-ssms-mfa-authentication-configure/)。
* 向其他人授权访问你的数据库：[SQL 数据库身份验证和授权：授予访问权限](/documentation/articles/sql-database-manage-logins/) 确保其他人可以通过防火墙建立连接：[使用 Azure 门户配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: simplify content structure; remove configuration steps-->