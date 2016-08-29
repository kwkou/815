<properties
   pageTitle="如何使 AppSource 通过 Azure Active Directory 的认证 | Microsoft Azure"
   description="详细说明如何使应用程序 AppSource 通过 Azure Active Directory 的认证。"
   services="active-directory"
   documentationCenter=""
   authors="skwan"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="07/05/2016"
   wacn.date="08/29/2016"/>  


#如何使 AppSource 通过 Azure Active Directory (AD) 的认证 

若要使 AppSource 通过 Azure AD 的认证，你的应用程序必须使用 OpenID Connect、OAuth 2.0 或 SAML 2.0 协议在 Azure AD 中实现多租户登录模式。

如果你不熟悉 Azure AD 登录或多租户应用程序开发：

1. 请先阅读有关[在 Azure AD 身份验证方案中使用浏览器到 Web 应用的方案][AAD-Auth-Scenarios-Browser-To-WebApp]的文章。
2. 接下来，请阅读 Azure AD [Web 应用程序快速入门指南][AAD-QuickStart-Web-Apps]，其中演示了如何实现登录，并随附了代码示例。
3. 若要了解如何在 Azure AD 中实现多租户登录模式，请查看 [How to sign in any Azure Active Directory (AD) user using the multi-tenant application pattern][AAD-Howto-Multitenant-Overview]（如何使用多租户应用程序模式将任何 Azure Active Directory (AD) 用户登录）。

## 相关内容
有关构建支持 Azure AD 登录的应用程序的详细信息，或要获得帮助和支持，请参阅 [Azure AD Developer's Guide][AAD-Dev-Guide]（Azure AD 开发人员指南）。

欢迎使用以下 Disqus 意见部分提供反馈，帮助我们改进与制作内容。

<!--Reference style links -->
[AAD-Auth-Scenarios]: /documentation/articles/active-directory-authentication-scenarios/
[AAD-Auth-Scenarios-Browser-To-WebApp]: /documentation/articles/active-directory-authentication-scenarios/#web-browser-to-web-application
[AAD-Dev-Guide]: /documentation/articles/active-directory-developers-guide/
[AAD-Howto-Multitenant-Overview]: /documentation/articles/active-directory-devhowto-multi-tenant-overview/
[AAD-QuickStart-Web-Apps]: /documentation/articles/active-directory-developers-guide/#web-application-quick-start-guides


<!--Image references-->

<!---HONumber=Mooncake_0822_2016-->