
<properties
    pageTitle="Azure Active Directory v2.0 和 OAuth 2.0 客户端凭据流 | Azure"
    description="使用 OAuth 2.0 身份验证协议的 Azure AD 实现构建 Web 应用程序。"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="9b7cfbd7-f89f-4e33-aff2-414edd584b07"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/13/2017"
    ms.author="dastrock" />  


# Azure Active Directory v2.0 和 OAuth 2.0 客户端凭据流
可以使用 [OAuth 2.0 客户端凭据授予](http://tools.ietf.org/html/rfc6749#section-4.4)（有时称为*双重 OAuth*）通过应用程序的标识来访问 Web 托管资源。这种授权通常用于必须在后台运行的服务器间交互，不需要立即与用户交互。此类应用程序通常称为*守护程序*或*服务帐户*。

> [AZURE.NOTE]
v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。
> 
> 

在较典型的*三重 OAuth* 中，客户端应用程序有权代表特定用户访问资源。该权限通常在[许可](/documentation/articles/active-directory-v2-scopes/)过程中由用户委托给应用程序。但是，在客户端凭据流中，权限直接授予应用程序本身。应用向资源出示令牌时，该资源强制要求应用本身拥有执行操作的授权，而不是用户拥有授权。

## 协议图
整个客户端凭据流类似于下图。本文稍后将介绍每个步骤。

![客户端凭据流](./media/active-directory-v2-flows/convergence_scenarios_client_creds.png)  


## 获取直接授权
应用常通过以下两种方式之一接收直接授权来访问资源：通过资源上的访问控制列表 (ACL)，或者通过 Azure Active Directory (Azure AD) 中的应用程序权限分配。这两种方法在 Azure AD 中很常见，建议用于执行客户端凭据流的客户端和资源。但是，资源可以选择以其他方式向其客户端授权。每个资源服务器可选择最适合其应用程序的方法。

### 访问控制列表
资源提供程序可根据它所知并对其授予特定级别访问权限的应用程序 ID 列表，强制实施授权检查。资源从 v2.0 终结点接收令牌时，可以将此令牌解码，并从 `appid` 和 `iss` 声明中提取客户端的应用程序 ID。然后将应用程序与它所维护的 ACL 相比较。ACL 的粒度和方法可能因资源不同而有较大差异。

常见用例是使用 ACL 对 Web 应用程序或 Web API 运行测试。Web API 可能仅向特定客户端授予部分完全权限。若要在 API 上运行端到端测试，请创建测试客户端，以便从 v2.0 终结点获取令牌并将令牌发送到 API。然后，API 会检查测试客户端应用程序 ID 的 ACL，以获取对 API 整个功能的完全访问权限。如果使用这种 ACL，不仅需要验证调用方的 `appid` 值，还要验证令牌的 `iss` 值是否受信任。

对于需要访问使用者用户（拥有个人 Microsoft 帐户）所拥有数据的守护程序和服务帐户而言，这种授权类型很常见。对于组织拥有的数据，建议通过应用程序权限获取必要的授权。

### 应用程序权限
可使用 API 公开一组应用程序权限，而不是使用 ACL。应用程序权限由组织管理员向应用程序授予，并且只可用于访问该组织与其员工所拥有的数据。例如，Microsoft Graph 公开多个应用程序权限来执行以下操作：

- 读取所有邮箱中的邮件
- 在所有邮箱中读取和写入邮件
- 以任何用户的身份发送邮件
- 读取目录数据

有关应用程序权限的详细信息，请转到 [Microsoft Graph](https://graph.microsoft.io)。

若要在应用中使用应用程序权限，请执行后续部分所述的步骤。

#### 在应用注册门户中请求权限
1. 在[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)中转到你的应用程序，或[创建一个应用](/documentation/articles/active-directory-v2-app-registration/)（如果尚未创建）。创建应用时，需使用至少一个应用程序机密。
2. 找到“直接应用程序权限”部分，然后添加应用所需的权限。
3. **保存**应用注册。

#### 建议：让用户登录应用
在构建使用应用程序权限的应用程序时，应用通常需要一个页面或视图，使管理员能够批准应用的权限。此页面可以是应用登录流的一部分、应用设置的一部分，或者专用的“连接”流。在许多情况下，合理的结果是应用只在用户使用工作或学校 Microsoft 帐户登录之后才显示此“连接”视图。

如果让用户登录到应用，可以在请求用户批准应用程序权限之前识别该用户所属组织。尽管在严格意义上不需要这样做，但这有助于为用户带来更直观的体验。若要让用户登录，请遵循 [v2.0 协议教程](/documentation/articles/active-directory-v2-protocols/)。

#### 向目录管理员请求权限
准备好向组织管理员请求权限时，可将用户重定向到 v2.0 *管理员许可终结点*。


	// Line breaks are for legibility only.

	GET https://login.microsoftonline.com/{tenant}/adminconsent?
	client_id=6731de76-14a6-49ae-97bc-6eba6914391e
	&state=12345
	&redirect_uri=http://localhost/myapp/permissions



	// Pro tip: Try pasting the following request in a browser!



	https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions


| 参数 | 条件 | 说明 |
| --- | --- | --- |
| tenant |必选 |要向其请求权限的目录租户。此参数可采用 GUID 或友好名称格式。如果不知道用户属于哪个租户并想让他们登录到任一租户，请使用 `common`。 |
| client\_id |必选 |[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)为应用分配的应用程序 ID。 |
| redirect\_uri |必选 |要向其发送响应，供应用处理的重定向 URI。它必须与门户中注册的其中一个重定向 URI 完全匹配，否则必须经过 URL 编码并可包含其他路径段。 |
| state |建议 |同样随令牌响应返回的请求中所包含的值。可以是所需的任何内容的字符串。该 state 用于在身份验证请求出现之前，于应用中编码用户的状态信息，例如之前所在的页面或视图。 |

此时，Azure AD 强制要求只有租户管理员可以登录来完成请求。系统将要求管理员批准在应用注册门户中针对应用请求的所有直接应用程序权限。

##### 成功的响应
如果管理员批准了应用程序的权限，成功响应如下所示：


	GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True


| 参数 | 说明 |
| --- | --- | --- |
| tenant |向应用程序授予所请求权限的目录租户（采用 GUID 格式）。 |
| state |同样随令牌响应返回的请求中所包含的值。可以是所需的任何内容的字符串。该 state 用于在身份验证请求出现之前，于应用中编码用户的状态信息，例如之前所在的页面或视图。 |
| admin\_consent |设置为 **true**。 |

##### 错误响应
如果管理员未批准应用程序的权限，失败响应如下所示：


	GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request


| 参数 | 说明 |
| --- | --- | --- |
| error |可用于将错误分类，以及对错误做出反应的错误代码字符串。 |
| error\_description |可帮助识别错误根本原因的特定错误消息。 |

从应用预配终结点收到成功响应后，应用便已获得它所请求的直接应用程序权限。现在，可以请求所需的资源令牌。

## 获取令牌
获取应用程序的必要授权后，可以继续获取 API 的访问令牌。若要使用客户端凭据授予获取令牌，请将 POST 请求发送到 `/token` v2.0 终结点：


	POST /common/oauth2/v2.0/token HTTP/1.1
	Host: login.microsoftonline.com
	Content-Type: application/x-www-form-urlencoded

	client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials



	curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/common/oauth2/v2.0/token'


| 参数 | 条件 | 说明 |
| --- | --- | --- |
| client\_id |必选 |[应用程序注册门户](https://apps.dev.microsoft.com/?referrer=/documentation/articles&deeplink=/appList)为应用分配的应用程序 ID。 |
| scope |必选 |在此请求中针对 `scope` 参数传递的值应该是所需资源的资源标识符（应用程序 ID URI），附有 `.default` 后缀。在 Microsoft Graph 示例中，该值为 `https://graph.microsoft.com/.default`。此值通知 v2.0 终结点为应用配置的所有直接应用程序权限，终结点应该为与要使用的资源关联的对象颁发令牌。 |
| client\_secret |必选 |在应用注册门户中为应用生成的应用程序机密。 |
| grant\_type |必选 |必须是 `client_credentials`。 |

##### 成功的响应
成功响应如下所示：


	{
	  "token_type": "Bearer",
	  "expires_in": 3599,
	  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
	}


| 参数 | 说明 |
| --- | --- |
| access\_token |请求的访问令牌。应用可以使用此令牌验证受保护的资源，例如验证 Web API。 |
| token\_type |指示令牌类型值。Azure AD 支持的唯一类型是 `bearer`。 |
| expires\_in |访问令牌的有效期（以秒为单位）。 |

##### 错误响应
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
| --- | --- |
| error |可用于对发生的错误分类以及对错误做出反应的错误代码字符串。 |
| error\_description |可帮助识别身份验证错误根本原因的特定错误消息。 |
| error\_codes |可帮助诊断的 STS 特定错误代码列表。 |
| timestamp |发生错误的时间。 |
| trace\_id |可帮助诊断的请求唯一标识符。 |
| correlation\_id |可帮助跨组件诊断的请求唯一标识符。 |

## 使用令牌
获取令牌后，使用该令牌对资源发出请求。令牌过期时，向 `/token` 终结点重复该请求，即可获取全新的访问令牌。


	GET /v1.0/me/messages
	Host: https://graph.microsoft.com
	Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...



	// Pro tip: Try the following command! (Replace the token with your own.)



	curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q" 'https://graph.microsoft.com/v1.0/me/messages'


## 代码示例
若要查看使用管理员许可终结点实现客户端凭据授予的应用程序示例，请参阅 [v2.0 守护程序代码示例](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)。

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update-->