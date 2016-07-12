<properties
    pageTitle="将 Azure Active Directory 单一登录与 SaaS 应用相集成 | Azure"
    description="为 Azure Active Directory 中的 SaaS 应用启用单一登录身份验证和用户预配集中式访问管理。有关如何将 Azure Active Directory 集成到 SaaS 应用的概述。"
    services="active-directory"
    keywords="将 Azure AD 与 SaaS 应用相集成"
    documentationCenter=""
    authors="curtand"
    manager="stevenpo"
    editor=""/>

<tags
      ms.service="active-directory"
    ms.date="04/26/2016"
      wacn.date="06/27/2016"/>

# 将 Azure Active Directory 单一登录与 SaaS 应用相集成  

[AZURE.INCLUDE [active-directory-sso-use-case-intro](../includes/active-directory-sso-use-case-intro.md)]

若要开始为组织中安装的应用设置单一登录，你将要使用 Azure Active Directory (Azure AD) 中的现有目录。你可以使用通过 Microsoft Azure、Office 365 或 Windows Intune 获取的 Azure AD 目录。如果有两个或更多个目录，请参阅[管理 Azure AD 目录](/documentation/articles/active-directory-administer/)来确定要使用哪一个。

## 身份验证

对于支持 SAML 2.0、WS-联合身份验证或 OpenID Connect 协议的应用程序，Azure Active Directory 将使用签名证书来建立信任关系。有关详细信息，请参阅[管理联合单一登录的证书](/documentation/articles/active-directory-sso-certs/)。

对于仅支持基于 HTML 窗体的登录的应用程序，Azure Active Directory 将使用“密码保管库”来建立信任关系。这样，组织中的用户便可以使用 SaaS 应用程序中的用户帐户信息自动登录到 SaaS 应用程序。Azure AD 将收集并安全地存储用户帐户信息和相关密码。有关详细信息，请参阅[基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/#password-based-single-sign-on)。

## 授权

预配的帐户通过单一登录身份验证之后，可授权用户使用应用程序。用户预配可以手动完成，在某些情况下，你可以根据 Azure Active Directory 中所做的更改在 SaaS 应用程序中添加和删除用户信息。有关使用现有 Azure AD 连接器自动化预配的详细信息，请参阅 [SaaS 应用程序的自动化用户预配和取消预配](/documentation/articles/active-directory-saas-app-provisioning/)。

否则，你可以手动将用户信息添加到应用，或使用应用商店中提供的其他预配解决方案。

## Access

Azure AD 提供多种可自定义的方式来向组织中的用户部署应用程序。你不会受限于任一特定的部署或访问解决方案。你可以使用[最符合需要的解决方案](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users)。

## 有关使用中应用程序的其他注意事项

为组织中已在使用的应用程序设置单一登录的过程，与为新应用程序创建新帐户的过程不同。基本步骤包括：将应用程序中的用户标识映射到 Azure AD 标识，以及了解集成之后用户如何体验登录应用程序。

> [AZURE.NOTE] 若要为现有应用程序设置 SSO，你需要有 Azure AD 和 SaaS 应用程序的全局管理员权限。

### 映射用户帐户

用户的标识通常有唯一身份标识符，可能是电子邮件地址或通用个人名称 (UPN)。需要将每个用户的应用程序标识链接（映射）到其各自的 Azure AD 标识。根据应用程序身份验证的要求，有几种方法可实现此目的。

有关映射应用程序标识与 Azure AD 标识的详细信息，请参阅[自定义 SAML 令牌中颁发的声明](http://social.technet.microsoft.com/wiki/contents/articles/31257.azure-active-directory-customizing-claims-issued-in-the-saml-token-for-pre-integrated-apps.aspx)和[为预配自定义属性映射](/documentation/articles/active-directory-saas-customizing-attribute-mappings/)。

### 了解用户的登录体验

为使用中的应用程序集成 SSO 时，请务必认识到用户体验会受到影响。对于所有应用程序，用户将开始使用其 Azure AD 凭据来登录。他们还可能需要使用不同的门户来访问应用程序。

针对某些应用程序的 SSO 可在应用程序本身的登录界面上完成，但对于其他应用程序，用户必须通过中心门户（例如“[我的应用](http://myapps.microsoft.com)”或“[Office365](http://portal.office.com/myapps)”）来登录。请在 [Azure Active Directory 的应用程序访问与单一登录是什么](/documentation/articles/active-directory-appssoaccess-whatis/)中了解有关不同类型的 SSO 及其用户体验的详细信息。

另一个有用的资源是[开发人员指导](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)一文中的隐藏用户许可。

## 后续步骤


对于你可以在应用库中找到的 SaaS 应用，Azure AD 提供了许多[有关如何集成 SaaS 应用的教程](/documentation/articles/active-directory-saas-tutorial-list/)。

如果某个应用不在应用库中，你可以[将该应用作为自定义应用程序添加到 Azure AD 应用库](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)。

Azure.com 库中还提供了有关这些问题的更多详细信息，请先阅读 [Azure Active Directory 的应用程序访问与单一登录是什么](/documentation/articles/active-directory-appssoaccess-whatis/)。

## 另请参阅

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)

<!---HONumber=Mooncake_0620_2016-->