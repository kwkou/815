<properties
	pageTitle="v2.0 终结点的类型 | Azure"
	description="Azure AD v2.0 终结点支持的应用和方案类型。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="07/04/2016"/>

# v2.0 终结点的类型
v2.0 终结点支持各种现代应用体系结构的身份验证，所有这些体系结构基于行业标准协议 [OAuth 2.0](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow) 和/或 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/#openid-connect-sign-in-flow)。本文档简要介绍你可以构建的应用类型（无论你使用哪种语言或平台）。其中将会帮助你了解一些高级方案，然后你便可以[开始编写代码](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 基础知识
所有使用 v2.0 终结点的应用都必须在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 上注册。应用注册进程会收集一些值并将其分配到你的应用：

- 用于唯一标识应用的**应用程序 ID**
- 用于将响应定向回到应用的**重定向 URI**
- 其他一些特定于方案的值。有关详细信息，请了解如何[注册应用](/documentation/articles/active-directory-v2-app-registration/)。

注册后，应用将向 Azure Active Directory v2.0 终结点发送请求，以便与 Azure AD 通信。我们提供了用于处理这些请求详细信息的开源框架和库，你也可以自行编写对这些终结点的请求，来实现身份验证逻辑：

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
https://login.microsoftonline.com/common/oauth2/v2.0/token
```
<!-- TODO: Need a page for libraries to link to -->

## Web 应用
对于通过浏览器访问的 Web 应用（.NET、PHP、Java、Ruby、Python、Node 等），可以使用 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/#openid-connect-sign-in-flow) 来执行用户登录。在 OpenID Connect 中，Web 应用将接收 `id_token`，这是一个安全令牌，用于验证用户的标识并以声明形式提供有关用户的信息：


		// Partial raw id_token
		eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...
		
		// Partial content of a decoded id_token
		{
			"name": "John Smith",
			"email": "john.smith@gmail.com",
			"oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
			...
		}


你可以在 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)中了解提供给应用的各种令牌和声明。

在 Web 服务器应用中，登录身份验证流采用以下高级步骤：

![Web 应用泳道图像](../media/active-directory-v2-flows/convergence_scenarios_webapp.png)

使用从 v2.0 终结点收到的公共签名密钥验证 id\_token 便足以确保用户的标识正确，以及设置可在后续页面请求中用来识别用户的会话 Cookie。

若要查看此方案的工作方式，请尝试运行[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)部分中提供的 Web 应用登录代码示例之一。

除了简单登录，Web 服务器应用可能还需要访问其他一些 Web 服务，例如 REST API。在这种情况下，Web 服务器应用可以使用 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow)参与合并的 OpenID Connect 和 OAuth 2.0 流。[WebApp-WebAPI 入门主题](/documentation/articles/active-directory-v2-devquickstarts-webapp-webapi-dotnet/)中介绍了此方案。

## Web API
你可以使用 v2.0 终结点来保护 Web 服务，例如应用的 RESTful Web API。Web API 使用 OAuth 2.0 access\_token 而不是 id\_token 和会话 Cookie 来保护数据以及对传入的请求进行身份验证。Web API 调用方会在 HTTP 请求的授权标头中附加一个 access\_token：

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

然后 Web API 使用此 access\_token 来验证 API 调用方的标识，并从 access\_token 中编码的声明提取调用方的相关信息。你可以在 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)中了解提供给应用的各种令牌和声明。

Web API 可让用户通过公开权限来选择添加/排除特定的功能或数据（也称为[范围](/documentation/articles/active-directory-v2-scopes/)）。为了使调用应用能够获取某个范围的权限，用户必须在执行流的过程中同意该范围。v2.0 终结点负责向用户请求权限，并将这些权限记录在 Web API 收到的所有 access\_token 中。Web API 只需关心验证每次调用所收到的 access\_token，并执行适当的授权检查即可。

Web API 可以从各种应用接收 access\_token，其中包括 Web 服务器应用、桌面和移动应用、单页应用、服务器端守护程序，甚至其他 Web API。Web API 身份验证的高级流如下所示：

![Web API 泳道图像](../media/active-directory-v2-flows/convergence_scenarios_webapi.png)

若要了解 authorization\_codes、refresh\_tokens 和获取 access\_tokens 的详细步骤，请参阅 [OAuth 2.0 协议](/documentation/articles/active-directory-v2-protocols-oauth-code/)。

若要了解如何使用 OAuth2 access\_tokens 保护 Web API，请查看[入门部分](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)中的 Web API 代码示例。


## 移动和本机应用
安装在设备中的应用（如移动和桌面应用）通常需要访问用于存储数据和代表用户执行各种功能的后端服务或 Web API。这些应用可以使用 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols-oauth-code/)将登录凭据和授权添加到后端服务。

在此流中，应用将在用户登录时从 v2.0 终结点接收 authorization\_code，这表示应用代表当前登录用户调用后端服务的权限。然后，应用可以在后台交换 OAuth 2.0 access\_token 和 refresh\_token 的 authoriztion\_code。应用可以使用 access\_token 在 HTTP 请求中向 Web API 进行身份验证，并可以在旧的 access\_token 过期时，用 refresh\_token 获取新的 access\_token。

![本机应用泳道图像](./media/active-directory-v2-flows/convergence_scenarios_native.png)

## 单页应用 (javascript)
许多现代应用都有一个单页应用 (SPA) 前端，该前端主要是以 javascript 编写的，并且通常使用 AngularJS、Ember.js、Durandal 等框架。Azure AD v2.0 终结点支持这些使用 [OAuth 2.0 隐式流](/documentation/articles/active-directory-v2-protocols-implicit/)的应用。

在此流中，应用直接从 v2.0 授权终结点接收令牌，而不需要执行任何后端服务器到服务器的交换。这样，所有身份验证逻辑和会话处理将完全在 javascript 客户端中发生，且无需执行额外的页面重定向。

![隐式流泳道图像](./media/active-directory-v2-flows/convergence_scenarios_implicit.png)

若要查看此方案的工作方式，请尝试运行[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)部分中提供的单页应用代码示例之一。

## 当前限制
v2.0 终结点目前不支持这些类型的应用，但这项支持已列入开发路线图中。[v2.0 限制文章](/documentation/articles/active-directory-v2-limitations/)中说明了 v2.0 终结点的其他限制和局限性。

### 守护程序/服务器端应用
包含长时运行进程或不需要用户操作的应用还需要通过其他方法访问受保护的资源，例如 Web API。这些应用可以通过 OAuth 2.0 客户端凭据流，使用应用的标识（而不是用户的委派标识）来进行身份验证和获取令牌。

v2.0 终结点中目前不支持客户端凭据流。若要查看此流在正式版 Azure AD 服务中如何工作，请参阅 [GitHub 上的守护程序代码示例](https://github.com/AzureADSamples/Daemon-DotNet)。

### 链接的 Web API（代理）
许多体系结构包含需要调用另一个下游 Web API 的 Web API，这两者都受 v2.0 终结点的保护。此方案常见于具有 Web API 后端的本机客户端，该后端将调用 Office 365 或图形 API 等 Microsoft Online 服务。

可以使用 OAuth 2.0 Jwt 持有者凭据授权（也称为[代理流](/documentation/articles/active-directory-v2-protocols/#oauth2-on-behalf-of-flow)）来支持这种链接的 Web API 方案。但是，v2.0 终结点中目前尚未实现代理流。若要查看此流在正式版 Azure AD 服务中如何工作，请参阅 [GitHub 上的代理代码示例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

<!---HONumber=Mooncake_0516_2016-->