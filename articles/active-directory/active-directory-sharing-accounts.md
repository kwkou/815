<properties
    pageTitle="使用 Azure AD 共享帐户 | Azure"
    description="介绍组织如何使用 Azure Active Directory 来安全共享本地应用和使用者云服务的帐户。"
    services="active-directory"
    documentationcenter=""
    author="curtand"
    manager="femila"
    editor="" />
<tags
    ms.assetid="e2d77104-d978-46a3-bfea-03ffdf3b61e6"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/04/2017"
    ms.author="curtand"
    wacn.date="06/12/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="c37c95515c3f570d91855854b4ad06cc425be0cd"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="sharing-accounts-with-azure-ad"></a>使用 Azure AD 共享帐户
## <a name="overview"></a>概述
组织有时需要针对多人使用单个用户名和密码。 这通常发生在两个情况下：

- 所有用户必须使用唯一的登录名和密码才能访问应用程序时（无论是本地应用还是使用者云服务，例如公司的社交媒体帐户）。
- 创建多用户环境时。 可能有一个具有提升权限的本地帐户，并且该帐户可用来执行核心安装、管理和恢复活动（例如 Office 365 的本地“全局管理员”帐户或 Salesforce 中的 root 帐户）。

传统上，这些帐户的共享方式是将凭据（用户名/密码）分发给适当的人员，或者将凭据存储在多个受信任代理可以访问的共享位置。

传统的共享模式有几个缺点：

- 要使所有人都能访问新的应用程序，需要向他们分发凭据。
- 每个共享的应用程序可能都需要唯一的一组共享凭据，从而要求用户记住多组凭据。 当用户必须记住许多凭据时，他们会采取有风险的做法，风险就会随之增加。 （例如写下密码）。
- 你不知道谁有权访问应用程序。
- 你不知道谁 *访问了* 应用程序。
- 当你需要删除某个应用程序的访问权限时，必须更新凭据，并将凭据重新分发给需要访问该应用程序的所有人。

## <a name="azure-active-directory-account-sharing"></a>Azure Active Directory 帐户共享
Azure AD 提供使用共享帐户的新方法，从而可以消除这些缺点。

通过使用访问面板并选择最适合该应用程序的单一登录类型，Azure AD 管理员可以配置用户可访问的应用程序。 在这些类型中， *基于密码的单一登录*可在登录该应用的过程中，让 Azure AD 充当某种“代理”。

用户使用他们的组织帐户登录一次即可。 这与他们平时用来访问桌面或电子邮件的帐户相同。 他们只能发现和访问分配给他们的那些应用程序。 使用共享帐户时，此应用程序列表可以包含任意数目的共享凭据。 最终用户不需要记住或写下他们可能要使用的多个帐户。

共享帐户不仅提高了监管力度和可用性，也增强了安全性。 有权使用凭据的用户看不到共享密码，而是通过协调的身份验证流程获取密码的使用权限。 此外，使用某些密码 SSO 应用程序时，可以选择让 Azure AD 定期使用复杂的长密码来轮换（更新）密码，以提升帐户安全性。 管理员可以轻松授予或吊销对应用程序的访问权限，还知道谁有权访问帐户以及谁曾经访问了帐户。

Azure AD 支持任何企业移动性套件 (EMS)、高级或基本许可用户的共享帐户，包括所有类型的密码单一登录应用程序。 可以共享应用程序库中数千个预先集成的应用程序的帐户，并可使用[自定义 SSO 应用](/documentation/articles/active-directory-sso-integrate-saas-apps/)添加自己的密码身份验证应用程序。

支持帐户共享的 Azure AD 功能包括：

- [密码单一登录](/documentation/articles/active-directory-appssoaccess-whatis/#password-based-single-sign-on/)
- 密码单一登录代理
- 自定义密码应用
- 最终用户访问门户
- [Active Directory 应用商店](https://azure.microsoft.com/marketplace/active-directory/all/)

## <a name="sharing-an-account"></a>共享帐户
若要使用 Azure AD 来共享帐户，你需要：

- 添加应用程序[应用库](https://azure.microsoft.com/marketplace/active-directory/)或[自定义应用程序](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)
- 将应用程序配置为使用密码单一登录 (SSO)
- 可选：在某些应用程序（例如 LinkedIn）中，你可以启用 [Azure AD 自动轮换密码](http://blogs.technet.com/b/ad/archive/2015/02/20/azure-ad-automated-password-roll-over-for-facebook-twitter-and-linkedin-now-in-preview.aspx)

还可以使用多重身份验证 (MFA) 更安全地生成共享帐户（了解有关[使用 Azure AD 保护应用程序](/documentation/articles/multi-factor-authentication-get-started-cloud/)的详细信息）。

## <a name="related-articles"></a>相关文章

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)

<!--Update_Description: wording update -->