<properties
    pageTitle="Azure AD Connect：无缝单一登录故障排除 | Azure"
    description="本主题介绍了如何排除 Azure Active Directory 无缝单一登录（Azure AD 无缝 SSO）故障。"
    services="active-directory"
    keywords="什么是 Azure AD Connect, 安装 Active Directory, Azure AD 所需的组件, SSO, 单一登录"
    documentationcenter=""
    author="swkrish"
    manager="femila" />
<tags
    ms.assetid="9f994aca-6088-40f5-b2cc-c753a4f41da7"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/08/2017"
    ms.author="billmath"
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="6fda447436bcf1166c9584f8d06d1b64a6bcdc0b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="how-to-troubleshoot-azure-active-directory-seamless-single-sign-on"></a>如何排除 Azure Active Directory 无缝单一登录故障

## <a name="known-issues"></a>已知问题

- 如果使用 Azure AD Connect 对 30 个以上 AD 林进行同步，则用来设置无缝 SSO 的向导不能正常工作。 作为一种解决方法，可以在租户中[手动启用](#manual-reset-of-azure-ad-seamless-sso)无缝 SSO 功能。
- 将 Azure AD 服务 URL（https://autologon.microsoftazuread-sso.com、https://aadg.chinacloudapi.cn.nsatc.net）添加到“受信任的站点”区域而非“本地 Intranet”区域。

## <a name="troubleshooting-checklist"></a>故障排除清单

使用以下清单来排除 Azure AD 无缝 SSO 故障：

1. 在 Azure AD Connect 工具中检查是否已在租户中启用无缝 SSO 功能。 如果无法启用该功能（例如，由于端口被阻止），请确保事先满足所有[先决条件](/documentation/articles/active-directory-aadconnect-sso/#pre-requisites/)。 如果启用该功能时仍出现问题，请联系 Microsoft 支持部门。
2. 已将两个服务 URL（https://autologon.microsoftazuread-sso.com 和 https://aadg.chinacloudapi.cn.nsatc.net）定义为 Intranet 区域设置的一部分。
3. 确保企业桌面已加入 AD 域。
4. 确保用户使用 AD 域帐户登录到桌面。
5. 确保用户的帐户来自已设置了无缝 SSO 的 AD 林。
6. 确保已在企业网络中连接该桌面。
7. 确保桌面的时间与 Active Directory 同步，并且各域控制器的时间偏差不超过 5 分钟。
8. 从用户的桌面中清除现有的 Kerberos 票证。 为此，可以在命令提示符下运行 **klist purge** 命令。 用户的 Kerberos 票证的有效期通常为 12 小时；请注意，你可能在 Active Directory 中对此进行了不同的设置。
9. 查看浏览器的控制台日志（在“开发人员工具”下）来帮助确定潜在问题。
10. 查看[域控制器日志](#domain-controller-logs)。

### <a name="domain-controller-logs"></a>域控制器日志

如果在域控制器上成功启用审核，则每当用户使用无缝 SSO 登录时，将在事件日志中记录一个安全条目（与计算机帐户 **AzureADSSOAcc$** 关联的事件 4769）。 可以使用以下查询来查找这些安全事件：

    <QueryList>
      <Query Id="0" Path="Security">
        <Select Path="Security">*[EventData[Data[@Name='ServiceName'] and (Data='AZUREADSSOACC$')]]</Select>
      </Query>
    </QueryList>

## <a name="manual-reset-of-azure-ad-seamless-sso"></a>手动重置 Azure AD 无缝 SSO

如果故障排除不起作用，请使用以下步骤在你的租户中手动重置 / 启用该功能：

### <a name="1-import-the-seamless-sso-powershell-module"></a>1.导入无缝 SSO PowerShell 模块

- 首先，下载并安装 [Microsoft Online Services 登录助手](http://go.microsoft.com/fwlink/?LinkID=286152)。
- 然后下载并安装[用于 Windows PowerShell 的 64 位 Azure Active Directory 模块](http://go.microsoft.com/fwlink/p/?linkid=236297)。
- 导航到 `%programfiles%\Azure Active Directory Connect` 文件夹。
- 使用以下命令导入无缝 SSO PowerShell 模块：`Import-Module .\AzureADSSO.psd1`。

### <a name="2-get-the-list-of-ad-forests-on-which-seamless-sso-has-been-enabled"></a>2.获取已在其中启用了无缝 SSO 的 AD 林的列表

- 在 PowerShell 中，调用 `New-AzureADSSOAuthenticationContext`。 这应当会提供一个弹出窗口，用于输入 Azure AD 租户管理员凭据。
- 调用 `Get-AzureADSSOStatus`。 这将提供已在其中启用了此功能的 AD 林的列表（请查看“域”列表）。

### <a name="3-disable-seamless-sso-for-each-ad-forest-that-it-was-set-it-up-on"></a>3.为已设置了无缝 SSO 的每个 AD 林禁用该功能

- 在 PowerShell 中，调用 `New-AzureADSSOAuthenticationContext`。 这应当会提供一个弹出窗口，用于输入 Azure AD 租户管理员凭据。
- 调用 `$creds = Get-Credential`。 这应当会提供一个弹出窗口，用于输入目标 AD 林的域管理员凭据。
- 调用 `Disable-AzureADSSOForest -OnPremCredentials $creds`。 这将从本地 DC 中删除 AZUREADSSOACCT 计算机帐户并为此特定的 AD 林禁用此功能。
- 针对已设置了此功能的每个 AD 林重复上述步骤。

### <a name="4-enable-seamless-sso-for-each-ad-forest"></a>4.为每个 AD 林启用无缝 SSO

- 调用 `New-AzureADSSOAuthenticationContext`。 这应当会提供一个弹出窗口，用于输入 Azure AD 租户管理员凭据。
- 调用 `Enable-AzureADSSOForest`。 这应当会提供一个弹出窗口，用于输入目标 AD 林的域管理员凭据。
- 针对要设置此功能的每个 AD 林重复上述步骤。

### <a name="5-enable-seamless-sso-on-your-tenant"></a>5.在租户中启用无缝 SSO

- 调用 `New-AzureADSSOAuthenticationContext`。 这应当会提供一个弹出窗口，用于输入 Azure AD 租户管理员凭据。
- 调用 `Enable-AzureADSSO` 并在 `Enable: ` 提示符处键入“true”，以在租户中启用此功能。

