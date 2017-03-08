<properties
    pageTitle="Azure AD Connect：同步服务实例 | Azure"
    description="本页记录了有关 Azure AD 实例的特殊注意事项。"
    services="active-directory"
    documentationcenter=""
    author="andkjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="f340ea11-8ff5-4ae6-b09d-e939c76355a3"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/07/2017"
    wacn.date="03/07/2017"
    ms.author="billmath" />  


# Azure AD Connect：有关实例的特殊注意事项
Azure AD Connect 最常用于全球范围内的 Azure AD 和 Office 365 实例。但也有其他实例，这些实例对 URL 具有不同的要求并且具有其他的特殊注意事项。

## 德国 Microsoft 云 <a name="microsoft-cloud-germany"></a>
[德国 Microsoft 云](http://www.microsoft.de/cloud-deutschland)是由德国数据信托运营的最高等级的云。

| 将在代理服务器中打开的 URL |
| --- |
| *.microsoftonline.de |
| *.chinacloudapi.cn |
| + 证书吊销列表 |

在登录 Azure AD 租户时，必须使用 onmicrosoft.de 域中的帐户。

德国 Microsoft 云中当前不存在的功能：

- **Azure AD Connect Health** 不可用。
- **自动更新**不可用。
- **密码写回**不可用。
- 其他 Azure AD Premium 服务不可用。

## Azure 政府版云

DirSync 的早期版本已支持该云。从 Azure AD Connect 的 1.1.180 版本起，将支持该云的新一代版本。这一代使用的是基于仅限美国的终结点，并具有不同的 URL 列表，可在你的代理服务器中打开。

| 将在代理服务器中打开的 URL |
| --- |
| *.microsoftonline.com |
| *.gov.us.microsoftonline.com |
| + 证书吊销列表 |

Azure AD Connect 不能自动检测到 Azure AD 租户位于政府版云中。当你安装 Azure AD Connect 时，需要改为执行以下操作。

1. 开始 Azure AD Connect 安装。
2. 看到第一页后，应接受其中的 EULA，但请不要继续，而是让安装向导运行。
3. 启动 regedit 并将注册表项 `HKLM\SOFTWARE\Microsoft\Azure AD Connect\AzureInstance` 更改为值 `2`。
4. 返回 Azure AD Connect 安装向导，接受 EULA，然后继续。在安装期间，请确保使用“自定义配置”安装路径（而不是快速安装）。然后，像往常一样继续安装。

Azure 政府版云中当前不存在的功能：

- **Azure AD Connect Health** 不可用。
- **自动更新**不可用。
- **密码写回**不可用。
- 其他 Azure AD Premium 服务不可用。

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0227_2017-->
<!---Update_Description: wording update -->