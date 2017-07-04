<properties
    pageTitle="Azure AD Connect 同步：目录扩展 | Azure"
    description="本主题介绍 Azure AD Connect 中的目录扩展功能。"
    services="active-directory"
    documentationcenter=""
    author="AndKjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="995ee876-4415-4bb0-a258-cca3cbb02193"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/08/2017"
    wacn.date="03/13/2017"
    ms.author="billmath" />

# Azure AD Connect 同步：目录扩展
目录扩展支持使用本地 Active Directory 中你自己的属性来扩展 Azure AD 中的架构。借助此功能，可以构建 LOB 应用并让其使用可继续在本地管理的属性。可以通过 [Azure AD Graph 目录扩展](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions)或 [Microsoft Graph](https://graph.microsoft.io/) 使用这些属性。使用 [Azure AD Graph 资源管理器](https://graphexplorer.cloudapp.net)和 [Microsoft Graph 资源管理器](https://graphexplorer2.azurewebsites.net/)可以分别查看可用属性。

目前没有任何 Office 365 工作负荷使用这些属性。

在安装向导的自定义设置路径中配置要同步的其他属性。
![架构扩展向导](./media/active-directory-aadconnectsync-feature-directory-extensions/extension2.png) 
安装显示以下属性，它们是有效的候选项：

- “用户”和“组”对象类型
- 单值属性：String、Boolean、Integer、Binary
- 多值属性：String、Binary

属性列表是在安装 Azure AD Connect 的过程中从架构缓存读取的。如果已使用附加属性扩展 Active Directory 架构，则只有在[刷新架构](/documentation/articles/active-directory-aadconnectsync-installation-wizard/#refresh-directory-schema/)后，这些新属性才可见。

Azure AD 中的对象最多可以有 100 个目录扩展属性。最大长度为 250 个字符。如果属性值长度超过此限制，同步引擎会将其截断。

在安装 Azure AD Connect 期间，将会注册可以使用这些属性的应用程序。

现在可以通过 Graph 使用这些属性：
![Graph](./media/active-directory-aadconnectsync-feature-directory-extensions/extension4.png)

这些属性带有 extension\_{AppClientId}\_ 前缀。Azure AD 租户中的所有属性具有相同的 AppClientId 值。

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->
