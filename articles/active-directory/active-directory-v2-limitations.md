<properties
    pageTitle="Azure Active Directory v2.0 终结点的限制和局限性 | Azure"
    description="Azure AD v2.0 终结点的限制和局限性列表。"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="a99289c0-e6ce-410c-94f6-c279387b4f66"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/13/2017"
    ms.author="dastrock" />

# 我是否应使用 v2.0 终结点？
构建与 Azure Active Directory (Azure AD) 集成的应用程序时，需要确定 v2.0 终结点和身份验证协议是否满足需求。原始的 Azure AD 终结点仍完全受支持，在某些方面甚至比 v2.0 的功能更丰富。但是，v2.0 终结点为开发人员[带来了明显的优势](/documentation/articles/active-directory-v2-compare/)。v2.0 的优势可能使你更乐于使用新的编程模型。

下面是 v2.0 终结点目前的建议使用场合：

- 如果要在应用程序中支持个人 Microsoft 帐户，请使用 v2.0 终结点。但在使用之前，请务必了解本文中所述的限制，尤其是适用于工作和学校帐户的限制。
- 如果应用程序仅需要支持工作和学校帐户，请使用[原始的 Azure AD 终结点](/documentation/articles/active-directory-developers-guide/)。

随着时间推移，v2.0 终结点将会逐步移除此处列出的限制，届时就只需要使用 v2.0 终结点。同时，本文旨在帮助用户判断 v2.0 终结点是否适合自己。我们将持续更新本文，反映 v2.0 终结点的最新状态。请不时返回查阅本文，重新评估 v2.0 功能是否符合要求。

如果现有的 Azure AD 应用未使用 v2.0 终结点，则不需要从头开始进行配置。将来，我们会提供一种方法，使现有的 Azure AD 应用程序能够与 v2.0 终结点配合使用。

## 应用类型的限制
v2.0 终结点目前不支持以下类型的应用。有关受支持应用类型的说明，请参阅 [Azure Active Directory v2.0 终结点的应用类型](/documentation/articles/active-directory-v2-flows/)。

### 独立 Web API
可以使用 v2.0 终结点[构建使用 OAuth 2.0 保护的 Web API](/documentation/articles/active-directory-v2-flows/#web-apis/)。但是，该 Web API 只能从具有相同应用程序 ID 的应用程序接收令牌。无法访问具有不同应用程序 ID 的客户端中的 Web API。该客户端无法请求或获取对 Web API 的权限。

若要了解如何构建可从具有相同应用程序 ID 的客户端接受令牌的 Web API，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分中的 v2.0 终结点 Web API 示例。

### Web API 代理流
许多体系结构都包含需要调用另一个下游 Web API 的 Web API，这两者都受 v2.0 终结点的保护。此方案常见于具有 Web API 后端的本机客户端，该后端反过来调用 Microsoft Online Services 的实例或其他支持 Azure AD 的自定义构建 Web API。

可以使用 OAuth 2.0 JSON Web 令牌 (JWT) 持有者凭据授权（也称为“代理流”）来创建此方案。但是，v2.0 终结点目前不支持代理流。若要查看此流在正式版 Azure AD 服务中的工作原理，请参阅 [GitHub 上的代理代码示例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

## 应用注册限制
目前，对于想要与 v2.0 终结点集成的每个应用，必须在新的 [Microsoft 应用程序注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)中创建一个应用注册。现有的 Azure AD 或 Microsoft 帐户应用不兼容 v2.0 终结点。不是在应用程序注册门户中注册的应用不兼容 v2.0 终结点。我们已计划在将来提供一种方法，使现有应用程序可用作 v2.0 应用。不过，现有的应用目前没有迁移路径，无法与 v2.0 终结点配合工作。

在应用程序注册门户中注册的应用无法与原始的 Azure AD 身份验证终结点配合工作。但是，可以使用在应用程序注册门户中创建的应用，成功地与 Microsoft 帐户身份验证终结点 `https://login.live.com` 集成。

此外，在[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)中创建的应用注册具有以下注意事项：

- 不支持**主页**属性（也称为*登录 URL*）。在没有主页的情况下，这些应用程序不会显示在 Office MyApps 面板中。
- 目前，每个应用程序 ID 只允许有两个应用机密。
- 应用注册只能由一个开发人员帐户查看和管理，不能在多个开发人员之间共享。
- 允许的重定向 URI 格式存在一些限制。有关重定向 URI 的详细信息，请参阅下一部分。

## 重定向 URI 的限制
在应用程序注册门户中注册的应用目前仅限使用一组有限的重定向 URI 值。Web 应用和服务的重定向 URI 必须以方案 `https` 开头，所有重定向 URI 值必须共享一个 DNS 域。例如，无法注册具有以下重定向 URI 之一的 Web 应用：

`https://login-east.contoso.com`  
`https://login-west.contoso.com`

注册系统会将现有重定向 URI 的完整 DNS 名称与要添加的重定向 URI 的 DNS 名称进行比较。如果满足以下任一条件，添加 DNS 名称的请求将会失败：

- 新重定向 URI 的完整 DNS 名称与现有重定向 URI 的 DNS 名称不匹配。
- 新重定向 URI 的完整 DNS 名称不是现有重定向 URI 的子域。

例如，如果应用具有以下重定向 URI：

`https://login.contoso.com`  


则可以添加类似于下面的内容：

`https://login.contoso.com/new`  


在此情况下，DNS 名称完全匹配。或者，可以添加以下内容：

`https://new.login.contoso.com`  


在此情况下，将引用 login.contoso.com 的 DNS 子域。如果希望应用使用 login-east.contoso.com 和 login-west.contoso.com 作为重定向 URI，必须按顺序添加以下重定向 URI：

`https://contoso.com`  
`https://login-east.contoso.com`  
`https://login-west.contoso.com`  

可以添加后两个重定向 URI，因为它们是第一个重定向 URI (contoso.com) 的子域。即将发布的版本中将取消此限制。

若要了解如何在应用程序注册门户中注册应用，请参阅[如何使用 v2.0 终结点注册应用](/documentation/articles/active-directory-v2-app-registration/)。

## 服务和 API 限制
v2.0 终结点目前支持登录到已在应用程序注册门户中注册的任何应用，前提是该应用已在[支持的身份验证流](/documentation/articles/active-directory-v2-flows/)列表中列出。但是，这些应用只能获取 OAuth 2.0 访问令牌来访问一组非常有限的资源。v2.0 终结点只为以下目的颁发访问令牌仅：

- 请求令牌的应用。如果逻辑应用包含多个不同的组件或层，则应用可为自身获取访问令牌。若要查看此方案的实际运行情况，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)教程。
- Outlook 邮件、日历和联系人 REST API，全都位于 https://outlook.office.com。若要了解如何编写访问这些 API 的应用，请参阅 [Office 入门](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)教程。
- Microsoft 图形 API。详细了解 [Microsoft Graph](https://graph.microsoft.io) 和可用的数据。

目前不支持其他服务。将来会添加更多的 Microsoft Online Services，并支持自定义构建的 Web API 和服务。

## 库和 SDK 限制
当前，对 v2.0 终结点的库支持有限。如果想要在生产应用程序中使用 v2.0 终结点，可使用以下选项：

- 如果要构建 Web 应用程序，可以放心使用 Microsoft 正式版服务器端中间件来执行登录和令牌验证。其中包括适用于 ASP.NET 的 OWIN Open ID Connect 中间件和 Node.js Passport 插件。有关使用 Microsoft 中间件的代码示例，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分。
- 对于其他平台以及本机与移动应用程序，可以通过直接在应用程序代码中发送和接收协议消息来与 v2.0 终结点集成。v2.0 OpenID Connect 和 OAuth 协议提供了[明确的说明文档](/documentation/articles/active-directory-v2-protocols/)来帮助用户执行这种集成。
- 最后，可以使用开源 Open ID Connect 和 OAuth 库来与 v2.0 终结点集成。v2.0 协议应与许多开源协议库兼容，不需要进行重大更改。此类库的可用性根据语言和平台而有所不同。[Open ID Connect](http://openid.net/connect/) 和 [OAuth 2.0](http://oauth.net/2/) 网站中维护了一份常用实现的列表。有关详细信息和通过 v2.0 终结点进行测试的开源客户端库与示例列表，请参阅 [Azure Active Directory v2.0 和身份验证库](/documentation/articles/active-directory-v2-libraries/)。

我们发布了仅适用于 .NET 的 [Microsoft 身份验证库 (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) 的初始预览版。欢迎在 .NET 客户端和服务器应用程序中试用此库，但作为预览库，它不具备正式版 (GA) 的质量支持。

## 协议限制
v2.0 终结点仅支持 Open ID Connect 和 OAuth 2.0。但是，并非每个协议的所有特性与功能都已合并到 v2.0 终结点中。

以下典型协议特性和功能目前在 v2.0 终结点中*不可用*：

- 允许应用结束用户会话的 OpenID Connect `end_session_endpoint` 参数在 v2.0 终结点中不可用。
- v2.0 终结点颁发的 ID 令牌只包含用户的成对标识符。这意味着，两个不同的应用程序将收到同一用户的不同 ID。请注意，可以通过查询 Microsoft Graph `/me` 终结点，获取可跨应用程序使用的用户相关 ID。
- 即使从用户获取查看其电子邮件的权限，v2.0 终结点颁发的 ID 令牌也不包含该用户的 `email` 声明。
- OpenID Connect UserInfo 终结点尚未在 v2.0 终结点上实现。但是，在该终结点上可能收到的所有用户配置文件数据都可从 Microsoft Graph `/me` 终结点获取。
- v2.0 终结点不支持在 ID 令牌中颁发角色或组声明。

若要进一步了解 v2.0 终结点支持的协议功能范围，请阅读 [OpenID Connect 和 OAuth 2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)。

## 工作和学校帐户限制
v2.0 终结点尚不支持一些特定于 Microsoft 企业用户的功能。有关详细信息，请阅读后续部分。

### 基于设备的条件访问、本机和移动应用，以及 Microsoft Graph
v2.0 终结点尚不支持针对移动和本机应用程序的设备身份验证，如在 iOS 或 Android 上运行的本机应用。在某些组织中，这可能会阻止本机应用程序调用 Microsoft Graph。如果管理员在应用程序上设置了基于设备的条件访问策略，则需要设备身份验证。对于 v2.0 终结点，最可能需要用到基于设备的条件访问的情形是，管理员要对 Microsoft Graph 中的一个资源（例如 Outlook API）设置策略。如果管理员设置了此策略，并且本机应用程序请求 Microsoft Graph 的令牌，则请求最终会失败，因为尚不支持设备身份验证。但是，如果配置了基于设备的策略，则支持请求 Microsoft Graph 令牌的 Web 应用程序。在 Web 应用场景中，设备身份验证是通过用户的 Web 浏览器执行的。

开发人员很可能无法控制何时对 Microsoft Graph 资源设置策略，甚至不知道已设置了这些策略。如果要为工作和学校用户构建应用程序，那么在 v2.0 终结点支持设备身份验证之前，应使用[原始的 Azure AD 终结点](/documentation/articles/active-directory-developers-guide/)。

### 针对联合租户的 Windows 集成身份验证
如果已在 Windows 应用程序中使用 Active Directory 身份验证库 (ADAL)（和原始的 Azure AD 终结点），则可能已在利用所谓的安全断言标记语言 (SAML) 断言授权。借助这种授权，联合 Azure AD 租户的用户可使用其本地 Active Directory 实例以静默方式进行身份验证，而无需输入凭据。v2.0 终结点目前不支持 SAML 断言授权。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update-->