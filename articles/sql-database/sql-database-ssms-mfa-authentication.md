<properties
   pageTitle="SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持 | Azure"
   description="将 Multi-Factored Authentication 与 SSMS 搭配用于 SQL 数据库和 SQL 数据仓库。"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jhubbard"
   editor=""
   tags=""/>  


<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="10/04/2016"
   ms.author="rick.byham@microsoft.com"/>

# SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持

现在，Azure SQL 数据库和 Azure SQL 数据仓库支持使用 *Active Directory 通用身份验证*，从 SQL Server Management Studio (SSMS) 进行连接。Active Directory 通用身份验证是支持 *Azure 多重身份验证* (MFA) 的交互式工作流。Azure MFA 可帮助保护对数据和应用程序的访问，同时满足用户对简单登录的需求。它利用一系列简单的验证选项提供强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知，用户可以根据自己的偏好选择所用的方法。有关多重身份验证的说明，请参阅[多重身份验证](/documentation/articles/multi-factor-authentication/)。

SSMS 现在支持：

- 搭配 Azure AD 使用交互式 MFA，可能会进行弹出式对话框验证。
- 非交互式 Active Directory 密码和 Active Directory 集成身份验证方法，可在许多不同的应用程序（ADO.NET、JDBC、ODBC 等）中使用。这两种方法绝对不会产生弹出式对话框。

将用户帐户配置为使用 MFA 时，交互式身份验证工作流会通过弹出式对话框、使用智能卡等方式，要求额外的用户交互。将用户帐户配置为使用 MFA 时，用户必须选择要连接的 Azure 通用身份验证。如果用户帐户不需要 MFA，用户仍然可以使用另外两个 Azure Active Directory 身份验证选项。

## SQL 数据库和 SQL 数据仓库的 Active Directory 通用身份验证限制

- SSMS 是目前唯一通过 Active Directory 通用身份验证针对 MFA 启用的工具。
- 使用通用身份验证登录到某个 SSMS 实例的 Azure Active Directory 帐户只能有一个。若要以另一个 Azure AD 帐户登录，则必须使用另一个 SSMS 实例。（此限制仅限于 Active Directory 通用身份验证；如果使用 Active Directory 密码验证、Active Directory 集成身份验证或 SQL Server 身份验证，可以登录到不同的服务器）。
- 对于对象资源管理器、查询编辑器和查询存储可视化效果，SSMS 支持 Active Directory 通用身份验证。
- DacFx 和架构设计器均不支持通用身份验证。
- Active Directory 通用身份验证不支持 MSA 帐户。
- 对于从其他 Active Directory 导入到当前 Active Directory 的用户，SSMS 不支持 Azure Active Directory 通用身份验证。不支持这些用户，因为需要使用租户 ID 验证帐户，但没有任何机制提供这种验证。
- 除了必须使用支持的 SSMS 版本，Active Directory 通用身份验证没有其他软件需求。

## 配置步骤

实现多重身份验证需要四个基本步骤。

1. **配置 Azure Active Directory** — 有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)、[将自己的域名添加到 Azure AD](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)、[Microsoft Azure now supports federation with Windows Server Active Directory](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/)（Microsoft Azure 现在支持与 Windows Server Active Directory 联合）、[管理 Azure AD 目录](https://msdn.microsoft.com/zh-cn/library/azure/hh967611.aspx)和[使用 Windows PowerShell 管理 Azure AD](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx)。

2. **配置 MFA** — 有关分步说明，请参阅[配置 Azure 多重身份验证](/documentation/articles/multi-factor-authentication-whats-next/)。

3. **配置 SQL 数据库或 SQL 数据仓库以进行 Azure AD 身份验证** — 有关分步说明，请参阅[使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库](/documentation/articles/sql-database-aad-authentication/)。

4. **下载 SSMS** — 在客户端计算机上，从[下载 SQL Server Management Studio (SSMS)](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 下载最新的 SSMS（至少是 2016 年 8 月版）。

## 使用通用身份验证搭配 SSMS 进行连接

以下步骤演示如何使用最新的 SSMS 连接到 SQL 数据库或 SQL 数据仓库。

1. 若要使用通用身份验证进行连接，请在“连接到服务器”对话框中选择“Active Directory 通用身份验证”。
![1mfa-universal-connect][1]

2. 像往常一样，必须对 SQL 数据库和 SQL 数据仓库单击“选项”，然后在“选项”对话框中指定数据库。单击“连接”。
3. 显示“登录到帐户”对话框时，提供 Azure Active Directory 标识的帐户和密码。
![2mfa-sign-in][2]

    > [AZURE.NOTE] 如果使用不需要 MFA 的帐户进行通用身份验证，可以在此时连接。对于需要 MFA 的用户，请继续执行以下步骤。
 
4. 可能会显示两个 MFA 设置对话框。这个一次性操作根据 MFA 管理员设置而定，因此可能是可选的。对于已启用 MFA 的域，有时会预定义此步骤（例如，域会要求用户使用智能卡和 PIN 码）。
![3mfa-setup][3]

5. 通过第二个可能出现的一次性对话框，可以选择身份验证方法的详细信息。可能的选项由管理员配置。
![4mfa-verify-1][4]
 
6. Azure Active Directory 向你发送确认信息。收到验证码后，将其输入“输入验证码”框，然后单击“登录”。
![5mfa-verify-2][5]

验证完成后，SSMS 便会正常连接（假设凭据和防火墙访问有效）。

##后续步骤  

向其他人授权访问你的数据库：[SQL 数据库身份验证和授权：授予访问权限](/documentation/articles/sql-database-manage-logins/) 确保其他人可以通过防火墙建立连接：[使用 Azure 门户配置 Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-configure-firewall-settings/)


[1]: ./media/sql-database-ssms-mfa-auth/1mfa-universal-connect.png
[2]: ./media/sql-database-ssms-mfa-auth/2mfa-sign-in.png
[3]: ./media/sql-database-ssms-mfa-auth/3mfa-setup.png
[4]: ./media/sql-database-ssms-mfa-auth/4mfa-verify-1.png
[5]: ./media/sql-database-ssms-mfa-auth/5mfa-verify-2.png

<!---HONumber=Mooncake_1024_2016-->