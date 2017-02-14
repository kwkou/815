<properties
    pageTitle="Azure Active Directory v2.0 终结点的应用类型 | Azure"
    description="Azure Active Directory v2.0 终结点支持的应用类型和方案。"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="494a06b8-0f9b-44e1-a7a2-d728cf2077ae"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/13/2017"
    ms.author="dastrock" />  


# Azure Active Directory v2.0 终结点的应用类型
Azure Active Directory (Azure AD) v2.0 终结点支持各种现代应用体系结构的身份验证，所有这些体系结构都基于行业标准协议 [OAuth 2.0 或 OpenID Connect](/documentation/articles/active-directory-v2-protocols/)。本文介绍可以使用 Azure AD v2.0 构建的应用类型（不管偏好的语言或平台是什么）。本文中的信息旨在帮助用户了解一些高级方案，使用户能够[开始编写代码](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)。

> [AZURE.NOTE]
v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。
> 
> 

## 基础知识
必须在 [Microsoft 应用程序注册门户](https://apps.dev.microsoft.com)中注册每个使用 v2.0 终结点的应用。应用注册过程将收集并分配应用的以下值：

- 用于唯一标识应用的**应用程序 ID**
- 可用于将响应定向回到应用的**重定向 URI**
- 其他一些特定于方案的值

请在[注册应用](/documentation/articles/active-directory-v2-app-registration/)中了解详细信息。

注册应用后，应用将通过向 Azure AD v2.0 终结点发送请求来与 Azure AD 通信。我们提供了用于处理这些请求详细信息的开源框架和库。你也可以通过针对这些终结点创建请求，来自行实现身份验证逻辑：

	
	https://login.microsoftonline.com/common/oauth2/v2.0/authorize
	https://login.microsoftonline.com/common/oauth2/v2.0/token

<!-- TODO: Need a page for libraries to link to -->


## Web 应用  <a name="web-apps"></a>
对于用户通过浏览器访问的 Web 应用（.NET、PHP、Java、Ruby、Python、Node），可以使用 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/) 来执行用户登录。在 OpenID Connect 中，Web 应用将接收一个 ID 令牌。ID 令牌是一个安全令牌，用于验证用户的标识并以声明形式提供用户相关信息：


	// Partial raw ID token
	eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...
	
	// Partial content of a decoded ID token
	{
		"name": "John Smith",
		"email": "john.smith@gmail.com",
		"oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
		...
	}


可以在 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)中了解提供给应用的各种令牌和声明。

在 Web 服务器应用中，登录身份验证流大致采用以下步骤：

![Web 应用身份验证流](./media/active-directory-v2-flows/convergence_scenarios_webapp.png)  


使用从 v2.0 终结点接收的公共签名密钥验证 ID 令牌便可确保用户的标识正确。这会设置一个可在后续页面请求中用于识别用户的会话 Cookie。

若要查看此方案的工作方式，请尝试运行 v2.0 [入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分中提供的 Web 应用登录代码示例之一。

除了简单登录以外，Web 服务器应用可能还需要访问其他 Web 服务，例如 REST API。在这种情况下，Web 服务器应用将使用 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols/)参与合并的 OpenID Connect 和 OAuth 2.0 流。有关此方案的详细信息，请参阅 [Web 应用和 Web API 入门](/documentation/articles/active-directory-v2-devquickstarts-webapp-webapi-dotnet/)。


## Web API <a name="web-apis"></a>
可以使用 v2.0 终结点来保护 Web 服务，例如应用的 RESTful Web API。Web API 使用 OAuth 2.0 访问令牌而不是 ID 令牌和会话 Cookie 来保护数据以及对传入的请求进行身份验证。Web API 调用方将在 HTTP 请求的授权标头中追加一个访问令牌，如下所示：


	GET /api/items HTTP/1.1
	Host: www.mywebapi.com
	Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
	Accept: application/json
	...


Web API 使用此访问令牌来验证 API 调用方的标识，并从访问令牌中编码的声明提取调用方的相关信息。若要了解提供给应用的各种令牌和声明，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)。

Web API 可让用户通过公开权限（也称为[范围](/documentation/articles/active-directory-v2-scopes/)）来选择添加或排除特定的功能或数据。为了使调用应用能够获取某个范围的权限，用户必须在执行流的过程中许可该范围。v2.0 终结点向用户请求权限，然后将这些权限记录在 Web API 收到的所有访问令牌中。Web API 将验证每次调用后收到的访问令牌，并执行授权检查。

Web API 可以从各种应用接收访问令牌，其中包括 Web 服务器应用、桌面和移动应用、单页应用、服务器端守护程序，甚至其他 Web API。Web API 的高级流如下所示：

![Web API 身份验证流](./media/active-directory-v2-flows/convergence_scenarios_webapi.png)  


若要详细了解授权代码、刷新令牌和获取访问令牌的详细步骤，请参阅 [OAuth 2.0 协议](/documentation/articles/active-directory-v2-protocols-oauth-code/)。

若要了解如何使用 OAuth2 访问令牌保护 Web API，请查看[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分提供的 Web API 代码示例。

## 移动和本机应用  <a name="mobile-and-native-apps"></a>
在设备中安装的应用（如移动和桌面应用）通常需要访问用于存储数据和代表用户执行功能的后端服务或 Web API。这些应用可以使用 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols-oauth-code/)将登录凭据和授权添加到后端服务。

在此流中，应用在用户登录时从 v2.0 终结点接收授权代码。授权代码表示应用有权代表登录用户调用后端服务。应用可在后台交换 OAuth 2.0 访问令牌和刷新令牌的授权代码。应用可以使用访问令牌在 HTTP 请求中向 Web API 进行身份验证；在旧的访问令牌过期时，可以使用刷新令牌获取新的访问令牌。

![本机应用身份验证流](./media/active-directory-v2-flows/convergence_scenarios_native.png)  


## 单页应用 (JavaScript)
许多新式应用都有一个单页应用前端（主要以 JavaScript 编写）。通常，该前端是使用 AngularJS、Ember.js、Durandal.js 等框架编写的。Azure AD v2.0 终结点使用 [OAuth 2.0 隐式流](/documentation/articles/active-directory-v2-protocols-implicit/)为这些应用提供支持。

在此流中，应用直接从 v2.0 授权终结点接收令牌，而不需要执行任何服务器到服务器的交换。所有身份验证逻辑和会话处理将完全在 JavaScript 客户端中发生，无需额外的页面重定向。

![隐式身份验证流](./media/active-directory-v2-flows/convergence_scenarios_implicit.png)  


若要查看此方案的实际运行情况，请尝试运行[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分提供的单页应用代码示例之一。

### 守护程序和服务器端应用
包含长时间运行的进程或不需要与用户交互的应用也需要通过一种方法来访问受保护的资源，例如 Web API。这些应用可以通过 OAuth 2.0 客户端凭据流，使用应用的标识（而不是用户的委托标识）来进行身份验证和获取令牌。

在此流中，应用通过直接与 `/token` 终结点交互来获取终结点：

![守护程序应用身份验证流](./media/active-directory-v2-flows/convergence_scenarios_daemon.png)  


若要构建守护程序应用，请参阅[入门](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started/)部分中的客户端凭据文档，或者尝试学习一个 [.NET 示例应用](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)。

## 当前限制
v2.0 终结点目前不支持本部分中所述的应用类型，但这些应用已列入将来的开发路线图中。有关 v2.0 终结点的其他限制和局限性，请参阅[我是否应使用 v2.0 终结点？](/documentation/articles/active-directory-v2-limitations/)。

### 链接的 Web API（代理）
许多体系结构都包含需要调用另一个下游 Web API 的 Web API，这两者都受 v2.0 终结点的保护。此方案常见于具有 Web API 后端的本机客户端，该后端反过来调用 Office 365 或图形 API 等 Microsoft Online Services 实例。

可以使用 OAuth 2.0 JSON Web 令牌 (JWT) 持有者凭据授权（也称为[代理流](/documentation/articles/active-directory-v2-protocols/)）来支持这种链接的 Web API 方案。v2.0 终结点中目前尚未实现代理流。若要查看此流在正式版 Azure AD 服务中的工作原理，请参阅 [GitHub 上的代理代码示例](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update-->