
<properties
	pageTitle="Azure AD v2.0 OAuth 客户端凭据流 | Azure"
	description="使用 Azure AD 的 OAuth 2.0 身份验证协议实现构建 Web 应用程序。"
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
	ms.date="09/26/2016"
	wacn.date="10/31/2016"
	ms.author="dastrock"/>  


# v2.0 协议 - OAuth 2.0 客户端凭据流

通过 [OAuth 2.0 客户端凭据授予](http://tools.ietf.org/html/rfc6749#section-4.4)（有时称为“双重 OAuth”），可以使用应用程序的标识来访问 Web 托管的资源。它通常用于必须在后台运行的服务器到服务器交互，不需要最终用户立即干预。此类应用程序通常称为**守护程序**或**服务帐户**。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

在较典型的“三重 OAuth”中，客户端应用程序有权代表特定用户访问资源。该权限通常在[许可](/documentation/articles/active-directory-v2-scopes/)过程中由用户**委托**给应用程序。但是，在客户端凭据流中，权限**直接**授予应用程序本身。当应用向资源出示令牌时，该资源将强制要求该应用本身拥有执行某个操作的授权 - 而不是某个用户拥有授权。

## 获取直接授权 
应用通常通过两种方式接收直接授权来访问资源：通过资源上的访问控制列表，或者通过 Azure AD 中的应用程序权限分配。资源可选择其他多种方式向其客户端授权，每个资源服务器可以选择最适合其应用程序的方式。以下两种方法在 Azure AD 中很常见，建议用于想要执行客户端凭据流的客户端和资源。

### 访问控制列表
给定的资源提供程序可根据它所知的应用程序 ID 列表，强制实施授权检查并授予特定级别的访问权限。当资源从 v2.0 终结点接收令牌时，可以将此令牌解码，并从 `appid` 和 `iss` 声明中提取客户端的应用程序 ID。然后，可将该应用程序与资源自身维护的某个访问控制列表 (ACL) 进行比较。不同资源的访问控制列表粒度级和方法可能大不相同。

此类 ACL 的常见用例包括 Web 应用程序或 Web API 的测试运行器。Web API 可能只将其部分权限授予各种客户端。但是，若要在 API 上运行端到端测试，则会创建测试客户端，以便从 v2.0 终结点获取令牌并将令牌发送到 API。然后，API 可将测试客户端的应用程序 ID 加入 ACL，获取对 API 整个功能的完全访问权限。请注意，如果服务中有此类列表，应确保不仅要验证调用方的 `appid`，并且还要验证令牌的 `iss` 是否可信。

对于需要使用个人 Microsoft 帐户访问使用者用户所拥有的数据的向导和服务帐户而言，这种授权类型很常见。对于组织拥有的数据，建议通过应用程序权限获取必要的授权。

### 应用程序权限
在不使用 ACL 的情况下，API 可以公开一组可授予应用程序的**应用程序权限**。组织管理员将应用程序权限授予某个应用程序，该权限只可用于访问该组织与其员工拥有的数据。例如，Microsoft Graph 公开多个应用程序权限：

- 读取所有邮箱中的邮件
- 在所有邮箱中读取和写入邮件
- 以任何用户的身份发送邮件
- 读取目录数据
- [更多](https://graph.microsoft.io)

若要在应用中获取这些权限，可执行以下步骤。

#### 在应用注册门户中请求权限

- 在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList) 中导航到你的应用程序，或[创建一个应用](/documentation/articles/active-directory-v2-app-registration/)（如果没有）。需要确保应用程序至少创建了一个应用程序机密。
- 找到“直接应用程序权限”部分并添加应用所需的权限。
- 请务必**保存**应用注册

#### 建议：让用户登录到该应用

在构建使用应用程序权限的应用程序时，应用通常需要一个页面/查看，使管理员能够审批应用的权限。此页面可以是应用注册流的一部分、应用设置的一部分，或专用“连接”流的一部分。在许多情况下，合理的结果是应用只在用户使用工作或学校 Microsoft 帐户登录之后才显示此“连接”视图。

将用户登录到应用后，可以识别用户所属的组织，然后要求他们审批应用程序权限。尽管在严格意义上不需要这样做，但有助于为组织用户带来更直观的体验。若要让用户登录，请遵循 [v2.0 协议教程](/documentation/articles/active-directory-v2-protocols/)。

#### 向目录管理员请求权限

准备向公司管理员请求权限时，可以将用户重定向到 v2.0 **管理员许可终结点**。


	// Line breaks for legibility only
	
	GET https://login.microsoftonline.com/{tenant}/adminconsent?
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e
	&state=12345
	&redirect_uri=http://localhost/myapp/permissions



	// Pro Tip: Try pasting the below request in a browser!



	https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions


| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必填 | 要向其请求权限的目录租户。可以使用 guid 或友好名称格式提供。如果不知道用户属于哪个租户并想要让他们登录到任一租户，请使用 `common`。 |
| client\_id | 必填 | 注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| redirect\_uri | 必填 | 要向其发送响应，供应用处理的 redirect\_uri。它必须完全符合在门户中注册的 redirect\_uris 之一，否则必须经过 url 编码并可以包含其他路径段。 |
| state | 建议 | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。该状态用于对发出身份验证请求出现之前，有关用户在应用中的状态的信息（例如前面所在的页面或视图）编码。 |

此时，Azure AD 将强制要求只有租户管理员可以登录来完成请求。系统将要求管理员审批在注册门户中针对应用请求的所有直接应用程序权限。

##### 成功的响应
如果管理员批准了应用程序的权限，成功响应如下：


	GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True


| 参数 | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 向应用程序授予所请求权限的目录租户（采用 guid 格式）。 |
| state | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。该状态用于对发出身份验证请求出现之前，有关用户在应用中的状态的信息（例如前面所在的页面或视图）编码。 |
| admin\_consent | 将设置为 `True`。 |


##### 错误响应
如果管理员未批准了应用程序的权限，失败响应如下：


	GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request


| 参数 | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别错误根本原因的具体错误消息。 |

从应用预配终结点收到成功响应后，应用便已获得它所请求的直接应用程序权限。现在可以继续请求所需资源的令牌。

## 获取令牌
获取应用程序的必要授权后，可以继续获取 API 的访问令牌。若要使用客户端凭据授予获取令牌，请将 POST 请求发送到 `/token` v2.0 终结点：


	POST /common/oauth2/v2.0/token HTTP/1.1
	Host: login.microsoftonline.com
	Content-Type: application/x-www-form-urlencoded
	
	client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials


	curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/common/oauth2/v2.0/token'


| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| client\_id | 必填 | 注册门户 ([apps.dev.microsoft.com](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)) 分配给应用的应用程序 ID。 |
| 作用域 | 必填 | 在此请求中针对 `scope` 参数传递的值应该是所需资源的资源标识符（应用 ID URI），并附有 `.default` 后缀。因此，在给定的 Microsoft Graph 示例中，该值应是 `https://graph.microsoft.com/.default`。此值通知 v2.0 终结点有关为应用配置的所有直接应用程序权限，终结点应该针对所需资源相关的对象颁发令牌。 |
| client\_secret | 必填 | 在注册门户中为应用生成的应用程序机密。 |
| grant\_type | 必填 | 必须是 `client_credentials`。 | 

#### 成功的响应
成功响应如下所示：


	{
	  "token_type": "Bearer",
	  "expires_in": 3599,
	  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
	}


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| access\_token | 请求的访问令牌。应用程序可以使用此令牌来验证受保护的资源，例如 Web API。 |
| token\_type | 指示令牌类型值。Azure AD 支持的唯一类型是 `Bearer`。 |
| expires\_in | 访问令牌的有效期（以秒为单位）。 |

#### 错误响应
错误响应如下所示：


	{
	  "error": "invalid_scope",
	  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
	  "error_codes": [
	    70011
	  ],
	  "timestamp": "2016-01-09 02:02:12Z",
	  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
	  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
	}


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error\_codes | 帮助诊断的 STS 特定错误代码列表。 |
| timestamp | 发生错误的时间。 |
| trace\_id | 帮助诊断的请求唯一标识符。 |
| correlation\_id | 帮助跨组件诊断的请求唯一标识符。 |

## 使用令牌
获取令牌后，可以使用该令牌对资源发出请求。当令牌过期时，只需向 `/token` 终结点重复该请求，即可获取全新的访问令牌。


	GET /v1.0/me/messages
	Host: https://graph.microsoft.com
	Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...



	// Pro Tip: Try the below command out! (but replace the token with your own)



	curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q" 'https://graph.microsoft.com/v1.0/me/messages'


## 代码示例
若要查看使用管理员许可终结点实现 client\_credentials 授予的应用程序示例，请参阅 [v2.0 daemon code sample](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)（v2.0 守护程序代码示例）。

<!---HONumber=Mooncake_1024_2016-->
