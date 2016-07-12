<properties
	pageTitle="v2.0 终结点的限制和局限性 | Microsoft Azure"
	description="Azure AD v2.0 终结点的限制和局限性列表。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="03/18/2016"
	wacn.date="07/04/2016"/>
# 我是否应使用 v2.0 终结点？

当你创建与 Azure Active Directory 集成的应用程序时，必须判断 v2.0 终结点和身份验证协议是否符合需求。原有的 Azure AD 应用模型依然完全受到支持，并且在某些方面比 v2.0 的功能更丰富。但是，v2.0 终结点为开发人员[带来了明显的优势](/documentation/articles/active-directory-v2-compare/)，使你更乐于使用新的编程模型。随着时间的推移，v2.0 将发展到包含所有 Azure AD 功能，到时你只需使用 v2.0 终结点即可。

目前，v2.0 终结点可让你实现两项核心功能：

- 让用户使用个人帐户和工作帐户登录。
- 调用[融合的 Outlook API](https://dev.outlook.com)。

可以在 Web、移动和电脑应用程序等项目中实现这两种功能。如果这些相对受限的功能对你的应用程序有作用，我们建议使用 v2.0 终结点。如果你应用程序需要 Microsoft 服务中的任何其他功能，我们建议继续使用 Azure AD 的实际可行终结点和 Microsoft 帐户。今后，v2.0 终结点将与 Azure AD 和 Microsoft 帐户兼容，到时我们将帮助所有开发人员过渡到 v2.0 终结点。

在此同时，本文旨在帮助你判断 v2.0 终结点是否对你适用。我们将持续更新本文，以反映 v2.0 终结点的当前状态，请记得回来重新评估 v2.0 功能是否符合你的要求。

如果你的某个采用 Azure AD 的现有应用未使用 v2.0 终结点，你不需要从头开始进行配置。将来，我们会提供一种方法使现有的 Azure AD 应用程序可以使用 v2.0 终结点。

## 应用限制
v2.0 终结点目前不支持以下类型的应用。有关受支持应用类型的说明，请参阅[此文](/documentation/articles/active-directory-v2-flows/)。

##### 独立 Web API
在 v2.0 终结点中，你可以[构建使用 OAuth 2.0 保护的 Web API](/documentation/articles/active-directory-v2-flows/#web-apis)。但是，该 Web API 只能从共享相同应用程序 ID 的应用程序接收令牌。不支持构建从具有不同应用程序 ID 的客户端访问的 Web API。该客户端无法请求或获取对 Web API 的权限。

若要了解如何构建从具有相同应用 ID 的客户端接受令牌的 Web API，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)部分中的 v2.0 终结点 Web API 示例。

##### 守护程序/服务器端应用
包含长时运行进程或不需要用户操作的应用还需要通过其他方法访问受保护的资源，例如 Web API。这些应用可以通过 OAuth 2.0 客户端凭据流，使用应用的标识（而不是用户的委派标识）来进行身份验证和获取令牌。

v2.0 终结点目前不支持此流，也就是说，应用只能在发生交互式用户登录流之后获取令牌。不久之后将添加客户端凭据流。如果想要查看使用原始 Azure AD 终结点的客户端凭据流，请参阅 [GitHub 上的守护程序示例](https://github.com/AzureADSamples/Daemon-DotNet)。

##### Web API 代理流
许多体系结构包含需要调用另一个下游 Web API 的 Web API，这两者都受 v2.0 终结点的保护。此方案常见于具有 Web API 后端的本机客户端，该后端将调用 Microsoft Online 服务或另一个支持 Azure AD 的自定义构建 Web API。

使用 OAuth 2.0 Jwt 持有者凭据授权（也称为“代理流”）可支持此方案。但是，v2.0 终结点目前不支持代理流。若要查看此流在正式版 Azure AD 服务中如何工作，请参阅 [GitHub 上的代理代码示例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

## 应用注册限制
目前，所有想要与 v2.0 终结点集成的应用都必须在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 上创建新的应用注册。任何现有的 Azure AD 或 Microsoft 帐户应用将不会与 v2.0 终结点兼容，并且无法在任何门户（包括新的应用注册门户）中注册应用。我们计划提供一种方法来实现可将现有应用程序用作 v2.0 应用，但目前还没有将应用迁移到 v2.0 终结点的途径。

同样地，在新的应用注册门户中注册的应用将无法配合原有 Azure AD 身份验证终结点来运行。但是，你可以使用在应用注册门户中创建的应用，成功地与 Microsoft 帐户身份验证终结点 `https://login.live.com` 集成。

在新的应用程序注册门户中注册的应用目前限制为一组有限的 redirect\_uri 值。Web 应用和服务的 redirect\_uri 必须以方案或 `https` 开头，而所有其他平台的 redirect\_uri 必须使用 `urn:ietf:oauth:2.0:oob` 的硬编码值。

若要了解如何在新的应用程序注册门户中注册应用，请参阅[此文](/documentation/articles/active-directory-v2-app-registration/)。

## 服务和 API 限制
v2.0 终结点目前支持登录所有已在新应用程序注册门户中注册的应用，前提是该应用已在[支持的身份验证流](/documentation/articles/active-directory-v2-flows/)列表中列出。但是，这些应用只能获取 OAuth 2.0 访问令牌来访问非常有限的资源集。v2.0 终结点只为以下项目颁发 access\_token：

- 请求令牌的应用。如果逻辑应用包含多个不同的组件或层，则应用可为自身获取 access\_token。若要查看此方案的工作方式，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)教程。
- Outlook 邮件、日历和联系人 REST API，全都位于 https://outlook.office.com。 若要了解如何编写访问这些 API 的应用，请参阅 [Office Getting Started（Office 入门）](https://www.msdn.com/office/office365/howto/authenticate-Office-365-APIs-using-v2)教程。
- Microsoft 图形 API。若要了解 Microsoft Graph 和可用的所有数据，请访问 [https://graph.microsoft.io](https://graph.microsoft.io)。

目前不支持其他服务。将来会添加更多的 Microsoft Online 服务，并支持自定义构建的 Web API 和服务。

## 库和 SDK 限制
为了帮助你试用，我们提供了与 v2.0 终结点兼容的 Active Directory 身份验证库体验版。但是，此版本的 ADAL 处于预览状态 - 目前不受支持，未来几个月将有大幅改动。如果你想要尽快让应用配合 v2.0 终结点一起运行，[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)部分中提供了有关使用 ADAL for .NET、iOS、Android 和 Javascript 的代码示例。

如果你想要在生产应用程序中使用 v2.0 终结点，可使用以下选项：

- 如果你要构建 Web 应用程序，可以放心使用我们的正式版服务器端中间件来执行登录和令牌验证。其中包括适用于 ASP.NET 的 OWIN Open ID Connect 中间件和 NodeJS Passport 插件。[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)部分中也提供了有关使用这些中间件的示例代码。
- 对于其他平台以及本机与移动应用程序，你还可以通过直接在应用程序代码中发送和接收协议消息来与 v2.0 终结点集成。v2.0 OpenID Connect 和 OAuth 协议[有明确的说明文档](/documentation/articles/active-directory-v2-protocols/)，可帮助你执行这种集成。
- 最后，你可以使用开源 Open ID Connect 和 OAuth 库来与 v2.0 终结点集成。v2.0 协议应与许多开源协议库兼容，不需要你进行重大更改。此类库的可用性根据语言和平台而有所不同，[Open ID Connect](http://openid.net/connect/) 和 [OAuth 2.0](http://oauth.net/2/) 网站维护了一份常见实现的列表。下面是已通过 v2.0 终结点测试的开源客户端库和示例。

  - [Java WSO2 Identity Server](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server)
  - [Java Gluu Federation](https://github.com/GluuFederation/oxAuth)
  - [Node.Js passport-openidconnect](https://www.npmjs.com/package/passport-openidconnect)
  - [PHP OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP)
  - [iOS OAuth2 Client](https://github.com/nxtbgthng/OAuth2Client)
  - [Android OAuth2 Client](https://github.com/wuman/android-oauth-client)
  - [Android OpenID Connect Client](https://github.com/kalemontes/OIDCAndroidLib)

## 协议限制
v2.0 终结点仅支持 Open ID Connect 和 OAuth 2.0。但是，并非每个协议的所有特性与功能都已合并到 v2.0 终结点中。示例包括：

- OpenID Connect `end_sesssion_endpoint`
- OAuth 2.0 客户端凭据授权

若要进一步了解 v2.0 终结点支持的协议功能范围，请阅读 [OpenID Connect & OAuth 2.0 Protocol Reference（OpenID Connect 和 OAuth 2.0 协议参考）](/documentation/articles/active-directory-v2-protocols/)。

## 高级 Azure AD 开发人员功能
Azure Active Directory 服务提供一组开发人员功能（v2.0 终结点尚不支持这些功能），包括：

- Azure AD 用户的组声明
- 应用程序角色和角色声明

<!---HONumber=Mooncake_0516_2016-->