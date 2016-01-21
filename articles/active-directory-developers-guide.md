<properties
   pageTitle="Azure Active Directory 开发人员指南"
   description="介绍面向开发人员的 Azure Active Directory 资源的综合性指南"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="10/16/2015"
   wacn.date="01/21/2016"/>


# Azure Active Directory 开发人员指南

## 概述
作为标识管理即服务 (IDMaaS) 平台，Azure Active Directory 为开发人员提供了有效的方法将标识管理功能集成到其应用程序中。以下文章概述了 Azure Active Directory 的实现和主要功能。我们建议你按顺序阅读这些文章。<!-- 如果你要深入了解，请转到[入门](#getting-started)。-->


1. [Azure Active Directory 集成的好处](/documentation/articles/active-directory-how-to-integrate)：了解与 Azure Active Directory 集成为何能够为安全登录和授权提供最佳解决方案。

2. [Active Directory 身份验证方案](/documentation/articles/active-directory-authentication-scenarios)：利用 Azure Active Directory 中的简化身份验证来登录应用程序。
3. [Azure Active Directory 图形 API](/documentation/articles/active-directory-graph-api)：使用 Azure Active Directory 图形 API 通过 REST API 终结点以编程方式访问 Azure AD。

<!-- 3. [将应用程序与 Azure Active Directory 集成](/documentation/articles/active-directory-integrating-applications)：了解如何从 Azure Active Directory 添加、更新和删除应用程序，以及有关集成应用的品牌准则。-->

<!-- 5. [Azure Active Directory 身份验证库](/documentation/articles/active-directory-authentication-libraries)：使用 Azure 身份验证库轻松对用户进行身份验证以获取访问令牌。-->

## 入门

以下教程是专门针对多种平台编写的，可帮助你快速开始使用 Azure Active Directory 进行开发。作为先决条件，你必须[获取一个 Azure Active Directory 租户](/documentation/articles/active-directory-howto-tenant)。
<!--
### 移动和电脑应用程序快速入门指南

|[![iOS](./media/active-directory-developers-guide/ios.png)](/documentation/articles/active-directory-devquickstarts-ios)|[![Android](./media/active-directory-developers-guide/android.png)](active-directory-devquickstarts-android)|[![.NET](./media/active-directory-developers-guide/net.png)](active-directory-devquickstarts-dotnet)| [![Windows Phone](./media/active-directory-developers-guide/windows.png)](active-directory-devquickstarts-windowsphone)|[![Windows 应用商店](./media/active-directory-developers-guide/windows.png)](active-directory-devquickstarts-windowsstore)|[![Xamarin](./media/active-directory-developers-guide/xamarin.png)](active-directory-devquickstarts-xamarin)|[![Cordova](./media/active-directory-developers-guide/cordova.png)](active-directory-devquickstarts-cordova)
|:--:|:--:|:--:|:--:|:--:|:--:|:--:
|[iOS](active-directory-devquickstarts-ios)|[Android](/documentation/articles/active-directory-devquickstarts-android)|[.NET](active-directory-devquickstarts-dotnet)|[Windows Phone](active-directory-devquickstarts-windowsphone)|[Windows 应用商店](active-directory-devquickstarts-windowsstore)|[Xamarin](active-directory-devquickstarts-xamarin)|[Cordova](active-directory-devquickstarts-cordova)

### WEB 应用快速入门指南

|[![.NET](./media/active-directory-developers-guide/net.png)](/documentation/articles/active-directory-devquickstarts-webapp-dotnet)|[![Javascript](./media/active-directory-developers-guide/javascript.png)](active-directory-devquickstarts-angular)|[![Node.js](./media/active-directory-developers-guide/nodejs.png)](active-directory-devquickstarts-openidconnect-nodejs)
|:--:|:--:|:--:|
|[.NET](/documentation/articles/active-directory-devquickstarts-webapp-dotnet)|[Javascript](/documentation/articles/active-directory-devquickstarts-angular)|[Node.js](active-directory-devquickstarts-openidconnect-nodejs)

### Web API 快速入门指南

|[![.NET](./media/active-directory-developers-guide/net.png)](/documentation/articles/active-directory-devquickstarts-webapi-dotnet)|[![Node.js](./media/active-directory-developers-guide/nodejs.png)](active-directory-devquickstarts-webapi-nodejs)
|:--:|:--:|
|[.NET](/documentation/articles/active-directory-devquickstarts-webapi-dotnet)|[Node.js](/documentation/articles/active-directory-devquickstarts-webapi-nodejs)

### 查询目录快速入门指南

| [![.NET](./media/active-directory-developers-guide/graph.png)](/documentation/articles/active-directory-graph-api-quickstart)|
|:--:|
|[Graph API](/documentation/articles/active-directory-graph-api-quickstart)|

-->
## 操作说明

以下文章介绍如何使用 Azure Active Directory (AD) 执行具体的任务。

- [获取 Azure Active Directory 租户](/documentation/articles/active-directory-howto-tenant)
- [使用 Office 365 API 创建应用](https://msdn.microsoft.com/office/office365/howto/getting-started-Office-365-APIs)
- [将适用于 Office 365 的 WEB 应用提交到卖家仪表板](https://msdn.microsoft.com/office/office365/howto/submit-web-apps-seller-dashboard)

## 引用

以下文章提供了有关 REST 和身份验证库 API、协议、错误、代码示例和终结点的基础参考信息。

### 支持
- [已标记问题](http://stackoverflow.com/questions/tagged/azure-active-directory)：通过搜索标记 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 和 [adal](http://stackoverflow.com/questions/tagged/adal) 查找有关堆栈溢出的 Azure Active Directory 解决方案。

### 代码

- [Azure Active Directory 开放源代码库](http://github.com/AzureAD)：查找库源代码的最简单方法是使用我们的[库列表](/documentation/articles/active-directory-authentication-libraries)。

- [Azure Active Directory 示例](http://github.com/AzureADSamples)：浏览示例列表的最简单办法是使用[代码示例的索引](/documentation/articles/active-directory-code-samples)。


#### Graph API

- [Graph API 参考](https://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)：Azure Active Directory Graph API 的 REST 参考。[查看新的交互式 Graph API 参考体验](https://msdn.microsoft.com/zh-cn/library/Azure/Ad/Graph/api/api-catalog)。

- [Graph API 权限作用域](https://msdn.microsoft.com/zh-cn/library/Azure/Ad/Graph/api/graph-api-permission-scopes)：用于控制应用程序必须对租户中目录数据具有的访问权限的 OAuth 2.0 权限作用域。


#### 身份验证协议

- [SAML 2.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn195591.aspx)：SAML 2.0 协议使应用程序能够为其用户提供单一登录体验。


- [OAuth 2.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn645545.aspx)：OAuth 2.0 协议使你能够授权访问 Azure AD 租户中的 WEB 应用和 Web API。


- [OpenID Connect 1.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn645541.aspx)：OpenID Connect 1.0 协议扩展了 OAuth 2.0，使其能够用作身份验证协议。


- [WS-Federation 1.2 协议参考](http://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html)：Web 服务联合身份验证版本 1.2 规范中指定的 WS-Federation 1.2 协议。

- [支持的安全令牌和声明](/documentation/articles/active-directory-token-and-claims)：该指南可帮助你了解和评估 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 令牌中的声明。

## 社交

- [Active Directory 团队博客](http://blogs.technet.com/b/ad/)：实时了解 Azure AD 领域的最新开发情况。

- [Azure Active Directory Graph 团队博客](http://blogs.msdn.com/b/aadgraphteam)：特定于图形 API 的 Azure Active Directory 信息。

- [云标识](http://www.cloudidentity.net)：从 Azure Active Directory PM 原理的角度讲解标识管理即服务的设计理念。

<!---HONumber=60-->