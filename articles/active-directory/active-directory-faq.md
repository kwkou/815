<properties
    pageTitle="Azure Active Directory 常见问题 | Azure"
    description="Azure Active Directory 常见问题，其中提供了有关访问 Azure 和 Azure Active Directory、密码管理和应用程序访问的问题的解答。"
    services="active-directory"
    documentationcenter=""
    author="MarkusVi"
    manager="femila"
    editor="" />  

<tags
    ms.assetid="b8207760-9714-4871-93d5-f9893de31c8f"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="10/31/2016"
    ms.author="markusvi" 
    wacn.date="12/09/2016"/>  


# Azure Active Directory 常见问题
Azure Active Directory 是综合性的标识即服务 (IDaaS) 解决方案，其涉及到标识、访问管理和安全的方方面面。

有关详细信息，请参阅 [What is Azure Active Directory?](/documentation/articles/active-directory-whatis/)（什么是 Azure Active Directory？）。

## 访问 Azure 和 Azure Active Directory
**问：尝试在 Azure 经典管理门户 (https://manage.windowsazure.cn) 中访问 Azure AD 时，为何收到“找不到订阅”？**

**答：**若要访问 Azure 经典管理门户，每位用户都必须拥有 Azure 订阅的权限。如果使用付费版 Office 365 或 Azure AD，请导航到 [http://aka.ms/accessAAD](http://aka.ms/accessAAD) 完成一次性激活步骤，否则需要激活完整的 [Azure 试用版](/pricing/1rmb-trial/)或付费版订阅。

有关详细信息，请参阅：

- [Azure 订阅与 Azure Active Directory 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory/)
- [在 Azure 中管理 Office 365 订阅的目录](/documentation/articles/active-directory-manage-o365-subscription/)

- - -
**问：Azure AD、Office 365 与 Azure 之间有哪种关系？**

**答：**Azure Active Directory 为所有 Microsoft 在线服务提供通用的标识和访问功能。无论使用的是 Office 365、Azure、Intune 还是其他服务，都可以使用 Azure AD 来为上述所有服务启用登录和访问管理。

事实上，为 Microsoft 在线服务启用的所有用户都定义为一个或多个 Azure AD 实例中的用户帐户。你可以为这些帐户启用免费的 Azure AD 功能，例如云应用程序访问。

此外，Azure AD 付费版服务（例如，Azure AD Basic、Premium、EMS 等）可通过综合性的企业级管理和安全解决方案来弥补其他在线服务（例如 Office 365 和 Azure）的不足。

- - -
## 混合 Azure AD 入门
**问：如何将我的本地目录连接到 Azure AD？**

**答：**可以使用 **Azure AD Connect** 将本地目录连接到 Azure AD。

有关详细信息，请参阅 [Integrating your on-premises identities with Azure Active Directory](/documentation/articles/active-directory-aadconnect/)（将本地标识与 Azure Active Directory 集成）。

- - -
**问：如何设置本地目录与云应用程序之间的 SSO？**

**答：**只需在本地目录与 Azure AD 之间设置 SSO。只要通过 Azure AD 访问云应用程序，该服务就会自动让用户使用其本地凭据正确进行身份验证。

通过联合身份验证解决方案（例如 ADFS）或通过配置密码哈希同步，就能轻松地从本地实现 SSO。可以使用 Azure AD Connect 配置向导轻松部署这两个选项。

有关详细信息，请参阅 [Integrating your on-premises identities with Azure Active Directory](/documentation/articles/active-directory-aadconnect/)（将本地标识与 Azure Active Directory 集成）。

- - -
**问：Azure Active Directory 是否为组织中的用户提供自助服务门户？**

**答：**是的，Azure Active Directory 提供 [Azure AD 访问面板](http://myapps.microsoft.com)，方便用户进行自助服务和应用程序访问。如果你是 Office 365 客户，可以在 Office 365 门户中找到许多相同的功能。

## 密码管理
**问：是否可以使用 Azure AD 密码写回但不使用密码同步？ （也就是说，我想要使用 Azure AD SSPR 和密码写回，但不想将密码存储在云中。）**

**答：**无需将 AD 密码同步到 Azure AD 即可启用写回。在联合环境中，Azure AD SSO 依赖本地目录来对用户进行身份验证。在这种情况下，并不需要在 Azure AD 中跟踪本地密码。

- - -
**问：需要多长时间才能将密码写回到 AD 本地？**

**答：**密码写回实时运行。

有关详细信息，请参阅 [Getting started with Password Management](/documentation/articles/active-directory-passwords-getting-started/)（密码管理入门）。

- - -
**问：是否可以对管理员管理的密码使用密码写回？**

**答：**可以。如果已启用密码写回，管理员执行的密码操作将写回到你的本地环境。

有关密码相关问题的详细解答，请参阅 [Password Management Frequently Asked Questions](/documentation/articles/active-directory-passwords-faq/)（密码管理常见问题）。

- - -
## 应用程序访问
**问：在哪里可以找到与 Azure AD 预先集成的应用程序及其功能的列表？**

**答：**Azure AD 中包含 Microsoft、应用程序服务提供商或第三方提供的 2600 多个预先集成的应用程序。所有预先集成的应用程序都支持 SSO。SSO 允许你使用组织凭据来访问应用。某些应用程序还支持自动预配和自动取消预配

有关预先集成的应用程序的完整列表，请参阅 [Active Directory Marketplace](https://azure.microsoft.com/marketplace/active-directory/)（Active Directory 应用商店）。

- - -
**问：如果 Azure AD 应用商店中没有我需要的应用程序怎么办？**

**答：**使用 Azure AD Premium，可以添加和配置所需的任何应用程序。你可以根据应用程序的功能和你的喜好来配置 SSO 和自动预配。

有关详细信息，请参阅：

- [使用 SCIM 启用从 Azure Active Directory 到应用程序的用户和组自动预配](/documentation/articles/active-directory-scim-provisioning/)

- - -
**问：用户如何使用 Azure Active Directory 来登录应用程序？**

**答：**Azure Active Directory 提供多种方式供用户查看和访问其应用程序，例如：

- Azure AD 访问面板
- Office 365 应用程序启动器
- 直接登录联合应用
- 联合、基于密码或现有的应用的深层链接

有关详细信息，请参阅[为用户部署 Azure AD 集成的应用程序](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users/)。

- - -
**问：Azure Active Directory 可通过哪些不同的方式来启用对应用程序的身份验证和单一登录？**

**答：**Azure Active Directory 支持使用多种标准化协议（例如 SAML 2.0、OpenID Connect、OAuth 2.0 和 WS-Federation）进行身份验证和授权。对于仅支持基于窗体的身份验证的应用程序，Azure AD 还支持密码保管和自动化登录功能。

有关详细信息，请参阅：

- [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)
- [Active Directory 身份验证协议](/documentation/articles/active-directory-developers-guide/)
- [Azure Active Directory 中单一登录的工作原理是什么？](/documentation/articles/active-directory-appssoaccess-whatis/#how-does-single-sign-on-with-azure-active-directory-work/)

<!---HONumber=Mooncake_1128_2016-->
