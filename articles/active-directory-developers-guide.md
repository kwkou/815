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
   ms.date="05/08/2015"
   wacn.date="06/16/2015"/>


# Azure Active Directory 开发人员指南

## 概述
作为标识管理即服务 (IDMaaS) 平台，Azure Active Directory 为开发人员提供了有效的方法将标识管理功能集成到其应用程序中。以下文章概述了 Azure Active Directory 的实现和主要功能。我们建议你按顺序阅读这些文章。如果你要深入了解，请转到[入门](#getting-started)。


1. **[如何与 Azure AD 集成](/documentation/articles/active-directory-how-to-integrate)**：了解与 Azure Active Directory 集成为何能够为安全登录和授权提供最佳解决方案。

2. **[使用 Azure AD 登录](/documentation/articles/active-directory-authentication-scenarios)**：利用 Azure Active Directory 的简化身份验证来登录应用程序。

3. **[查询目录](https://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)**：使用 Azure Active Directory Graph API 通过 REST API 终结点以编程方式访问 Azure AD。

4. **[了解应用程序模型](https://msdn.microsoft.com/zh-cn/library/azure/dn151122.aspx)**：了解如何注册应用程序，以及多租户应用程序的品牌准则。

5. **[库](https://msdn.microsoft.com/zh-cn/library/azure/dn151135.aspx)**：使用 Azure 身份验证库轻松对用户进行身份验证以获取访问令牌。

## 入门

以下教程是专门针对多种平台编写的，可帮助你快速开始使用 Azure Active Directory 进行开发。作为先决条件，你必须[获取一个 Azure Active Directory 租户](/documentation/articles/active-directory-howto-tenant)。

#### 移动或 PC 应用程序快速入门指南

- [iOS](/documentation/articles/active-directory-devquickstarts-ios)
- [Android](/documentation/articles/active-directory-devquickstarts-android)
- [.NET](/documentation/articles/active-directory-devquickstarts-dotnet)
- [Windows Phone](/documentation/articles/active-directory-devquickstarts-windowsphone)
- [Windows 应用商店](/documentation/articles/active-directory-devquickstarts-windowsstore)
- [Xamarin](/documentation/articles/active-directory-devquickstarts-xamarin)
- [Cordova](/documentation/articles/active-directory-devquickstarts-cordova)


#### Web 应用程序或 Web API 快速入门指南

- [.NET Web 应用](/documentation/articles/active-directory-devquickstarts-webapp-dotnet)
- [.NET Web API](/documentation/articles/active-directory-devquickstarts-webapi-dotnet)
- [Javascript](/documentation/articles/active-directory-devquickstarts-angular)
- [Node.js](/documentation/articles/active-directory-devquickstarts-webapi-nodejs)


## 操作说明

以下文章介绍如何使用 Azure Active Directory (AD) 执行具体的任务。

- [如何获取 Azure AD 租户](/documentation/articles/active-directory-howto-tenant)
- [如何列出 Azure AD 应用程序库中的应用程序](/documentation/articles/active-directory-app-gallery-listing)
- [如何在应用程序中开始使用 Office 365 API](https://msdn.microsoft.com/office/office365/howto/getting-started-Office-365-APIs)


## 引用

以下文章提供了有关 REST 和身份验证库 API、协议、错误、代码示例和终结点的基础参考信息。

####  支持
- **[从何处获得支持](http://stackoverflow.com/questions/tagged/azure-active-directory)**：通过搜索标记 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 和 [adal](http://stackoverflow.com/questions/tagged/adal) 查找有关堆栈溢出的 Azure AD 解决方案。

#### 代码

- **[Azure AD 开放源代码库](http://github.com/AzureAD)**：查找库源代码的最简单方法是使用我们的[库列表](https://msdn.microsoft.com/zh-cn/library/azure/dn151135.aspx)。

- **[Azure AD 示例](http://github.com/AzureADSamples)**：浏览示例列表的最简单办法是使用[代码示例索引](/documentation/articles/active-directory-code-samples)。


#### Graph API

- **[Graph API 参考](https://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)**：Azure Active Directory Graph API 的 REST 参考。[查看新的交互式 Graph API 参考体验](https://msdn.microsoft.com/zh-cn/library/Azure/Ad/Graph/api/api-catalog)。

- **[Graph API 权限作用域](https://msdn.microsoft.com/zh-cn/library/Azure/Ad/Graph/api/graph-api-permission-scopes)**：用于控制应用程序必须对租户中目录数据具有的访问权限的 OAuth 2.0 权限作用域。


#### 身份验证协议

- **[SAML 2.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn195591.aspx)**：SAML 2.0 协议使应用程序能够为其用户提供单一登录体验。


- **[OAuth 2.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn645545.aspx)**：OAuth 2.0 协议使你能够授权访问 Azure AD 租户中的 Web 应用程序和 Web API。


- **[OpenID Connect 1.0 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn645541.aspx)**：OpenID Connect 1.0 协议扩展了 OAuth 2.0，使其能够用作身份验证协议。


- **[WS-Federation 1.2 协议参考](https://msdn.microsoft.com/zh-cn/library/azure/dn903702.aspx)**：Web 服务联合身份验证版本 1.2 规范中指定的 WS-Federation 1.2 协议。

- **[支持的安全令牌和声明](/documentation/articles/active-directory-token-and-claims)**：该指南可帮助你了解和评估 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 令牌中的声明。

## 社交

- **[Active Directory 团队博客](http://blogs.technet.com/b/ad/)**：实时了解 Azure AD 领域的最新开发情况。

- **[Azure AD Graph 博客](http://blogs.msdn.com/b/aadgraphteam)**：特定于图形 API 的 Azure AD 信息。

- **[云标识](http://www.cloudidentity.net)**：从 Azure Active Directory PM 原理的角度讲解标识管理即服务的设计理念。

<!---HONumber=60-->