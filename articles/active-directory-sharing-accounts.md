<properties
	pageTitle="使用 Azure AD 共享帐户 | Azure"
	description="介绍组织如何使用 Azure Active Directory 来安全共享本地应用和使用者云服务的帐户。"
	services="active-directory"
	documentationCenter=""
	authors="msStevenPo"
 	manager="stevenpo"
	editor=""/>

 <tags
	ms.service="active-directory"
 	ms.date="02/09/2016"  
 	wacn.date="06/27/2016"/>

# 使用 Azure AD 共享帐户

## 概述
组织有时需要针对多人使用单个用户名和密码。这通常发生在两个情况下：

- 每个用户必须使用唯一的登录名和密码访问应用程序时（无论是本地应用还是使用者云服务，例如公司的社交媒体帐户）。
- 创建多用户环境时。你可能有一个具有提升权限的本地帐户，并且该帐户可用来执行核心安装、管理和恢复活动（例如 Office 365 的本地“全局管理员”帐户或 Salesforce 中的 root 帐户）。

传统上，这些帐户的共享方式是通过将凭据（用户名/密码）分发给适当的人员，或者将凭据存储在多个受信任代理可以访问的共享位置。

传统的共享模式有几个缺点：

- 要使所有人都能访问新的应用程序，需要向他们分发凭据。
- 每个共享的应用程序可能都需要唯一的一组共享凭据，而用户必须记住多组凭据。如果用户必须记住许多凭据，他们将采取有风险的做法（例如写下密码），因此增加了风险。
- 你不知道谁有权访问应用程序。
- 你不知道谁访问了应用程序。
- 当你需要删除某个应用程序的访问权限时，必须更新凭据，并将凭据重新分发给需要访问该应用程序的所有人。

## Azure Active Directory 帐户共享

Azure AD 提供使用共享帐户的新方法，从而可以消除这些缺点。

通过使用访问面板并选择最适合该应用程序的单一登录类型，Azure AD 管理员可以配置用户可访问的应用程序。在这些类型中，基于密码的单一登录可在登录该应用的过程中，让 Azure AD 充当某种“代理”。

用户使用他们的组织帐户登录一次即可。这与他们平时用来访问桌面或电子邮件的帐户相同。他们只能发现和访问分配给他们的那些应用程序。使用共享帐户时，此应用程序列表可以包含任意数目的共享凭据。最终用户不需要记住或写下他们可能要使用的多个帐户。

共享帐户不仅提高了监管力度和可用性，也增强了安全性。有权使用凭据的用户看不到共享密码，而是通过协调的身份验证流程获取密码的使用权限。此外，使用某些密码 SSO 应用程序时，你可以选择让 Azure AD 定期使用复杂的长密码来轮换（更新）密码，以提升帐户安全性。管理员可以轻松授予或吊销对应用程序的访问权限，还知道谁有权访问帐户以及谁曾经访问了帐户。

Azure AD 支持任何 Enterprise Mobility Suite (EMS)、高级或基本许可用户的共享帐户，包括所有类型的密码单一登录应用程序。你可以共享应用库中数千个预先集成的应用程序的帐户，并可以使用[自定义 SSO 应用](/documentation/articles/active-directory-single-sign-on-newly-acquired-saas-apps/)添加你自己的密码身份验证应用程序。

支持帐户共享的 Azure AD 功能包括：

- [密码单一登录](/documentation/articles/active-directory-appssoaccess-whatis.md/#password-based-single-sign-on)
- 密码单一登录代理
- [组分配](/documentation/articles/active-directory-accessmanagement-self-service-group-management/)
- 自定义密码应用
- [应用使用情况仪表板/报告](/documentation/articles/active-directory-passwords-get-insights/)
- 最终用户访问门户


## 共享帐户
若要使用 Azure AD 来共享帐户，你需要：

- 将应用程序配置为使用密码单一登录 (SSO)
- 使用基于组的分配，并选择输入共享凭据的选项
- 可选：在某些应用程序（例如 Facebook、Twitter 或 LinkedIn）中，你可以启用 [Azure AD 自动轮换密码](http://blogs.technet.com/b/ad/archive/2015/02/20/azure-ad-automated-password-roll-over-for-facebook-twitter-and-linkedin-now-in-preview.aspx)的选项

你还可以使用 Multi-Factor Authentication (MFA) 提高共享帐户的安全性（了解有关[使用 Azure AD 保护应用程序](/documentation/articles/multi-factor-authentication-get-started/)的信息），并可以使用 [Azure AD 自助服务](/documentation/articles/active-directory-accessmanagement-self-service-group-management/)组管理来委派有关谁有权访问应用程序的管理权。

## 相关文章

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)


<!---HONumber=Mooncake_0620_2016-->