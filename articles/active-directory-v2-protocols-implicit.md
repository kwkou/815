<properties
	pageTitle="Azure AD v2.0 隐式流 | Azure"
	description="使用单页面应用隐式流的 Azure AD v2.0 实现构建 Web 应用程序。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="msmbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>

# v2.0 协议 - 使用隐式流的 SPA
使用 v2.0 终结点，你可以让具有 Microsoft 的个人和工作/学校帐户的用户登录单一页面应用。主要在浏览器上运行的单一页面和其他 JavaScript 应用程序在身份验证时面临一些有趣的挑战：

- 这些应用程序的安全特征与传统的基于服务器的 Web 应用程序大不相同。
- 许多授权服务器与标识提供者不支持 CORS 请求。
- 重定向离开应用程序的完整网页浏览器变得对用户经验特别有侵略性。

对于这些应用程序（AngularJS、Ember.js、React.js 等），Azure AD 支持 OAuth 2.0 隐式授权流。有关隐式流的说明，请参阅 [OAuth 2.0 规范](http://tools.ietf.org/html/rfc6749#section-4.2)。其主要优点是它可让应用程序从 Azure AD 获取令牌，无需要执行后端服务器凭据交换。这可让应用登录用户、维护会话，并获取客户端 JavaScript 代码中所有其他 Web API 的令牌。使用隐式流时有几个重要的安全注意事项 - 特别是关于[客户端](http://tools.ietf.org/html/rfc6749#section-10.3)和[用户模拟](http://tools.ietf.org/html/rfc6749#section-10.3)。

如果你想要使用隐式流与 Azure AD 将身份验证添加到 JavaScript 应用，建议使用我们的开放源代码 JavaScript 库 [adal.js](https://github.com/AzureAD/azure-activedirectory-library-for-js)。[此处](/documentation/articles/active-directory-appmodel-v2-overview/#getting-started)有几篇 AngularJS 教程可以帮助你入门。

但是，如果你不想要使用单一页面应用程序中的库，而是自行发送协议消息，请遵循下面的常规步骤。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。
    
## 协议图
整个隐式登录流如下所示 - 下面将详细描述每个步骤。

![OpenId Connect Swimlanes](./media/active-directory-v2-flows/convergence_scenarios_implicit.png)

## 发送登录请求

若要一开始将用户登录应用程序，可以发送 [OpenID Connect](/documentation/articles/active-directory-v2-protocols-oidc/) 授权请求，以及从 v2.0 终结点获取 `id_token`：


		// Line breaks for legibility only
		
		https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&response_type=id_token+token
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&scope=openid%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
		&response_mode=fragment
		&state=12345
		&nonce=678910


> [AZURE.TIP] 单击下面的链接以执行此请求！ 登录之后，你的浏览器应重定向至地址栏中具有 `id_token` 的 `https://localhost/myapp/`。<a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token+token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=openid%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必填 | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints)。 |
| client\_id | 必填 | 注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com)) 分配给应用的应用程序 ID。 |
| response\_type | 必填 | 必须包含 OpenID Connect 登录的 `id_token`。也可以包含 response\_type `token`。此处使用 `token`，让应用能够立即从授权终结点接收访问令牌，而无需向授权终结点发出第二次请求。如果使用 `token` response\_type，`scope` 参数必须包含范围，以指出要对哪个资源发出令牌。 |
| redirect\_uri | 建议 | 应用程序的 redirect\_uri，应用程序可在此发送及接收身份验证响应。其必须完全符合在门户中注册的其中一个 redirect\_uris，否则必须是编码的 url。 |
| 作用域 | 必填 | 范围的空格分隔列表。对于 OpenID Connect，它必须包含范围 `openid`，该范围在同意 UI 中将转换为“将你登录”权限。（可选）你可能还想要包含 `email` 或 `profile` [范围](/documentation/articles/active-directory-v2-scopes/)，以获取对其他用户数据的访问权限。也可以在此请求中包含其他范围，以请求同意各种资源。 |
| response\_mode | 建议 | 指定将生成的令牌送回到应用程序所应该使用的方法。对于隐式流应该是 `fragment`。 |
| state | 建议 | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。随机生成的唯一值通常用于[防止跨站点请求伪造攻击](http://tools.ietf.org/html/rfc6749#section-10.12)。该状态也用于在身份验证请求出现之前，于应用程序中编码用户的状态信息，例如之前所在的网页或视图。 |
| nonce | 必填 | 由应用程序生成且包含在请求中的值，以声明方式包含在生成的 id\_token 中。应用程序接着便可确认此值，以减少令牌重新执行攻击。此值通常是随机的唯一字符串，可用以识别请求的来源。 |
| prompt | 可选 | 表示需要的用户交互类型。目前唯一的有效值为“login”、“none”和“consent”。`prompt=login` 强制用户在该请求上输入凭据，否定单一登录。`prompt=none` 则相反 — 它确保不对用户显示任何交互式提示。如果请求无法通过单一登录以无提示方式完成，v2.0 终结点将返回错误。`prompt=consent` 将在用户登录之后触发 OAuth 同意对话框，要求用户向应用授予权限。 |
| login\_hint | 可选 | 如果事先知道其用户名称，可用于预先填充用户登录页面的用户名称/电子邮件地址字段。通常，应用将在重新身份验证期间使用此参数，并且已经使用 `preferred_username` 声明从前次登录提取用户名。 |
| domain\_hint | 可选 | 可以是 `consumers` 或 `organizations` 之一。如果包含，它跳过用户在 v2.0 登录页面上经历的基于电子邮件的发现过程，导致稍微更加流畅的用户体验。通常，应用将在重新身份验证期间使用此参数，方法是从 id\_token 提取 `tid` 声明。如果 `tid` 声明值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，应该使用 `domain_hint=consumers`。否则使用 `domain_hint=organizations`。 |

此时，请求用户输入其凭据并完成身份验证。v2.0 终结点也将确保用户已经同意 `scope` 查询参数中指示的权限。如果用户未曾同意这些权限的任何一项，就请求用户同意请求的权限。[此处提供了权限、同意与多租户应用](/documentation/articles/active-directory-v2-scopes/)的详细信息。

用户经过身份验证并同意后，v2.0 终结点将使用 `response_mode` 参数中指定的方法，将响应返回到位于所指示的 `redirect_uri` 的应用。

#### 成功的响应

使用 `response_mode=fragment` 和 `response_type=id_token+token` 的成功响应如下所示（包含换行符以方便阅读）：


		GET https://localhost/myapp/#
		access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
		&token_type=Bearer
		&expires_in=3599
		&scope=https%3a%2f%2fgraph.microsoft.com%2fmail.read 
		&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
		&state=12345


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| access\_token | 如果 `response_type` 包含 `token`，则包含该参数。应用请求的访问令牌，在本例中为 Microsoft Graph 的访问令牌。访问令牌不得进行解码或检查，可被视为不透明的字符串。 |
| token\_type | 如果 `response_type` 包含 `token`，则包含该参数。始终为 `Bearer`。 |
| expires\_in | 如果 `response_type` 包含 `token`，则包含该参数。表示令牌有效的秒数（针对缓存目的）。 |
| 作用域 | 如果 `response_type` 包含 `token`，则包含该参数。表示 access\_token 的有效范围。 |
| id\_token | 应用程序请求的 id\_token。可以使用 id\_token 验证用户的标识，并以用户身份开始会话。有关 id\_token 及其内容的更多详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。 |
| state | 如果请求中包含状态参数，响应中就应该出现相同的值。应用程序应该验证请求和响应中的状态值是否完全相同。 |


#### 错误响应
错误响应可能也发送到 `redirect_uri`，让应用可以适当地处理：

		
		GET https://localhost/myapp/#
		error=access_denied
		&error_description=the+user+canceled+the+authentication


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |

## 验证 id\_token
仅接收 id\_token 不足以验证用户，必须身份验证 id\_token 签名，并按照应用的要求验证令牌中的声明。v2.0 终结点使用 [JSON Web 令牌 (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 和公钥加密对令牌进行签名并验证其是否有效。

可以选择验证客户端代码中的 `id_token`，但是常见的做法是将 `id_token` 发送到后端服务器，并在那里执行验证。验证 id\_token 的签名后，就有几项声明需要验证。有关详细信息，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)，包括[验证令牌](/documentation/articles/active-directory-v2-tokens/#validating-tokens)和[有关签名密钥滚动更新的重要信息](/documentation/articles/active-directory-v2-tokens/#validating-tokens)。我们建议利用库来分析和验证令牌 - 对于大多数语言和平台至少有一个可用。
<!--TODO: Improve the information on this-->

你可能还希望根据自己的方案验证其他声明。一些常见的验证包括：

- 确保用户/组织已注册应用。
- 确保用户拥有正确的授权/权限
- 确保身份验证具有一定的强度，例如多重身份验证。

有关 id\_token 中声明的详细信息，请参阅 [v2.0 应用模型令牌参考](/documentation/articles/active-directory-v2-tokens/)。

完全验证 id\_token 后，即可开始与用户的会话，并使用 id\_token 中的声明来获取应用中的用户相关信息。此信息可以用于显示、记录和授权，等等。

## 获取访问令牌

你已经将用户登录到单页面应用，现在可以获取访问令牌以调用受到 Azure AD 保护的 Web API，例如 [Microsoft Graph](https://graph.microsoft.io)。即使你已使用 `token` response\_type 收到令牌，也仍可以使用此方法获取其他资源的令牌，而无需再次将用户重定向到登录页。

在正常的 OpenID Connect/OAuth 流中，可以通过对 v2.0 `/token` 终结点进行请求来实现此目的。但是，v2.0 终结点不支持 CORS 请求，因此进行 AJAX 调用以获取和刷新令牌是不可能的。相反，你可以在隐藏的 iframe 中使用隐式流，以获取其他 Web API 的新令牌：


		// Line breaks for legibility only
		
		https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&response_type=token
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment
		&state=12345&nonce=678910
		&prompt=none
		&domain_hint=organizations
		&login_hint=myuser@mycompany.com
		

> [AZURE.TIP] 请尝试将以下请求粘贴到浏览器中！（如果希望成功，请先修改 domain\_hint 和 login\_hint 值）


		https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&response_mode=fragment&state=12345&nonce=678910&prompt=none&domain_hint=organizations&login_hint=myuser@mycompany.com


| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必填 | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。允许的值为 `common`、`organizations`、`consumers` 和租户标识符。有关更多详细信息，请参阅[协议基础知识](/documentation/articles/active-directory-v2-protocols/#endpoints)。 |
| client\_id | 必填 | 注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com)) 分配给应用的应用程序 ID。 |
| response\_type | 必填 | 必须包含 OpenID Connect 登录的 `id_token`。还可以包含其他 response\_type，例如 `code`。 |
| redirect\_uri | 建议 | 应用程序的 redirect\_uri，应用程序可在此发送及接收身份验证响应。其必须完全符合在门户中注册的其中一个 redirect\_uris，否则必须是编码的 url。 |
| 作用域 | 必填 | 范围的空格分隔列表。若要获取令牌，请包含相应资源请求的所有[范围](/documentation/articles/active-directory-v2-scopes/)。 |
| response\_mode | 建议 | 指定将生成的令牌送回到应用程序所应该使用的方法。可以是 `query`、`form_post` 或 `fragment` 之一。 |
| state | 建议 | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。随机生成的唯一值通常用于防止跨站点请求伪造攻击。该状态也用于在身份验证请求出现之前，于应用程序中编码用户的状态信息，例如之前所在的网页或视图。 |
| nonce | 必填 | 由应用程序生成且包含在请求中的值，以声明方式包含在生成的 id\_token 中。应用程序接着便可确认此值，以减少令牌重新执行攻击。此值通常是随机的唯一字符串，可用以识别请求的来源。 |
| prompt | 必填 | 若要刷新并获取隐藏 iframe 中的令牌，应该使用 `prompt=none` 以确保 iframe 不会在 v2.0 登录页面上挂起，而是立即返回。 |
| login\_hint | 必填 | 若要刷新并获取隐藏 iframe 中的令牌，必须在此提示中包含用户的用户名，以便区分用户在特定时间点可能具有的多个会话。可以使用 `preferred_username` 声明从前次登录提取用户名。 |
| domain\_hint | 必填 | 可以是 `consumers` 或 `organizations` 之一。若要刷新并获取隐藏 iframe 中的令牌，必须在请求中包含 domain\_hint。应该从前次登录的 id\_token 提取 `tid` 声明，以确定要使用哪个值。如果 `tid` 声明值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，应该使用 `domain_hint=consumers`。否则使用 `domain_hint=organizations`。 |

由于 `prompt=none` 参数，此请求将立即成功或立即失败，并且返回给应用程序。使用 `response_mode` 参数中指定的方法，将成功的响应发送到位于所指示的 `redirect_uri` 的应用。

#### 成功的响应
使用 `response_mode=fragment` 的成功响应如下所示：


		GET https://localhost/myapp/#
		access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
		&state=12345
		&token_type=Bearer
		&expires_in=3600
		&scope=https%3A%2F%2Fgraph.windows.net%2Fdirectory.read


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| access\_token | 应用请求的令牌。 |
| token\_type | 始终为 `Bearer`。 |
| state | 如果请求中包含状态参数，响应中就应该出现相同的值。应用程序应该验证请求和响应中的状态值是否完全相同。 |
| expires\_in | 访问令牌的有效期（以秒为单位）。 |
| 作用域 | 访问令牌有效的范围。 |

#### 错误响应
错误响应可能也发送到 `redirect_uri`，让应用可以适当地处理。如果 `prompt=none`，预期的错误将为：

		
		GET https://localhost/myapp/#
		error=user_authentication_required
		&error_description=the+request+could+not+be+completed+silently


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |

如果在 iframe 请求中收到此错误，用户必须再次以交互方式登录以检索新令牌。可以选择对应用程序合理的任何方式处理这种情况。

## 刷新令牌

`id_token` 和 `access_token` 很快就会过期，因此应用必须准备好定期刷新这些令牌。若要刷新任一类型的令牌，可以通过使用 `prompt=none` 参数控制 Azure AD 的行为，来执行上面的同一隐藏 iframe 请求。如果想要接收新的 `id_token`，请务必使用 `response_type=id_token` 和 `scope=openid`，以及 `nonce` 参数。


## 发送注销请求

v2.0 终结点目前不支持 OpenIdConnect `end_session_endpoint`。这意味着应用程序无法向 v2.0 终结点发送请求，因而无法结束用户会话并清除 v2.0 终结点设置的 Cookie。
若要将用户注销，应用只需结束自身的用户会话，并完整地将用户会话留给 v2.0 终结点即可。下次用户尝试登录时，将看到列出其活动登录帐户的“选择帐户”页面。在该页面上，用户可以选择注销任一帐户，结束 v2.0 终结点的会话。

<!--

When you wish to sign the user out of the  app, it is not sufficient to clear your app's cookies or otherwise end the session with the user.  You must also redirect the user to the v2.0 endpoint for sign out.  If you fail to do so, the user will be able to re-authenticate to your app without entering their credentials again, because they will have a valid single sign-on session with the v2.0 endpoint.

You can simply redirect the user to the `end_session_endpoint` listed in the OpenID Connect metadata document:

		
		GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
		post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F


| Parameter | | Description |
| ----------------------- | ------------------------------- | ------------ |
| post_logout_redirect_uri | recommended | The URL which the user should be redirected to after successful logout.  If not included, the user will be shown a generic message by the v2.0 endpoint.  |

-->
<!---HONumber=Mooncake_0516_2016-->