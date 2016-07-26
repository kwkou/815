<properties
	pageTitle="Azure AD .NET 协议概述 | Azure"
	description="本文介绍如何使用 Azure Active Directory 和 OAuth 2.0，通过 HTTP 消息来授权访问租户中的 Web 应用程序和 Web API。"
	services="active-directory"
	documentationCenter=".net"
	authors="priyamohanram"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/31/2016"
	wacn.date="07/26/2016"/>


# 使用 OAuth 2.0 和 Azure Active Directory 来授权访问 Web 应用程序


Azure Active Directory (Azure AD) 使用 OAuth 2.0，使你能够授权访问 Azure AD 租户中的 Web 应用程序和 Web API。本指南与语言无关，介绍在不使用我们的任何开放源代码库的情况下，如何发送和接收 HTTP 消息。

[OAuth 2.0 规范第 4.1 部分](https://tools.ietf.org/html/rfc6749#section-4.1)描述了 OAuth 2.0 授权代码流。它用于在大部分的应用类型（包括 Web 应用和本机安装的应用）中执行身份验证与授权。

[AZURE.INCLUDE [active-directory-protocols-getting-started](../includes/active-directory-protocols-getting-started.md)]


## OAuth 2.0 授权流

概括而言，应用程序的整个授权流看起来有点类似于：

![OAuth 授权代码流](./media/active-directory-protocols-oauth-code/active-directory-oauth-code-flow-native-app.png)


## 请求授权代码

授权代码流始于客户端将用户定向到的 `/authorize` 终结点。在此请求中，客户端指示必须向用户获取的权限。你可以在 Azure 经典门户的应用程序页中，通过底部抽屉中的“查看终结点”按钮获取 OAuth 2.0 终结点。

		
		// Line breaks for legibility only
		
		https://login.microsoftonline.com/{tenant}/oauth2/authorize?
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&response_type=code
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&response_mode=query
		&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
		&state=12345


| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------- |
| tenant | 必填 | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。独立于租户的令牌的允许值为租户标识符，例如 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`、`contoso.onmicrosoft.com` 或 `common` |
| client\_id | 必填 | 将应用注册到 Azure AD 时，分配给应用的应用程序 ID。可以在 Azure 管理门户中找到此值。单击“Active Directory”，单击目录，单击应用程序，然后单击“配置” |
| response\_type | 必填 | 必须包括授权代码流的 `code`。 |
| redirect\_uri | 建议 | 应用程序的 redirect\_uri，应用程序可在此发送及接收身份验证响应。其必须完全符合在门户中注册的其中一个 redirect\_uris，否则必须是编码的 url。对于本机和移动应用，应使用默认值 `urn:ietf:wg:oauth:2.0:oob`。 |
| 作用域 | 必填 | 你希望用户同意的[范围](active-directory-v2-scopes.md)的空格分隔列表。 |
| response\_mode | 建议 | 指定将生成的令牌送回到应用程序所应该使用的方法。可以是 `query` 或 `form_post`。 |
| state | 建议 | 同样随令牌响应返回的请求中所包含的值。其可以是你想要的任何内容的字符串。随机生成的唯一值通常用于[防止跨站点请求伪造攻击](http://tools.ietf.org/html/rfc6749#section-10.12)。该状态也用于在身份验证请求出现之前，于应用程序中编码用户的状态信息，例如之前所在的网页或视图。 |
| prompt | 可选 | 表示需要的用户交互类型。目前唯一的有效值为“login”、“none”和“consent”。`prompt=login` 强制用户在该请求上输入凭据，否定单一登录。`prompt=none` 则相反 - 它确保不对用户显示任何交互式提示。如果请求无法通过单一登录以无消息方式完成，v2.0 终结点将返回错误。`prompt=consent` 在用户登录之后触发 OAuth 同意对话框，询问用户是否要授予权限给应用程序。 |
| login\_hint | 可选 | 如果事先知道其用户名称，可用于预先填充用户登录页面的用户名称/电子邮件地址字段。通常应用在重新身份验证期间使用此参数，已经使用 `preferred_username` 声明从上一个登录撷使用者户名称。 |
| domain\_hint | 可选 | 可以是 `consumers` 或 `organizations`。如果包含，它跳过用户在 v2.0 登录页面上经历的基于电子邮件的发现过程，导致稍微更加流畅的用户体验。通常，应用将在重新身份验证期间使用此参数，方法是从前次登录提取 `tid`。如果 `tid` 声明值是 `9188040d-6c67-4c5b-b112-36a304b66dad`，应该使用 `domain_hint=consumers`。否则使用 `domain_hint=organizations`。 |

> [AZURE.NOTE] 如果用户属于某个组织，则该组织的管理员可以代表该用户许可或拒绝，也可以允许该用户进行许可。仅当管理员允许时，用户才有权许可。

此时，将请求用户输入其凭据，并许可 `scope` 查询参数中指定的权限。用户经过身份验证并同意后，Azure AD 将在请求的 `redirect_uri` 地址中向应用发送响应。

### 成功的响应

成功的响应如下所示：

		
		http://localhost:12345/?
		code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA
		&state=12345
		&session_state=733ad279-b681-49c3-9215-951abf94d2c5


| 参数 | 说明 |
| -----------------------| --------------- |
| admin\_consent | 如果管理员同意许可请求提示的内容，则该值为 True。|
| 代码 | 应用程序请求的授权代码。应用程序可以使用该授权代码请求目标资源的访问令牌。 |
| session\_state | 一个标识当前用户会话的唯一值。此值为 GUID，但应将其视为无需检查即可传递的不透明值。 |
| state | 如果请求中包含状态参数，响应中就应该出现相同的值。应用程序应该验证请求中的状态值
如果请求中包含状态参数，响应中会出现相同的值。应用程序最好验证请求和响应中的状态值是否完全相同。

### 错误响应

错误响应可能也发送到 `redirect_uri`，以便应用程序可以适当地处理。

		
		GET http://localhost:12345/?
		error=access_denied
		&error_description=the+user+canceled+the+authentication


## 使用授权代码请求访问令牌

你已获取授权代码并获得用户授权，现在可以通过将 POST 请求发送到 `/token` 终结点，使用该代码兑换所需资源的访问令牌：

		
		// Line breaks for legibility only
		
		POST /{tenant}/oauth2/token HTTP/1.1
		Host: https://login.microsoftonline.com
		Content-Type: application/x-www-form-urlencoded
		
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
		&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&grant_type=authorization_code
		&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: client_secret only required for web apps


| 参数 | | 说明 |
| ----------------------- | ------------------------------- | --------------------- |
| tenant | 必填 | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。独立于租户的令牌的允许值为租户标识符，例如 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`、`contoso.onmicrosoft.com` 或 `common` |
| client\_id | 必填 | 将应用注册到 Azure AD 时，分配给应用的应用程序 ID。可以在 Azure 经典门户中找到此信息。单击“Active Directory”，单击目录，单击应用程序，然后单击“配置” |
| grant\_type | 必填 | 必须是授权代码流的 `authorization_code`。 |
| 代码 | 必填 | 在上一部分中获取的 `authorization_code` |
| redirect\_uri | 必填 | 用于获取 `authorization_code` 的相同 `redirect_uri` 值。 |
| client\_secret | Web Apps 所需 | 在应用程序注册门户中为应用程序创建的应用程序机密。其不应用于本机应用程序，因为设备无法可靠地存储 client\_secrets。Web Apps 和 Web API 都需要应用程序机密，能够将 `client_secret` 安全地存储在服务器端。 |


### 成功的响应

成功的响应如下所示：

		
		{
		  "access_token": " eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
		  "token_type": "Bearer",
		  "expires_in": "3600",
		  "expires_on": "1388444763",
		  "resource": "https://service.contoso.com/",
		  "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA",
		  "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
		"id_token": " eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83ZmU4MTQ0Ny1kYTU3LTQzODUtYmVjYi02ZGU1N2YyMTQ3N2UvIiwiaWF0IjoxMzg4NDQwODYzLCJuYmYiOjEzODg0NDA4NjMsImV4cCI6MTM4ODQ0NDc2MywidmVyIjoiMS4wIiwidGlkIjoiN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlIiwib2lkIjoiNjgzODlhZTItNjJmYS00YjE4LTkxZmUtNTNkZDEwOWQ3NGY1IiwidXBuIjoiZnJhbmttQGNvbnRvc28uY29tIiwidW5pcXVlX25hbWUiOiJmcmFua21AY29udG9zby5jb20iLCJzdWIiOiJKV3ZZZENXUGhobHBTMVpzZjd5WVV4U2hVd3RVbTV5elBtd18talgzZkhZIiwiZmFtaWx5X25hbWUiOiJNaWxsZXIiLCJnaXZlbl9uYW1lIjoiRnJhbmsifQ.”
		}
		


| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| access\_token | 请求的访问令牌。应用程序可以使用此令牌来验证受保护的资源，例如 Web API。 |
| token\_type | 指示令牌类型值。Azure AD 唯一支持的类型是 Bearer。有关持有者令牌的详细信息，请参阅 [OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)（OAuth2.0 授权框架：持有者令牌用法 (RFC 6750)）](http://www.rfc-editor.org/rfc/rfc6750.txt) |
| expires\_in | 访问令牌的有效期（以秒为单位）。 |
| 作用域 | 向客户端应用程序授予的模拟权限。默认权限为 `user_impersonation`。受保护资源的所有者可在 Azure AD 中注册其他值。|
| refresh\_token | OAuth 2.0 刷新令牌。应用程序可以使用此令牌，在当前的访问令牌过期之后获取其他访问令牌。刷新令牌的生存期很长，而且可以用于延长保留资源访问权限的时间。 |
| id\_token | 无符号 JSON Web 令牌 (JWT)。应用程序可以 base64Url 解码此令牌的段，以请求已登录用户的相关信息。应用程序可以缓存并显示值，但不应依赖于这些值来获取任何授权或安全边界。 |

### JWT 令牌声明
`id_token` 参数值中的 JWT 令牌可以解码为以下声明：

		
		{
		 "typ": "JWT",
		 "alg": "none"
		}.
		{
		 "aud": "2d4d11a2-f814-46a7-890a-274a72a7309e",
		 "iss": "https://sts.windows.net/7fe81447-da57-4385-becb-6de57f21477e/",
		 "iat": 1388440863,
		 "nbf": 1388440863,
		 "exp": 1388444763,
		 "ver": "1.0",
		 "tid": "7fe81447-da57-4385-becb-6de57f21477e",
		 "oid": "68389ae2-62fa-4b18-91fe-53dd109d74f5",
		 "upn": "frank@contoso.com",
		 "unique_name": "frank@contoso.com",
		 "sub": "JWvYdCWPhhlpS1Zsf7yYUxShUwtUm5yzPmw_-jX3fHY",
		 "family_name": "Miller",
		 "given_name": "Frank"
		}.


`id_token` 参数包含以下声明类型。有关 JSON Web 令牌的详细信息，请参阅 [JWT IETF 草案规范](http://go.microsoft.com/fwlink/?LinkId=392344)。有关令牌类型和声明的详细信息，请阅读 [Supported Token and Claim Types（支持的令牌和声明类型）](active-directory-token-and-claims.md)

| 声明类型 | 说明 |
|------------|-------------|
| aud | 令牌的受众。如果向客户端应用程序颁发令牌，则受众是客户端的 `client_id`。
| exp | 过期日期。令牌的过期时间。对于要生效的令牌，当前日期/时间必须小于或等于 `exp` 值。该时间表示为自 1970 年 1 月 1 日 (1970-01-01T0:0:0Z) UTC 至令牌颁发时间的秒数。 |
| family\_name | 用户的姓氏。应用程序可以显示此值。 |
| given\_name | 用户的名字。应用程序可以显示此值。 |
| iat | 颁发时间。颁发 JWT 的时间。该时间表示为自 1970 年 1 月 1 日 (1970-01-01T0:0:0Z) UTC 至令牌颁发时间的秒数。 |
| iss | 标识令牌颁发者 |
| nbf | 不在此之前的时间。令牌生效的时间。若要使令牌生效，当前日期/时间必须大于或等于 Nbf 值。该时间表示为自 1970 年 1 月 1 日 (1970-01-01T0:0:0Z) UTC 至令牌颁发时间的秒数。 |
| oid | 用户对象在 Azure AD 中的对象标识符 (ID)。 |
| sub | 令牌使用者标识符。这是令牌描述的用户的持久不可变标识符。在缓存逻辑中使用此值。 |
| tid | 颁发令牌的 Azure AD 租户的租户标识符 (ID)。 |
| unique\_name | 可显示给用户的唯一标识符。通常，这是用户主体名称 (UPN)。 |
| upn | 用户的用户主体名称。 |
| ver | 版本。JWT 令牌的版本，通常为 1.0。 |

### 错误响应

下面是一个示例错误响应：
		
		
		{
		  "error": "invalid_grant",
		  "error_description": "AADSTS70002: Error validating credentials. AADSTS70008: The provided authorization code or refresh token is expired. Send a new interactive authorization request for this user and resource.\r\nTrace ID: 3939d04c-d7ba-42bf-9cb7-1e5854cdce9e\r\nCorrelation ID: a8125194-2dc8-4078-90ba-7b6592a7f231\r\nTimestamp: 2016-04-11 18:00:12Z",
		  "error_codes": [
		    70002,
		    70008
		  ],
		  "timestamp": "2016-04-11 18:00:12Z",
		  "trace_id": "3939d04c-d7ba-42bf-9cb7-1e5854cdce9e",
		  "correlation_id": "a8125194-2dc8-4078-90ba-7b6592a7f231"
		}

| 参数 | 说明 |
| ----------------------- | ------------------------------- |
| error | 用于分类发生的错误类型与响应错误的错误码字符串。 |
| error\_description | 帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error\_codes | 帮助诊断的 STS 特定错误代码列表。 |
| timestamp | 发生错误的时间。 |
| trace\_id | 帮助诊断的请求唯一标识符。 |
| correlation\_id | 帮助跨组件诊断的请求唯一标识符。|

## 使用访问令牌来访问资源

你已经成功获取 `access_token`，现在可以通过在 `Authorization` 标头中包含令牌，在 Web API 的请求中使用令牌。[RFC 6750](http://www.rfc-editor.org/rfc/rfc6750.txt) 规范说明了如何使用 HTTP 请求中的持有者令牌访问受保护的资源。

### 示例请求

		
		GET /data HTTP/1.1
		Host: service.contoso.com
		Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ


## 刷新访问令牌

访问令牌的生存期很短，必须在其过期后刷新才能继续访问资源。若要刷新 `access_token`，可以向 `/token` 终结点提交另一个 `POST` 请求，但这次要提供 `refresh_token` 而不是 `code`。

刷新令牌的生存期未提供，并因策略设置和 Azure AD 吊销授权代码授予的时间而异。应用程序应该预期到新访问令牌请求失败的情况并进行相应处理。在这种情况下，应用程序应该返回到请求新访问令牌的代码。

下面提供了一个请求**特定于租户的**终结点（也可以使用**公用**终结点）使用刷新令牌获取新访问令牌的示例：

		
		// Line breaks for legibility only
		
		POST /{tenant}/oauth2/token HTTP/1.1
		Host: https://login.microsoftonline.com
		Content-Type: application/x-www-form-urlencoded
		
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
		&grant_type=refresh_token
		&resource=https%3A%2F%2Fservice.contoso.com%2F
		&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps

| 参数 | 说明 |
|-----------|-------------|
| access\_token | 所请求的新访问令牌。|
| expires\_in | 令牌的剩余生存期，以秒为单位。典型值为 3600（1 小时）。 |
| expires\_on | 令牌过期的日期和时间。该日期表示为自 1970-01-01T0:0:0Z UTC 至过期时间的秒数。 |
| refresh\_token | 一个新增的 OAuth 2.0 refresh\_token，可用于在此响应中的某个令牌过期时请求新访问令牌。 |
| resource | 标识可使用访问令牌访问的受保护资源。 |
| 作用域 | 向本机客户端应用程序授予的模拟权限。默认权限为 **user\_impersonation**。目标资源的所有者可在 Azure AD 中注册备用值。 |
| token\_type | 令牌类型。唯一支持的值为 **bearer**。 |

### 成功的响应

成功的令牌响应如下：
		
		
		{
		  "token_type": "Bearer",
		  "expires_in": "3600",
		  "expires_on": "1460404526",
		  "resource": "https://service.contoso.com/",
		  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
		  "refresh_token": "AwABAAAAv YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl PM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfmVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA"
		}


### 错误响应

下面是一个示例错误响应：

		
		{
		  "error": "invalid_resource",
		  "error_description": "AADSTS50001: The application named https://foo.microsoft.com/mail.read was not found in the tenant named 295e01fc-0c56-4ac3-ac57-5d0ed568f872.  This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant.  You might have sent your authentication request to the wrong tenant.\r\nTrace ID: ef1f89f6-a14f-49de-9868-61bd4072f0a9\r\nCorrelation ID: b6908274-2c58-4e91-aea9-1f6b9c99347c\r\nTimestamp: 2016-04-11 18:59:01Z",
		  "error_codes": [
		    50001
		  ],
		  "timestamp": "2016-04-11 18:59:01Z",
		  "trace_id": "ef1f89f6-a14f-49de-9868-61bd4072f0a9",
		  "correlation_id": "b6908274-2c58-4e91-aea9-1f6b9c99347c"
		}
		

<!---HONumber=AcomDC_0718_2016-->