<properties
    pageTitle="Azure Active Directory 开发人员指南 | Azure"
    description="本文提供面向开发人员的 Azure Active Directory 资源的综合性指南。"
    services="active-directory"
    documentationcenter="dev-center-name"
    author="bryanla"
    manager="mbaldwin"
    editor="" />  

<tags
    ms.assetid="5c872c89-ef04-4f4c-98de-bc0c7460c7c2"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="10/24/2016"
    ms.author="mbaldwin" 
    wacn.date="12/09/2016"/>  


# Azure Active Directory 开发人员指南
## 概述
作为标识管理即服务 (IDMaaS) 平台，Azure Active Directory (AD) 为开发人员提供了有效的方法将标识管理功能集成到其应用程序中。以下文章概述了 Azure AD 的实现和主要功能。我们建议你按顺序阅读这些文章。如果你要深入了解，请转到[入门](#getting-started)。

1. [Azure AD 集成的好处](/documentation/articles/active-directory-how-to-integrate/)：了解与 Azure AD 集成为何能够提供最佳的安全登录和授权解决方案。
2. [Azure AD 身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)：利用 Azure AD 中的简化身份验证来登录应用程序。
3. [将应用程序与 Azure AD 集成](/documentation/articles/active-directory-integrating-applications/)：了解如何从 Azure AD 添加、更新和删除应用程序，以及集成应用的品牌准则。
4. [Azure AD Graph API](/documentation/articles/active-directory-graph-api/)：使用 Azure AD Graph API 通过 REST API 终结点以编程方式访问 Azure AD。也可以通过 [Microsoft Graph](https://graph.microsoft.io/) 访问 Azure AD Graph API。Microsoft Graph 提供统一的 API，可以通过单个 REST API 终结点和单个访问令牌访问多个 Microsoft 云服务 API。
5. [Azure AD 身份验证库](/documentation/articles/active-directory-authentication-libraries/)：使用适用于 .NET、JavaScript、Objective-C、Android 及其他语言的 Azure AD 身份验证库，轻松对用户进行身份验证，获取访问令牌。

## 入门 <a name="getting-started"></a>
以下教程是专门针对多种平台编写的，可帮助你快速开始使用 Azure Active Directory 进行开发。作为先决条件，你必须[获取一个 Azure Active Directory 租户](/documentation/articles/active-directory-howto-tenant/)。

### 移动和电脑应用程序快速入门指南
| [![iOS](./media/active-directory-developers-guide/ios.png)](/documentation/articles/active-directory-devquickstarts-ios/) | [![Android](./media/active-directory-developers-guide/android.png)](/documentation/articles/active-directory-devquickstarts-android/) | [![.NET](./media/active-directory-developers-guide/net.png)](/documentation/articles/active-directory-devquickstarts-dotnet/) | [![Windows Universal](./media/active-directory-developers-guide/windows.png)](/documentation/articles/active-directory-devquickstarts-windowsstore/) | [![Xamarin](./media/active-directory-developers-guide/xamarin.png)](/documentation/articles/active-directory-devquickstarts-xamarin/) | [![Cordova](./media/active-directory-developers-guide/cordova.png)](/documentation/articles/active-directory-devquickstarts-cordova/) | [![OAuth 2.0](./media/active-directory-developers-guide/oauth-2.png)](/documentation/articles/active-directory-protocols-oauth-code/) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [iOS](/documentation/articles/active-directory-devquickstarts-ios/) |[Android](/documentation/articles/active-directory-devquickstarts-android/) |[.NET](/documentation/articles/active-directory-devquickstarts-dotnet/) |[Windows Universal](/documentation/articles/active-directory-devquickstarts-windowsstore/) |[Xamarin](/documentation/articles/active-directory-devquickstarts-xamarin/) |[Cordova](/documentation/articles/active-directory-devquickstarts-cordova/) |[直接与 OAuth 2.0 集成](/documentation/articles/active-directory-protocols-oauth-code/) |

### Web 应用程序快速入门指南
| [![.NET](./media/active-directory-developers-guide/net.png)](/documentation/articles/active-directory-devquickstarts-webapp-dotnet/) | [![Java](./media/active-directory-developers-guide/java.png)](/documentation/articles/active-directory-devquickstarts-webapp-java/) | [![AngularJS](./media/active-directory-developers-guide/angularjs.png)](/documentation/articles/active-directory-devquickstarts-angular/) | [![Javascript](./media/active-directory-developers-guide/javascript.png)](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) | [![Node.js](./media/active-directory-developers-guide/nodejs.png)](/documentation/articles/active-directory-devquickstarts-openidconnect-nodejs/) | [![OpenID Connect](./media/active-directory-developers-guide/openid-connect.png)](/documentation/articles/active-directory-protocols-openid-connect-code/) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| [.NET](/documentation/articles/active-directory-devquickstarts-webapp-dotnet/) |[Java](/documentation/articles/active-directory-devquickstarts-webapp-java/) |[AngularJS](/documentation/articles/active-directory-devquickstarts-angular/) |[Javascript](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) |[Node.js](/documentation/articles/active-directory-devquickstarts-openidconnect-nodejs/) |[直接与 OpenID Connect 集成](/documentation/articles/active-directory-protocols-openid-connect-code/) |

### Web API 快速入门指南
| [![.NET](./media/active-directory-developers-guide/net.png)](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/) | [![Node.js](./media/active-directory-developers-guide/nodejs.png)](/documentation/articles/active-directory-devquickstarts-webapi-nodejs/) |
|:---:|:---:|
| [.NET](/documentation/articles/active-directory-devquickstarts-webapi-dotnet/) |[Node.js](/documentation/articles/active-directory-devquickstarts-webapi-nodejs/) |

### 查询目录快速入门指南
| [![.NET](./media/active-directory-developers-guide/graph.png)](/documentation/articles/active-directory-graph-api-quickstart/) |
|:---:|
| [Graph API](/documentation/articles/active-directory-graph-api-quickstart/) |

## 操作方法
以下文章介绍如何使用 Azure Active Directory 执行特定任务：

- [获取 Azure AD 租户](/documentation/articles/active-directory-howto-tenant/)
- [通过多租户应用程序模式使 Azure AD 用户登录](/documentation/articles/active-directory-devhowto-multi-tenant-overview/)
- 使用 ADAL 在 [Android](/documentation/articles/active-directory-sso-android/) 和 [iOS](/documentation/articles/active-directory-sso-ios/) 设备上实现跨应用 SSO
- [针对 Azure AD 认证 AppSource 应用程序](/documentation/articles/active-directory-devhowto-appsource-certified/)
- [列出 Azure AD 应用程序库中的应用程序](/documentation/articles/active-directory-app-gallery-listing/)
- [将适用于 Office 365 的 Web 应用提交到卖家仪表板](https://msdn.microsoft.com/office/office365/howto/submit-web-apps-seller-dashboard)
- [了解 Azure Active Directory 应用程序清单](/documentation/articles/active-directory-application-manifest/)
- [了解客户端应用程序中登录按钮和应用获取按钮的品牌准则](/documentation/articles/active-directory-branding-guidelines/)
- [预览：如何构建可以使用个人帐户和工作或学校帐户来登录用户的应用](/documentation/articles/active-directory-appmodel-v2-overview/)
- [预览：使用 PowerShell 在 Azure AD 中配置令牌生存期](/documentation/articles/active-directory-configurable-token-lifetimes/)。请参阅[策略操作](https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/api/policy-operations)和[策略实体](https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/api/entity-and-complex-type-reference#policy-entity)，了解有关通过 Azure AD 图形 API 进行配置的详细信息。

## 引用
以下文章提供了有关 REST 和身份验证库 API、协议、错误、代码示例和终结点的基础参考信息。

### 支持
- [已标记问题](http://stackoverflow.com/questions/tagged/azure-active-directory)：通过搜索标记 [azure-active-directory](http://stackoverflow.com/questions/tagged/azure-active-directory) 和 [adal](http://stackoverflow.com/questions/tagged/adal) 查找有关堆栈溢出的 Azure Active Directory 解决方案。
- 请参阅 [Azure AD 开发人员术语表](/documentation/articles/active-directory-dev-glossary/)，了解应用程序开发和集成的一些相关常用术语。

### 代码
- [Azure Active Directory 开放源代码库](http://github.com/AzureAD)：查找库源代码的最简单方法是使用我们的[库列表](/documentation/articles/active-directory-authentication-libraries/)。
- [Azure Active Directory 示例](https://github.com/azure-samples?query=active-directory)：浏览示例列表的最简单办法是使用[代码示例的索引](/documentation/articles/active-directory-code-samples/)。
- [用于 .NET 的 Active Directory 身份验证库 (ADAL)](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) - 适用于[最新主要版本](https://docs.microsoft.com/active-directory/adal/microsoft.identitymodel.clients.activedirectory)和[以前主要版本](https://docs.microsoft.com/active-directory/adal/v2/microsoft.identitymodel.clients.activedirectory)的参考文档。

### Graph API
- [Graph API 参考](https://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)：Azure Active Directory Graph API 的 REST 参考。[查看交互式 Graph API 参考体验](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。
- [Graph API 权限范围](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)：OAuth 2.0 权限范围，该权限范围用于控制应用对租户中目录数据的访问权限。

### 身份验证和授权协议
- [Azure AD 中的签名密钥滚动更新](/documentation/articles/active-directory-signing-key-rollover/)：了解 Azure AD 的签名密钥滚动更新频率，以及如何更新最常见应用程序方案的密钥。
- [OAuth 2.0 协议：使用授权代码授权](/documentation/articles/active-directory-protocols-oauth-code/)：可以使用 OAuth 2.0 协议的授权代码授权，授予对 Azure Active Directory 租户中 Web 应用程序和 Web API 的访问权限。
- [OAuth 2.0 协议：了解隐式授权](/documentation/articles/active-directory-dev-understanding-oauth2-implicit-grant/)：了解隐式授权，以及它是否适合自己的应用程序。
- [OAuth 2.0 协议：使用客户端凭据进行服务到服务的调用](/documentation/articles/active-directory-protocols-oauth-service-to-service/)：OAuth 2.0 客户端凭据授权允许 Web 服务（机密客户端）在调用其他 Web 服务时使用自己的凭据进行身份验证，而不是模拟用户。在这种情况下，客户端通常是中间层 Web 服务、后台程序服务或网站。
- [OpenID Connect 1.0 协议：登录和身份验证](/documentation/articles/active-directory-protocols-openid-connect-code/)：OpenID Connect 1.0 协议扩展了 OAuth 2.0，使其能够用作身份验证协议。客户端应用程序可以接收 id\_token 来管理登录过程，或增加授权代码流，同时接收 id\_token 和授权代码。
- [SAML 2.0 协议参考](/documentation/articles/active-directory-saml-protocol-reference/)：SAML 2.0 协议使应用程序能够为其用户提供单一登录体验。
- [WS 联合身份验证 1.2 协议](http://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html)：Azure Active Directory 根据 Web Services 联合身份验证版本 1.2 规范支持 WS 联合身份验证 1.2。若要了解联合元数据文档，请参阅[联合元数据](/documentation/articles/active-directory-federation-metadata/)。
- [支持的令牌和声明类型](/documentation/articles/active-directory-token-and-claims/)：可以通过本指南来了解和评估 SAML 2.0 令牌与 JSON Web 令牌 (JWT) 中的声明。


### 构建
这些概述演示文稿是关于使用本身就在工程团队中工作的 Azure Active Directory 功能发言人来开发应用。演示文稿涵盖了各重要主题，包括 IDMaaS、身份验证、联合身份验证以及单一登录。



## 社交
- [Active Directory 团队博客](http://blogs.technet.com/b/ad/)：Azure Active Directory 领域的最新开发情况。
- [Azure Active Directory Graph 团队博客](http://blogs.msdn.com/b/aadgraphteam)：特定于图形 API 的 Azure Active Directory 信息。
- [云标识](http://www.cloudidentity.com/blog/)：从 Azure Active Directory PM 原理的角度讲解标识管理即服务的设计理念。

## Windows Server 本地开发
有关使用 Windows Server 和 Active Directory 联合身份验证服务 (ADFS) 进行开发的指南，请参阅：

- [面向开发人员的 AD FS 方案](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/overview/ad-fs-scenarios-for-developers)：概述 AD FS 的组件及其工作原理，提供有关支持的身份验证/授权方案的详细信息。
- [AD FS 演练](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/ad-fs-development)：逐步说明如何实施相关身份验证/授权流程的演练文章的列表。

<!---HONumber=Mooncake_1128_2016-->
