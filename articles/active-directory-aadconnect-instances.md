<properties
	pageTitle="Azure AD Connect：同步服务实例 | Azure"
	description="本页记录了 Azure AD 实例的特殊注意事项。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/10/2016"
	wacn.date="06/24/2016"/>

# Azure AD Connect：有关实例的特殊注意事项
Azure AD Connect 最常用于全球范围内的 Azure AD 和 Office 365 实例。但也有其他实例，这些实例对 URL 具有不同的要求并且具有其他的特殊注意事项。

## 德国 Microsoft 云
[德国 Microsoft 云](http://www.microsoft.de/cloud-deutschland)是由德国数据信托运营的最高等级的云。

此云目前提供预览版。许多通常可以由自己完成的方案（如验证域）必须由运营商完成。有关如何参加使用预览版的详细信息，请与你当地的 Microsoft 代表联系。

在代理服务器中打开的 URL |
--- |
*.microsoftonline.de |
*.windows.net |
\+ 证书吊销列表 |

在登录你的 Azure AD 目录时，必须使用 onmicrosoft.de 域中的帐户。

德国 Microsoft 云中当前不存在的功能：

- Azure AD Connect Health 未提供。
- 自动更新未提供。
- 密码写回未提供。

## Microsoft Azure 政府版云
[Microsoft Azure 政府版云](https://azure.microsoft.com/features/gov/)是用于美国政府的云。

DirSync 的早期版本支持该云。从 Azure AD Connect 的 1.1.180 版本起，将支持下一代云。这一代使用的是基于仅限美国的终结点，并具有不同的 URL 列表，可在你的代理服务器中打开。

在代理服务器中打开的 URL |
--- |
*.microsoftonline.com |
*.gov.us.microsoftonline.com |
\+ 证书吊销列表 |

Azure AD Connect 不能自动检测到你的 Azure AD 目录位于政府版的云中。当你安装 Azure AD Connect 时，需要改为执行以下操作。

1. 开始 Azure AD Connect 安装。
2. 当你看到第一页后，这时应该接受 EULA，请不要继续，而是让安装向导运行。
3. 启动 regedit 并将注册表项 `HKLM\SOFTWARE\Microsoft\Azure AD Connect\AzureInstance` 更改为值 `2`。
4. 返回 Azure AD Connect 安装向导，接受 EULA，然后继续。在安装期间，请确保使用“自定义配置”安装路径（而不是快速安装）。然后，像往常一样继续安装。

Microsoft Azure 政府版云中当前不存在的功能：

- Azure AD Connect Health 未提供。
- 自动更新未提供。
- 密码写回未提供。

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0613_2016-->