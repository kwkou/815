<properties
	pageTitle="Azure AD Connect 同步：如何管理 Azure AD 服务帐户 | Azure"
	description="本主题介绍如何还原 Azure AD 服务帐户。"
	services="active-directory"
    keywords="如何重置 Azure AD Connect 同步连接器服务帐户的密码"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="06/10/2016"
	wacn.date="07/12/2016"/>

# Azure AD Connect 同步：如何管理 Azure AD 服务帐户

Azure AD 连接器所使用的服务帐户应该是免费服务。但如果你需要重置其凭据，则本主题适合你。例如，如果全局管理员错误地使用 PowerShell 对服务帐户重置了密码，则可能会发生这种情况。

## 重置凭据

如果 Azure AD 连接器上定义的服务帐户由于身份验证问题无法联系 Azure AD，则可以重置密码。

1. 登录到 Azure AD Connect 同步服务器并启动 PowerShell。
2. 运行 `Add-ADSyncAADServiceAccount`。  
![PowerShell cmdlet addadsyncaadserviceaccount](./media/active-directory-aadconnectsync-howto-azureadaccount/addadsyncaadserviceaccount.png)
3. 提供 Azure AD 全局管理员凭据。

此 cmdlet 将重置服务帐户的密码，并同时在 Azure AD 和同步引擎中更新该密码。

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0704_2016-->