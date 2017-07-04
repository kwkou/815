<properties
    pageTitle="将 Azure Active Directory 单一登录与 SaaS 应用相集成 | Azure"
    description="为 Azure Active Directory 中的 SaaS 应用启用单一登录身份验证和用户设置集中式访问管理。 有关如何将 Azure Active Directory 集成到 SaaS 应用的概述。"
    services="active-directory"
    keywords="将 Azure AD 与 SaaS 应用相集成"
    documentationcenter=""
    author="curtand"
    manager="femila"
    editor="" />
<tags
    ms.assetid="53b9d341-d1fc-4bbb-ac7c-3f4c68fcf00a"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="05/04/2017"
    wacn.date="06/12/2017"
    ms.author="curtand"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="e5afc5eeaeda0295569dbf5431cf18ecdef77477"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="integrate-azure-active-directory-single-sign-on-with-saas-apps"></a>将 Azure Active Directory 单一登录与 SaaS 应用相集成  

[AZURE.INCLUDE [active-directory-sso-use-case-intro](../../includes/active-directory-sso-use-case-intro.md)]

若要开始为组织中安装的应用设置单一登录，将要使用 Azure Active Directory (Azure AD) 中的现有目录。 可以使用通过 Azure、Office 365 或 Windows Intune 获取的 Azure AD 目录。 如果有两个或更多个目录，请参阅[管理 Azure AD 目录](/documentation/articles/active-directory-administer/)来确定要使用哪一个。

## <a name="authentication"></a>身份验证
对于支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 协议的应用程序，Azure Active Directory 将使用签名证书来建立信任关系。 有关详细信息，请参阅[管理联合单一登录的证书](/documentation/articles/active-directory-sso-certs/)。

对于仅支持基于 HTML 表单的登录的应用程序，Azure Active Directory 将使用“密码保管”来建立信任关系。 这样，组织中的用户便可以使用 SaaS 应用程序中的用户帐户信息自动登录到 SaaS 应用程序。 Azure AD 将收集并安全地存储用户帐户信息和相关密码。 有关详细信息，请参阅[基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/#password-based-single-sign-on/)。

## <a name="authorization"></a>授权
设置的帐户通过单一登录身份验证之后，可授权用户使用应用程序。 用户设置可以手动完成，在某些情况下，你可以根据 Azure Active Directory 中所做的更改在 SaaS 应用中添加和删除用户信息。 

否则，你可以手动将用户信息添加到应用，或使用应用商店中提供的其他设置解决方案。

## <a name="access"></a>访问
Azure AD 提供多种可自定义的方式来向组织中的最终用户部署应用程序。 你不会受限于任一特定的部署或访问解决方案。 可以使用[最符合需要的解决方案](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users/)。

## <a name="additional-considerations-for-applications-already-in-use"></a>有关使用中应用程序的其他注意事项
为组织中已在使用的应用程序设置单一登录的过程，与为新应用程序创建新帐户的过程不同。 基本步骤包括：将应用程序中的用户标识映射到 Azure AD 标识，以及了解集成之后用户如何体验登录应用程序。

> [AZURE.NOTE]
> 若要为现有应用程序设置 SSO，你需要有 Azure AD 和 SaaS 应用程序的全局管理员权限。
>
>

### <a name="mapping-user-accounts"></a>映射用户帐户
用户的标识通常有唯一标识符，可以是电子邮件地址或用户主体名称 (UPN)。 需要将每个用户的应用程序标识链接（映射）到其各自的 Azure AD 标识。 根据应用程序身份验证的要求，有几种方法可实现此目的。

有关映射应用程序标识与 Azure AD 标识的详细信息，请参阅 [自定义 SAML 令牌中颁发的声明](http://social.technet.microsoft.com/wiki/contents/articles/31257.azure-active-directory-customizing-claims-issued-in-the-saml-token-for-pre-integrated-apps.aspx)。

### <a name="understanding-the-users-log-in-experience"></a>了解用户的登录体验
为使用中的应用程序集成 SSO 时，请务必认识到用户体验会受到影响。 对于所有应用程序，用户将开始使用其 Azure AD 凭据来登录。 他们还可能需要使用不同的门户来访问应用程序。

针对某些应用程序的 SSO 可在应用程序本身的登录界面上完成，但对于其他应用程序，用户必须通过[中心门户](https://login.partner.microsoftonline.cn)（例如“我的应用”或“Office365”）来登录。请在 [Azure Active Directory 的应用程序访问与单一登录是什么](/documentation/articles/active-directory-appssoaccess-whatis/)中了解有关不同类型的 SSO 及其用户体验的详细信息。

另一个有价值的资源是[开发人员指导](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)一文中的*隐藏用户同意*。

## <a name="next-steps"></a>后续步骤

如果某个应用不在应用库中，你可以 [将该应用作为自定义应用程序添加到 Azure AD 应用库](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)。

Azure.com 库中还提供了有关所有这些问题的更多详细信息，请先阅读 [Azure Active Directory 的应用程序访问与单一登录是什么](/documentation/articles/active-directory-appssoaccess-whatis/)。

此外，请不要错过[有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)。

<!---Update_Description: wording update -->