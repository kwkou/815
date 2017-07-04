

<properties
    pageTitle="Azure AD v2.0 OAuth 授权代码流 | Azure"
    description="使用 Azure AD 的 OAuth 2.0 身份验证协议实现构建 Web 应用程序。"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="ae1d7d86-7098-468c-aa32-20df0a10ee3d"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/13/2017"
    ms.author="dastrock" />  


# v2.0 协议 - OAuth 2.0 授权代码流
OAuth 2.0 授权代码授予可用于设备上所安装的应用，以访问受保护的资源，例如 Web API。使用应用模型 v2.0 的 OAuth 2.0 实现，可以将登录名及 API 访问添加到移动应用和桌面应用。本指南与语言无关，介绍在不使用我们任何开源库的情况下，如何发送和接收 HTTP 消息。

<!-- TODO: Need link to libraries -->

> [AZURE.NOTE]
v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。
> 
> 

有关 OAuth 2.0 授权代码流的说明，请参阅 [OAuth 2.0 规范第 4.1 部分](http://tools.ietf.org/html/rfc6749)。它用于在大部分的应用类型（包括 [Web 应用](/documentation/articles/active-directory-v2-flows/#web-apps/)和[本机安装的应用](/documentation/articles/active-directory-v2-flows/#mobile-and-native-apps/)）中执行身份验证与授权。它让应用能够安全地获取 access\_token，用于访问以 v2.0 终结点保护的资源。

## 协议图
从较高层面讲，本机/移动应用程序的整个身份验证流有点类似于：

![OAuth 授权代码流](./media/active-directory-v2-flows/convergence_scenarios_native.png)

## 请求授权代码
授权代码流始于客户端将用户定向到 `/authorize` 终结点。在这项请求中，客户端指示必须向用户获取的权限：


	// Line breaks for legibility only
	
	https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e
	&response_type=code
	&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
	&response_mode=query
	&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
	&state=12345


> [AZURE.TIP]
单击下面的链接以执行此请求！ 登录之后，浏览器应重定向至地址栏中具有 `code` 的 `https://localhost/myapp/`。
<a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&state=12345" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>
> 
> 

| 参数 | | 说明 |
| --- | --- | --- |
| tenant |必填 |请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints/)。 |
| client\_id |必填 |注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| response\_type |必填 |必须包括授权代码流的 `code`。 |
| redirect\_uri |建议 |应用的 redirect\_uri，应用可在此发送及接收身份验证响应。必须完全符合在门户中注册的其中一个 redirect\_uri，否则必须是编码的 url。对于本机和移动应用，应使用默认值 `https://login.microsoftonline.com/common/oauth2/nativeclient`。 |
| scope |必填 |希望用户同意的[范围](/documentation/articles/active-directory-v2-scopes/)的空格分隔列表。 |
| response\_mode |建议 |指定将生成的令牌送回到应用所应该使用的方法。可以是 `query` 或 `form_post`。 |
| state |建议 |同样随令牌响应返回的请求中所包含的值。可以是想要的任何内容的字符串。随机生成的唯一值通常用于[防止跨站点请求伪造攻击](http://tools.ietf.org/html/rfc6749#section-10.12)。该 state 也用于在身份验证请求出现之前，于应用中编码用户的状态信息，例如之前所在的网页或视图。 |
| prompt |可选 |表示需要的用户交互类型。目前仅有的有效值为“login”、“none”和“consent”。`prompt=login` 将强制用户在该请求上输入凭据，取消单一登录。`prompt=none` 则相反 - 它将确保不对用户显示任何交互式提示。如果请求无法通过单一登录以无提示方式完成，v2.0 终结点将返回错误。`prompt=consent` 将在用户登录之后触发 OAuth 同意对话框，要求用户向应用授予权限。 |
| login\_hint |可选 |如果事先知道其用户名称，可用于预先填充用户登录页面的用户名称/电子邮件地址字段。通常，应用将在重新身份验证期间使用此参数，并且已经使用 `preferred_username` 声明从前次登录提取用户名。 |
| domain\_hint |可选 |可以是 `consumers` 或 `organizations`。如果包含，用户则会跳过 v2.0 登录页面上基于电子邮件的发现过程，从而带来稍显流畅的用户体验。通常，应用将在重新身份验证期间使用此参数，方法是从前次登录提取 `tid`。如果 `tid` 声明值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，则应使用 `domain_hint=consumers`，否则使用 `domain_hint=organizations`。 |

此时，将请求用户输入凭据并完成身份验证。v2.0 终结点也将确保用户已经同意 `scope` 查询参数中指示的权限。如果用户未曾同意这些权限的任何一项，则将请求用户同意请求的权限。[此处提供了权限、同意与多租户应用](/documentation/articles/active-directory-v2-scopes/)的详细信息。

用户经过身份验证并同意后，v2.0 终结点将使用 `response_mode` 参数中指定的方法，将响应返回到位于所指示的 `redirect_uri` 的应用。

#### 成功的响应
使用 `response_mode=query` 的成功响应如下所示：


	GET https://login.microsoftonline.com/common/oauth2/nativeclient?
	code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
	&state=12345


| 参数 | 说明 |
| --- | --- |
| code |应用请求的 authorization\_code。应用可以使用授权代码请求目标资源的访问令牌。Authorization\_code 的生存期很短，通常约 10 分钟后即会过期。 |
| state |如果请求中包含 state 参数，响应中就应该出现相同的值。应用应该验证请求和响应中的 state 值是否完全相同。 |

#### 错误响应
错误响应可能也发送到 `redirect_uri`，让应用可以适当地处理：


	GET https://login.microsoftonline.com/common/oauth2/nativeclient?
	error=access_denied
	&error_description=the+user+canceled+the+authentication


| 参数 | 说明 |
| --- | --- |
| error |可用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description |可帮助开发人员识别身份验证错误根本原因的特定错误消息。 |

#### 授权终结点错误的错误代码
下表描述了可在错误响应的 `error` 参数中返回的各个错误代码。

| 错误代码 | 说明 | 客户端操作 |
| --- | --- | --- |
| invalid\_request |协议错误，例如，缺少必需的参数。 |修复并重新提交请求。这通常是在初始测试期间捕获的开发错误。 |
| unauthorized\_client |不允许客户端应用程序请求授权代码。 |客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| access\_denied |资源所有者拒绝了许可 |客户端应用程序可以通知用户，除非用户许可，否则无法继续。 |
| unsupported\_response\_type |授权服务器不支持请求中的响应类型。 |修复并重新提交请求。这通常是在初始测试期间捕获的开发错误。 |
| server\_error |服务器遇到意外的错误。 |重试请求。这些错误可能是临时状况导致的。客户端应用程序可以向用户说明，其响应由于临时错误而延迟。 |
| temporarily\_unavailable |服务器暂时繁忙，无法处理请求。 |重试请求。客户端应用程序可以向用户说明，其响应由于临时状况而延迟。 |
| invalid\_resource |目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。 |这表示未在租户中配置该资源（如果存在）。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |

## 请求访问令牌 <a name="request-an-access-token"></a>
你已获取 authorization\_code 并获得用户授权，现在可以将 `POST` 请求发送到 `/token` 终结点，兑换 `code` 以获取所需资源的 `access_token`：


	// Line breaks for legibility only
	
	POST /{tenant}/oauth2/v2.0/token HTTP/1.1
	Host: https://login.microsoftonline.com
	Content-Type: application/x-www-form-urlencoded
	
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e
	&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
	&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
	&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
	&grant_type=authorization_code
	&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps


> [AZURE.TIP]
尝试在 Postman 中执行此请求！ （别忘了替换 `code`）[![在 Postman 中运行](./media/active-directory-v2-protocols-oauth-code/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)
> 
> 

| 参数 | | 说明 |
| --- | --- | --- |
| tenant |必填 |请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints/)。 |
| client\_id |必填 |注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| grant\_type |必填 |必须是授权代码流的 `authorization_code`。 |
| scope |必填 |范围的空格分隔列表。在此阶段请求的范围必须相当于或为第一个阶段中所请求的范围子集。如果此请求中指定的范围遍及多个资源服务器，v2.0 终结点将返回第一个范围内所指定资源的令牌。有关范围的更详细说明，请参阅[权限、同意和范围](/documentation/articles/active-directory-v2-scopes/)。 |
| code |必填 |在流的第一个阶段获取的 authorization\_code。 |
| redirect\_uri |必填 |用于获取 authorization\_code 的相同 redirect\_uri 值。 |
| client\_secret |必填（对于 Web 应用） |在应用注册门户中为应用创建的应用程序机密。不应用于本机应用，因为设备无法可靠地存储 client\_secret。对于能够将 client\_secret 安全地存储在服务器端的 Web 应用和 Web API 为必填参数。 |

#### 成功的响应
成功的令牌响应如下：

	
	{
		"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
		"token_type": "Bearer",
		"expires_in": 3599,
		"scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
		"refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
		"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
	}

| 参数 | 说明 |
| --- | --- |
| access\_token |请求的访问令牌。应用可以使用此令牌来验证受保护的资源，例如 Web API。 |
| token\_type |指示令牌类型值。Azure AD 唯一支持的类型是 Bearer |
| expires\_in |访问令牌的有效期（以秒为单位）。 |
| scope |access\_token 有效的范围。 |
| refresh\_token |OAuth 2.0 刷新令牌。应用可以使用此令牌，在当前的访问令牌过期之后获取其他访问令牌。Refresh\_token 的生存期很长，而且可以用于延长保留资源访问权限的时间。有关更多详细信息，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |
| id\_token |无符号 JSON Web 令牌 (JWT)。应用可以 base64Url 解码此令牌的段，以请求已登录用户的相关信息。应用可以缓存并显示值，但不应依赖于这些值来获取任何授权或安全边界。有关 id\_token 的详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |

#### 错误响应
错误响应如下所示：

	
	{
	  "error": "invalid_scope",
	  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
	  "error_codes": [
	    70011
	  ],
	  "timestamp": "2016-01-09 02:02:12Z",
	  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
	  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
	}
	

| 参数 | 说明 |
| --- | --- |
| error |可用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description |可帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error\_codes |可帮助诊断的 STS 特定错误代码列表。 |
| timestamp |发生错误的时间。 |
| trace\_id |可帮助诊断的请求唯一标识符。 |
| correlation\_id |可帮助跨组件诊断的请求唯一标识符。 |

#### 令牌终结点错误的错误代码 <a name="error-codes-for-token-endpoint-errors"></a>

| 错误代码 | 说明 | 客户端操作 |
| --- | --- | --- |
| invalid\_request |协议错误，例如，缺少必需的参数。 |修复并重新提交请求。 |
| invalid\_grant |授权代码无效或已过期。 |请尝试对 `/authorize` 终结点发出新请求 |
| unauthorized\_client |经过身份验证的客户端无权使用此权限授予类型。 |客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| invalid\_client |客户端身份验证失败。 |客户端凭据无效。若要修复，应用程序管理员应更新凭据。 |
| unsupported\_grant\_type |授权服务器不支持权限授予类型。 |更改请求中的授权类型。这种类型的错误应该只在开发过程中发生，并且应该在初始测试过程中检测到。 |
| invalid\_resource |目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。 |这表示未在租户中配置该资源（如果存在）。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| interaction\_required |请求需要用户交互。例如，需要额外的身份验证步骤。 |使用同一资源重试请求。 |
| temporarily\_unavailable |服务器暂时繁忙，无法处理请求。 |重试请求。客户端应用程序可以向用户说明，其响应由于临时状况而延迟。 |

## 使用访问令牌
你已经成功获取 `access_token`，现在可以通过在 `Authorization` 标头中包含令牌，在 Web API 的请求中使用令牌。

> [AZURE.TIP]
在 Postman 中执行此请求！ （先替换 `Authorization` 标头）[![在 Postman 中运行](./media/active-directory-v2-protocols-oauth-code/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)
> 
> 

	GET /v1.0/me/messages
	Host: https://graph.microsoft.com
	Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
	

## 刷新访问令牌
Access\_token 生存期很短，过期后必须刷新才能继续访问资源。为此，可以向 `/token` 终结点提交另一个 `POST` 请求，但这次要提供 `refresh_token` 而不是 `code`：


	// Line breaks for legibility only
	
	POST /{tenant}/oauth2/v2.0/token HTTP/1.1
	Host: https://login.microsoftonline.com
	Content-Type: application/x-www-form-urlencoded
	
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e
	&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
	&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
	&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
	&grant_type=refresh_token
	&client_secret=JqQX2PNo9bpM0uEihUPzyrh	  // NOTE: Only required for web apps


> [AZURE.TIP]
尝试在 Postman 中执行此请求！ （别忘了替换 `refresh_token`）
[![在 Postman 中运行](./media/active-directory-v2-protocols-oauth-code/runInPostman.png)](https://app.getpostman.com/run-collection/8f5715ec514865a07e6a)
> 
> 

| 参数 | | 说明 |
| --- | --- | --- |
| tenant |必填 |请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints/)。 |
| client\_id |必填 |注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| grant\_type |必填 |必须是授权代码流的此阶段的 `refresh_token`。 |
| scope |必填 |范围的空格分隔列表。在此阶段请求的范围必须等效于或者为原始 authorization\_code 请求阶段中所请求的范围子集。如果此请求中指定的范围遍及多个资源服务器，v2.0 终结点将返回第一个范围内所指定资源的令牌。有关范围的更详细说明，请参阅[权限、同意和范围](/documentation/articles/active-directory-v2-scopes/)。 |
| refresh\_token |必填 |在流的第二个阶段获取的 refresh\_token。 |
| redirect\_uri |必填 |用于获取 authorization\_code 的相同 redirect\_uri 值。 |
| client\_secret |必填（对于 Web 应用） |在应用注册门户中为应用创建的应用程序机密。不应用于本机应用，因为设备无法可靠地存储 client\_secret。对于能够将 client\_secret 安全地存储在服务器端的 Web 应用和 Web API 为必填参数。 |

#### 成功的响应
成功的令牌响应如下：

	
	{
		"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
		"token_type": "Bearer",
		"expires_in": 3599,
		"scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
		"refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
		"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
	}

| 参数 | 说明 |
| --- | --- |
| access\_token |请求的访问令牌。应用可以使用此令牌来验证受保护的资源，例如 Web API。 |
| token\_type |指示令牌类型值。Azure AD 唯一支持的类型是 Bearer |
| expires\_in |访问令牌的有效期（以秒为单位）。 |
| scope |access\_token 有效的范围。 |
| refresh\_token |新的 OAuth 2.0 刷新令牌。应该将旧刷新令牌替换为新获取的这个刷新令牌，以确保刷新令牌的有效期尽可能地长。 |
| id\_token |无符号 JSON Web 令牌 (JWT)。应用可以 base64Url 解码此令牌的段，以请求已登录用户的相关信息。应用可以缓存并显示值，但不应依赖于这些值来获取任何授权或安全边界。有关 id\_token 的详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |

#### 错误响应
	
	{
	  "error": "invalid_scope",
	  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
	  "error_codes": [
	    70011
	  ],
	  "timestamp": "2016-01-09 02:02:12Z",
	  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
	  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
	}


| 参数 | 说明 |
| --- | --- |
| error |可用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description |可帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error\_codes |可帮助诊断的 STS 特定错误代码列表。 |
| timestamp |发生错误的时间。 |
| trace\_id |可帮助诊断的请求唯一标识符。 |
| correlation\_id |可帮助跨组件诊断的请求唯一标识符。 |

有关错误代码的描述和建议的客户端操作，请参阅[令牌终结点错误的错误代码](#error-codes-for-token-endpoint-errors)。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update-->