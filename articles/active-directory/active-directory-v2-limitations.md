<properties
	pageTitle="v2.0 终结点的限制和局限性 | Azure"
	description="Azure AD v2.0 终结点的限制和局限性列表。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>  


<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/30/2016"
	wacn.date="11/08/2016"
	ms.author="dastrock"/>

# 我是否应使用 v2.0 终结点？

创建与 Azure Active Directory 集成的应用程序时，需要确定 v2.0 终结点和身份验证协议是否满足需求。原始 Azure AD 终结点仍完全受到支持，并且在某些方面比 v2.0 的功能更丰富。但是，v2.0 终结点为开发人员[带来了明显的优势](/documentation/articles/active-directory-v2-compare/)，使你更乐于使用新的编程模型。

当前，我们对使用 v2.0 终结点的建议如下：

- 如果要在应用程序中支持个人 Microsoft 帐户，则应使用 v2.0 终结点。但是务必要了解本文中列出的限制，尤其是关于工作帐户和学校帐户的限制。
- 如果应用程序仅需要支持工作和学校帐户，则应使用[原始的 Azure AD 终结点](/documentation/articles/active-directory-developers-guide/)。

随着时间推移，v2.0 终结点将会逐步移除此处列出的限制，因此你只需要使用 v2.0 终结点。在此同时，本文旨在帮助你判断 v2.0 终结点是否对你适用。我们将持续更新本文，以反映 v2.0 终结点的当前状态，因此请不时返回查阅本文以重新评估 v2.0 功能是否符合你的要求。

如果你的某个采用 Azure AD 的现有应用未使用 v2.0 终结点，你不需要从头开始进行配置。将来，我们会提供一种方法使现有的 Azure AD 应用程序可以使用 v2.0 终结点。

## 应用限制
v2.0 终结点目前不支持以下类型的应用。有关受支持应用类型的说明，请参阅[此文](/documentation/articles/active-directory-v2-flows/)。

##### 独立 Web API
在 v2.0 终结点中，你可以[构建使用 OAuth 2.0 保护的 Web API](/documentation/articles/active-directory-v2-flows/#web-apis/)。但是，该 Web API 只能从共享相同应用程序 ID 的应用程序接收令牌。不支持构建从具有不同应用程序 ID 的客户端访问的 Web API。该客户端无法请求或获取对 Web API 的权限。

若要了解如何构建从具有相同应用 ID 的客户端接受令牌的 Web API，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)中的 v2.0 终结点 Web API 示例。

##### Web API 代理流
许多体系结构包含需要调用另一个下游 Web API 的 Web API，这两者都受 v2.0 终结点的保护。此情况常见于具有 Web API 后端的本机客户端，该后端反过来调用 Microsoft Online 服务或另一个支持 Azure AD 的自定义构建的 Web API。

使用 OAuth 2.0 Jwt 持有者凭据授权（也称为“代理流”）可支持此方案。但是，v2.0 终结点目前不支持代理流。若要查看此流在正式版 Azure AD 服务中如何工作，请参阅 [GitHub 上的代理代码示例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

## 应用注册限制
目前，所有想要与 v2.0 终结点集成的应用都必须在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList) 上创建新的应用注册。任何现有的 Azure AD 或 Microsoft 帐户应用将不会与 v2.0 终结点兼容，并且无法在任何门户（包括新的应用注册门户）中注册应用。我们计划提供一种方法，使现有应用程序用作 v2.0 应用程序。但是目前没有使应用迁移到 v2.0 终结点的迁移路径。

同样地，在新的应用注册门户中注册的应用将无法配合原有 Azure AD 身份验证终结点来运行。但是，你可以使用在应用注册门户中创建的应用，成功地与 Microsoft 帐户身份验证终结点 `https://login.live.com` 集成。

此外，在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList) 创建的应用程序注册具有以下注意事项：

- 不支持**主页**属性（也称为**登录 URL**）。在没有主页的情况下，这些应用程序不会显示在 Office MyApps 面板中。
- 当前每个应用程序 Id 只允许有两个应用程序密码。
- 应用程序注册只能由一个开发人员帐户查看和管理，不能在多个开发人员之间共享。
- 允许的 redirect\_uri 的格式存在一些限制。请参阅以下部分了解详细信息。

## 重定向 URI 的限制
在新的应用程序注册门户中注册的应用目前限制为一组有限的 redirect\_uri 值。Web 应用和服务的 redirect\_uri 必须以协议 `https` 开头，并且所有 redirect\_uri 值必须共享一个 DNS 域。例如，不能注册具有 redirect\_uris 的 Web 应用程序：

`https://login-east.contoso.com`  
`https://login-west.contoso.com`

注册系统将现有 redirect\_uri 的完整 DNS 名称与要添加的 redirect\_uri 的 DNS 名称相比较。如果满足以下任一条件，添加 DNS 名称的请求将失败：

- 新的 redirect\_uri 的完整 DNS 名称与现有的 redirect\_uri 的 DNS 名称不匹配
- 新的 redirect\_uri 的完整 DNS 名称不是现有的 redirect\_uri 的子域

例如，如果应用当前拥有以下 redirect\_uri：

`https://login.contoso.com`  


则可以添加：

`https://login.contoso.com/new`

因为它与 DNS 名称完全匹配，或：

`https://new.login.contoso.com`  


因为它是 login.contoso.com 的 DNS 子域。如果你希望拥有使用 login-east.contoso.com 和 login-west.contoso.com 作为 redirect\_uris 的应用，则必须按顺序添加以下 redirect\_uris：

`https://contoso.com` 
`https://login-east.contoso.com` 
`https://login-west.contoso.com`

可以添加后两个 redirect\_uri，因为它们是第一个 (contoso.com) 的子域。即将发布的版本中将取消此限制。

若要了解如何在新的应用程序注册门户中注册应用，请参阅[此文](/documentation/articles/active-directory-v2-app-registration/)。

## 服务和 API 限制
v2.0 终结点目前支持登录所有已在新应用程序注册门户中注册的应用，前提是该应用已在[支持的身份验证流](/documentation/articles/active-directory-v2-flows/)列表中列出。但是，这些应用只能获取 OAuth 2.0 访问令牌来访问非常有限的资源集。v2.0 终结点只为以下项目颁发 access\_token：

- 请求令牌的应用。如果逻辑应用包含多个不同的组件或层，则应用可为自身获取 access\_token。若要查看此方案的工作方式，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)教程。
- Outlook 邮件、日历和联系人 REST API，全都位于 https://outlook.office.com。若要了解如何编写访问这些 API 的应用，请参阅 [Office 入门](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)教程。
- Microsoft 图形 API。若要了解 Microsoft Graph 和可用的所有数据，请访问 [https://graph.microsoft.io](https://graph.microsoft.io)。

目前不支持其他服务。将来会添加更多的 Microsoft Online 服务，并支持自定义构建的 Web API 和服务。

## 库和 SDK 限制
当前，对 v2.0 终结点的库支持非常有限。如果你想要在生产应用程序中使用 v2.0 终结点，可使用以下选项：

- 如果你要构建 Web 应用程序，可以放心使用我们的正式版服务器端中间件来执行登录和令牌验证。其中包括适用于 ASP.NET 的 OWIN Open ID Connect 中间件和 NodeJS Passport 插件。[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分中也提供了有关使用这些中间件的代码示例。
- 对于其他平台以及本机与移动应用程序，你还可以通过直接在应用程序代码中发送和接收协议消息来与 v2.0 终结点集成。v2.0 OpenID Connect 和 OAuth 协议[有明确的说明文档](/documentation/articles/active-directory-v2-protocols/)，可帮助你执行这种集成。
- 最后，你可以使用开源 Open ID Connect 和 OAuth 库来与 v2.0 终结点集成。v2.0 协议应与许多开源协议库兼容，不需要你进行重大更改。此类库的可用性根据语言和平台而有所不同，[Open ID Connect](http://openid.net/connect/) 和 [OAuth 2.0](http://oauth.net/2/) 网站维护了一份常见的实现列表。有关详细信息和经过 v2.0 终结点检验的开放源代码客户端库和示例列表，请参阅 [Azure Active Directory (AD) v2.0 和身份验证库](/documentation/articles/active-directory-v2-libraries/)。

我们还发布了仅用于 .NET 的 [Microsoft 身份验证库 (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) 的初始预览版。欢迎在 .NET 客户端和服务器应用程序中试用此库，但作为预览库，它不具备 GA 质量支持。

## 协议限制
v2.0 终结点仅支持 Open ID Connect 和 OAuth 2.0。但是，v2.0 终结点不支持这两个协议的某些特性与功能，包括：

- 允许应用程序结束与 v2.0 终结点的用户会话的 OpenID Connect `end_session_endpoint`。
- v2.0 终结点签发的 id\_token 只包含用户的成对标识符。这意味着两个不同的应用程序对同一用户将收到不同 ID。请注意，通过查询 Microsoft Graph `/me` 终结点，能够获得用户的可跨应用程序使用的相关 ID。
- 当前，即使从用户获取查看他们的电子邮件的权限，v2.0 终结点签发的 id\_token 也不包含该用户的 `email` 声明。
- OpenID Connect 用户信息终结点。目前尚未在 v2.0 终结点上实现用户信息终结点。但是，在该终结点上可能收到的所有用户概况数据都可从 Microsoft Graph `/me` 终结点获取。
- 角色和组声明。目前，v2.0 终结点不支持在 id\_token 中发布角色或组声明。

若要进一步了解 v2.0 终结点支持的协议功能范围，请阅读 [OpenID Connect 和 OAuth 2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)。

## 工作和学校帐户的限制
v2.0 终结点尚不支持一些特定于 Microsoft 组织/业务用户的功能。

##### 基于设备的条件性访问、本机和移动应用，以及 Microsoft Graph
v2.0 终结点尚不支持针对移动和本机应用程序的设备身份验证，如在 iOS 或 Android 上运行的本机应用。这可能会阻止本机应用程序调用特定组织的 Microsoft Graph。如果管理员在应用程序上设置了基于设备的条件性访问策略，则需要设备身份验证。对于 v2.0 终结点，基于设备的条件性访问的最可能的情形是管理员要对 Microsoft Graph，例如 Outlook API 中的一个资源设置策略。如果管理员设置了此策略，并且本机应用程序请求 Microsoft Graph 的令牌，则请求将最终失败，因为尚不支持设备身份验证。但是，如果配置了基于设备的策略，则支持请求 Microsoft Graph 令牌的 Web 应用程序。在 Web 应用场景中，通过用户的 Web 浏览器来执行设备身份验证。

作为开发人员，你很可能无法控制何时对 Microsoft Graph 资源设置策略，或者甚至不知道是何时设置的。如果要为工作和学校用户构建应用程序，那么在 v2.0 终结点支持设备身份验证之前，应使用[原始的 Azure AD 终结点](/documentation/articles/active-directory-developers-guide/)。

##### 针对联合租户的 Windows 集成身份验证
如果已在 Windows 应用程序中使用 ADAL（和原始的 Azure AD 终结点），则可能已经使用了所谓的 SAML 断言授予。此授予允许联合 Azure AD 租户的用户能够使用其本地 Active Directory 实例以无提示方式进行身份验证，而不用输入其凭据。当前 v2.0 终结点不支持 SAML 断言授予。

<!---HONumber=Mooncake_1031_2016-->
