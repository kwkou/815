
<properties
	pageTitle="Azure AD v2.0 OpenID Connect 协议 | Azure"
	description="使用 Azure AD 的 OpenID Connect 身份验证协议 v2.0 实现构建 Web 应用程序。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="msmbaldwin"
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


# v2.0 协议 - OpenID Connect
OpenID Connect 是构建在 OAuth 2.0 基础之上的身份验证协议，可用于将用户安全登录到 Web 应用程序。使用 v2.0 终结点的 OpenID Connect 实现，可以将登录和 API 访问权限添加到基于 Web 的应用程序中。本指南与语言无关，说明如何实现上述目的，并介绍在不使用我们的任何开放源代码库的情况下，如何发送和接收 HTTP 消息。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

[OpenID Connect](http://openid.net/specs/openid-connect-core-1_0.html) 扩展了 OAuth 2.0 *授权*协议，以将其用作*身份验证*协议，从而允许你使用 OAuth 执行单一登录。它引入了 `id_token` 的概念，这是一种安全令牌，可让客户端验证用户的标识，并获取有关用户的基本配置文件信息。由于它扩展了 OAuth 2.0，因此还可让应用程序安全地获取 **access\_tokens**，而这些令牌可用于访问[授权服务器](/documentation/articles/active-directory-v2-protocols/#the-basics/)保护的资源。如果要构建的 [Web 应用程序](/documentation/articles/active-directory-v2-flows/#web-apps/)托管在服务器中并通过浏览器访问，我们建议使用 OpenID Connect。

## 协议图 - 登录
最基本的登录流包含以下步骤 - 下面详细描述了每个步骤。

![OpenId Connect Swimlanes](./media/active-directory-v2-flows/convergence_scenarios_webapp.png)  


## 提取 OpenID Connect 元数据文档
OpenID Connect 描述了元数据文档，该文档包含了应用执行登录所需的大部分信息。这些信息包括诸如要使用的 URL、服务的公共签名密钥的位置等。对于 v2.0 终结点，应使用的 OpenID Connect 的元数据文档为：


	https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration


其中 `{tenant}` 可以接受以下四个不同值之一：

| 值 | 说明 |
| ----------------------- | ------------------------------- |
| `common` | 允许用户使用个人的 Microsoft 帐户和工作/学校帐户从 Azure Active Directory 登录应用程序。 |
| `organizations` | 仅允许用户使用工作/学校帐户从 Azure Active Directory 登录应用程序。 |
| `consumers` | 仅允许用户使用个人的 Microsoft 帐户 (MSA) 登录应用程序。 |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.partner.onmschina.cn` | 仅允许用户使用工作/学校帐户从特定的 Azure Active Directory 租户登录应用程序。可以使用 Azure AD 租户的友好域名或租户的 GUID 标识符。 |

元数据是一个简单的 json 文档，下面提供了其中一个代码段。[OpenID Connect 规范](https://openid.net)对其内容进行了完整描述。


	{
		"authorization_endpoint": "https:\/\/login.microsoftonline.com\/common\/oauth2\/v2.0\/authorize",
		"token_endpoint": "https:\/\/login.microsoftonline.com\/common\/oauth2\/v2.0\/token",
		"token_endpoint_auth_methods_supported": [
			"client_secret_post",
			"private_key_jwt"
		],
		"jwks_uri": "https:\/\/login.microsoftonline.com\/common\/discovery\/v2.0\/keys",
  
		...
  
	}


通常，使用此元数据文档来配置 OpenID Connect 库或 SDK；该库使用元数据来完成其工作。但是，如果不使用预生成的 OpenID Connect 库，则可以按照本文剩余部分的步骤来使用 v2.0 终结点执行 Web 应用中的登录。

## 发送登录请求
当 Web 应用需要对用户进行身份验证时，可以将用户定向到 `/authorize` 终结点。此请求类似于 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols-oauth-code/)的第一个阶段，不过有几个重要的区别：

- 该请求必须在 `scope` 参数中包含范围 `openid`。
- `response_type` 参数必须包含 `id_token`
- 该请求必须包含 `nonce` 参数


		// 换行符仅用于增强可读性
		
		GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&response_type=id_token
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&response_mode=form_post
		&scope=openid
		&state=12345
		&nonce=678910
	

> [AZURE.TIP] 单击下面的链接以执行此请求！ 登录之后，你的浏览器应重定向至地址栏中具有 `id_token` 的 `https://localhost/myapp/`。请注意，此请求会使用 `response_mode=query`（仅用于教程）。建议使用 `response_mode=form_post`。
    <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=openid&response_mode=query&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必填 | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints/)。 |
| client\_id | 必填 | 注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| response\_type | 必填 | 必须包含 OpenID Connect 登录的 `id_token`。还可以包含其他 response\_type，例如 `code`。 |
| redirect\_uri | 建议 | 应用程序的 redirect\_uri，应用程序可在此发送及接收身份验证响应。其必须完全符合在门户中注册的其中一个 redirect\_uris，否则必须是编码的 url。 |
| 作用域 | 必填 | 范围的空格分隔列表。针对 OpenID Connect，即必须包含范围 `openid`，其在同意 UI 中转换为“将你登录”权限。也可以在此请求中包含其他范围，以请求同意。 |
| nonce | 必填 | 由应用程序生成且包含在请求中的值，以声明方式包含在生成的 id\_token 中。应用程序接着便可确认此值，以减少令牌重新执行攻击。此值通常是随机的唯一字符串，可用以识别请求的来源。 |
| response\_mode | 建议 | 指定将生成的 authorization\_code 送回到应用程序所应该使用的方法。可以是“query”、“form\_post”或“fragment”。对于 Web 应用程序，建议使用 `response_mode=form_post`，确保以最安全的方式将令牌传输到你的应用程序。  
| state | 建议 | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。随机生成的唯一值通常用于[防止跨站点请求伪造攻击](http://tools.ietf.org/html/rfc6749#section-10.12)。该状态也用于在身份验证请求出现之前，于应用程序中编码用户的状态信息，例如之前所在的网页或视图。 |
| prompt | 可选 | 表示需要的用户交互类型。目前唯一的有效值为“login”、“none”和“consent”。`prompt=login` 强制用户在该请求上输入凭据，否定单一登录。`prompt=none` 则相反 — 它确保不对用户显示任何交互式提示。如果请求无法通过单一登录以无提示方式完成，v2.0 终结点将返回错误。`prompt=consent` 将在用户登录之后触发 OAuth 同意对话框，要求用户向应用授予权限。 |
| login\_hint | 可选 | 如果事先知道其用户名称，可用于预先填充用户登录页面的用户名称/电子邮件地址字段。通常，应用将在重新身份验证期间使用此参数，并且已经使用 `preferred_username` 声明从前次登录提取用户名。 |
| domain\_hint | 可选 | 可以是 `consumers` 或 `organizations` 之一。如果包含，它跳过用户在 v2.0 登录页面上经历的基于电子邮件的发现过程，导致稍微更加流畅的用户体验。通常，应用将在重新身份验证期间使用此参数，方法是从 id\_token 提取 `tid` 声明。如果 `tid` 声明值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，应该使用 `domain_hint=consumers`。否则使用 `domain_hint=organizations`。 |
此时，请求用户输入其凭据并完成身份验证。v2.0 终结点也将确保用户已经同意 `scope` 查询参数中指示的权限。如果用户未曾同意这些权限的任何一项，就请求用户同意请求的权限。[此处提供了权限、同意与多租户应用](/documentation/articles/active-directory-v2-scopes/)的详细信息。

用户经过身份验证并同意后，v2.0 终结点将使用 `response_mode` 参数中指定的方法，将响应返回到位于所指示的 `redirect_uri` 的应用。

#### 成功的响应
使用 `response_mode=form_post` 的成功响应如下所示：


	POST /myapp/ HTTP/1.1
	Host: localhost
	Content-Type: application/x-www-form-urlencoded
	
	id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&state=12345
	

| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| id\_token | 应用程序请求的 id\_token。可以使用 id\_token 验证用户的标识，并以用户身份开始会话。有关 id\_token 及其内容的更多详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |
| state | 如果请求中包含状态参数，响应中就应该出现相同的值。应用程序应该验证请求和响应中的状态值是否完全相同。 |

#### 错误响应
错误响应可能也发送到 `redirect_uri`，让应用可以适当地处理：


	POST /myapp/ HTTP/1.1
	Host: localhost
	Content-Type: application/x-www-form-urlencoded
	
	error=access_denied&error_description=the+user+canceled+the+authentication
	

| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |

#### 授权终结点错误的错误代码

下表描述了可在错误响应的 `error` 参数中返回的各个错误代码。

| 错误代码 | 说明 | 客户端操作 |
|------------|-------------|---------------|
| invalid\_request | 协议错误，例如，缺少必需的参数。 | 修复并重新提交请求。这通常是在初始测试期间捕获的开发错误。|
| unauthorized\_client | 不允许客户端应用程序请求授权代码。 | 客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| access\_denied | 资源所有者拒绝了许可 | 客户端应用程序可以通知用户除非用户许可，否则无法继续。 |
| unsupported\_response\_type | 授权服务器不支持请求中的响应类型。 | 修复并重新提交请求。这通常是在初始测试期间捕获的开发错误。|
|server\_error | 服务器遇到意外的错误。 | 重试请求。这些错误可能是临时状态导致的。客户端应用程序可以向用户说明，其响应由于临时错误而延迟。 |
| temporarily\_unavailable | 服务器暂时繁忙，无法处理请求。 | 重试请求。客户端应用程序可以向用户说明，其响应由于临时状态而延迟。 |
| invalid\_resource |目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。| 这表示未在租户中配置该资源（如果存在）。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |

## <a name="validating-the-id-token"></a>验证 id\_token
仅接收 id\_token 不足以验证用户，必须身份验证 id\_token 签名，并按照应用的要求验证令牌中的声明。v2.0 终结点使用 [JSON Web 令牌 (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 和公钥加密对令牌进行签名并验证其是否有效。

可以选择验证客户端代码中的 `id_token`，但是常见的做法是将 `id_token` 发送到后端服务器，并在那里执行验证。验证 id\_token 的签名后，就有几项声明需要验证。有关详细信息，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)，包括[验证令牌](/documentation/articles/active-directory-v2-tokens/#validating-tokens/)和[有关签名密钥滚动更新的重要信息](/documentation/articles/active-directory-v2-tokens/#validating-tokens/)。我们建议利用库来分析和验证令牌 - 对于大多数语言和平台至少有一个可用。
<!--TODO: Improve the information on this-->

你可能还希望根据自己的方案验证其他声明。一些常见的验证包括：

- 确保用户/组织已注册应用。
- 确保用户拥有正确的授权/权限
- 确保身份验证具有一定的强度，例如多重身份验证。

有关 id\_token 中声明的详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。

完全验证 id\_token 后，即可开始与用户的会话，并使用 id\_token 中的声明来获取应用中的用户相关信息。此信息可以用于显示、记录和授权，等等。

## 发送注销请求

v2.0 终结点目前不支持 OpenIdConnect `end_session_endpoint`。这意味着应用程序无法向 v2.0 终结点发送请求，因而无法结束用户会话并清除 v2.0 终结点设置的 Cookie。若要将用户注销，应用只需结束自身的用户会话，并完整地将用户会话留给 v2.0 终结点即可。下次用户尝试登录时，将看到列出其活动登录帐户的“选择帐户”页面。在该页面上，用户可以选择注销任一帐户，结束 v2.0 终结点的会话。

<!--

When you wish to sign the user out of the app, it is not sufficient to clear your app's cookies or otherwise end the session with the user.  You must also redirect the user to the v2.0 endpoint for sign out.  If you fail to do so, the user will be able to re-authenticate to your app without entering their credentials again, because they will have a valid single sign-on session with the v2.0 endpoint.

You can simply redirect the user to the `end_session_endpoint` listed in the OpenID Connect metadata document:


	GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
	post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F


| Parameter | | Description |
| ----------------------- | ------------------------------- | ------------ |
| post_logout_redirect_uri | recommended | The URL which the user should be redirected to after successful logout.  If not included, the user will be shown a generic message by the v2.0 endpoint.  |

-->


## 协议图 - 令牌获取
许多 Web 应用不仅需要将用户登录，而且还要代表该用户使用 OAuth 来访问 Web 服务。此方案合并了用于对用户进行身份验证的 OpenID Connect，同时将获取 authorization\_code，用于通过 OAuth 授权代码流来获取 access\_tokens。

完整的 OpenID Connect 登录和令牌获取流如下所示 - 下面详细描述了每个步骤。

![OpenId Connect Swimlanes](./media/active-directory-v2-flows/convergence_scenarios_webapp_webapi.png)  


## 获取访问令牌
若要获取访问令牌，需要稍微修改上述登录请求：


	// Line breaks for legibility only

	GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e		// Your registered Application Id
	&response_type=id_token%20code
	&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F 	  // Your registered Redirect Uri, url encoded
	&response_mode=form_post						      // 'query', 'form_post', or 'fragment'
	&scope=openid%20                                      // Include both 'openid' and scopes your app needs  
	offline_access%20										 
	https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
	&state=12345						 				 // Any value, provided by your app
	&nonce=678910										 // Any value, provided by your app


> [AZURE.TIP] 单击下面的链接以执行此请求！ 登录之后，你的浏览器应重定向至地址栏中具有 `id_token` 和 `code` 的 `https://localhost/myapp/`。请注意，此请求会使用 `response_mode=query`（仅用于教程）。建议使用 `response_mode=form_post`。
    <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token%20code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

通过在请求中包含权限范围并使用 `response_type=id_token code`，v2.0 终结点可确保用户已经同意 `scope` 查询参数中指示的权限，并且将授权代码返回到应用以交换访问令牌。

#### 成功的响应
使用 `response_mode=form_post` 的成功响应如下所示：


	POST /myapp/ HTTP/1.1
	Host: localhost
	Content-Type: application/x-www-form-urlencoded

	id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&state=12345


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| id\_token | 应用程序请求的 id\_token。可以使用 id\_token 验证用户的标识，并以用户身份开始会话。有关 id\_token 及其内容的更多详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |
| 代码 | 应用程序请求的 authorization\_code。应用程序可以使用授权代码请求目标资源的访问令牌。Authorization\_codes 的生存期很短，通常约 10 分钟后即过期。 |
| state | 如果请求中包含状态参数，响应中就应该出现相同的值。应用程序应该验证请求和响应中的状态值是否完全相同。 |

#### 错误响应
错误响应可能也发送到 `redirect_uri`，让应用可以适当地处理：


	POST /myapp/ HTTP/1.1
	Host: localhost
	Content-Type: application/x-www-form-urlencoded

	error=access_denied&error_description=the+user+canceled+the+authentication


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |

有关可能的错误代码的描述及其建议的客户端操作，请参阅[授权终结点错误的错误代码](#error-codes-for-authorization-endpoint-errors)。

获取授权 `code` 和 `id_token` 之后，可以将用户登录，并代表他们获取访问令牌。若要将用户登录，必须确切地按[上面](#validating-the-id-token)所述验证 `id_token`。若要获取访问令牌，可以遵循 [OAuth 协议文档](/documentation/articles/active-directory-v2-protocols-oauth-code/#request-an-access-token/)中所述的步骤。

<!---HONumber=Mooncake_1031_2016-->
