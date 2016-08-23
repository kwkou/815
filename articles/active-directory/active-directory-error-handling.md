<properties
	pageTitle="OAuth 2.0 中的错误处理 | Azure"
	description="本文介绍在使用 Azure Active Directory 时 OAuth 2.0 中出现的常见错误类，以及有关处理这些错误的最佳实践。"
	services="active-directory"
	documentationCenter=".net"
	authors="priyamohanram"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/31/2016"
	wacn.date="07/26/2016"/>

# OAuth 2.0 中的错误处理

[AZURE.INCLUDE [active-directory-protocols](../../includes/active-directory-protocols.md)]

在本文中，我们将了解在使用 Azure Active Directory (Azure AD) 为应用程序授权时可能遇到的常见错误类的一些最佳做法。有关授权终结点和令牌颁发终结点的详细信息，请参阅 [Azure AD 中应用程序的身份验证流](/documentation/articles/active-directory-protocols-oauth-code/)。

## 授权终结点错误

在授权过程的第一个步骤中，客户端应用程序向 Azure AD 的 `/authorize` 终结点发送一条请求，其中包含它需要从用户获取的权限列表。

授权终结点错误源自 `/authorize` 终结点。当错误显示在网页上时，将以 HTTP 200 错误返回；当客户端应用程序可用于处理错误时，将使用 HTTP 302 重定向代码。

下面是当请求中缺少必需的 `response_type` 参数时，Azure AD 授权终结点发出的示例 HTTP 302 错误响应。


	GET  HTTP/1.1 302 Found
	Location: http://localhost/myapp/?error=invalid_request&error_description=AADSTS90014%3a+The+request+body+must+contain+the+following+parameter%3a+%27response_type%27.%0d%0aTrace+ID%3a+57f5cb47-2278-4802-a018-d05d9145daad%0d%0aCorrelation+ID%3a+570a9ed3-bf1d-40d1-81ae-63465cc25488%0d%0aTimestamp%3a+2013-12-31+05%3a51%3a35Z&state=D79E5777-702E-4260-9A62-37F75FF22CCE
		

| 参数 | 说明 |
|-----------|-------------|
| error | [OAuth 2.0 授权框架](http://tools.ietf.org/html/rfc6749)第 5.2 部分中定义的错误代码值。下表描述了 Azure AD 返回的错误代码。 |
| error\_description | 错误的更详细说明。此消息不是最终用户友好的。 |
| state | 状态值是一个随机生成的不可重用值，它在请求中发送，并在响应中返回，以防止跨站点请求伪造 (CSRF) 攻击。 |

### 授权终结点错误的错误代码

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

## 令牌颁发终结点错误

应用程序从 `/authorize` 终结点收到授权代码后，将其传递到 `/token` 终结点以接收指定资源的访问令牌。

令牌颁发终结点错误源自 `/token` 终结点。这是一些 HTTP 错误代码，因为客户端直接调用令牌颁发终结点。除了 HTTP 状态代码，Azure AD 令牌颁发终结点还返回 JSON 文档，其中的对象会对错误进行描述。

例如，如果请求中的 `client_id` 参数无效，将返回如下所示的错误：

		
	HTTP/1.1 400 Bad Request
	Content-Type: application/json; charset=utf-8
		
	{"error":"invalid_request","error_description":"AADSTS90011: Request is ambiguous, multiple application identifiers found. Application identifiers: '197451ec-ade4-40e4-b403-02105abd9049, 597451ec-ade4-40e4-b403-02105abd9049'.\r\nTrace ID: 4457d068-2a03-42b2-97f2-d55325289d86\r\nCorrelation ID: 6b3474d8-233e-463f-b0a3-86433d8ba889\r\nTimestamp: 2013-12-31 06:31:41Z","error_codes":[90011],"timestamp":"2013-12-31 06:31:41Z","trace_id":"4457d068-2a03-42b2-97f2-d55325289d86","correlation_id":"6b3474d8-233e-463f-b0a3-86433d8ba889"}

### HTTP 状态代码

下表列出了令牌颁发终结点返回的 HTTP 状态代码。在某些情况下，错误代码足以描述响应，但在发生错误的情况下，你需要分析随附的 JSON 文档并检查其错误代码。

| HTTP 代码 | 说明 |
|-----------|-------------|
| 400 | 默认的 HTTP 代码。在大多数情况下使用，原因通常是请求格式不当。修复并重新提交请求。 |
| 401 | 身份验证失败。例如，请求缺少 client\_secret 参数。|
| 403 | 授权失败。例如，用户没有权限访问该资源。 |
| 500 | 服务出现内部错误。重试请求。 |

### 错误响应中的 JSON 文档对象

| JSON 对象 | 说明 |
|-------------|-------------|
| error | [OAuth 2.0 授权框架](http://tools.ietf.org/html/rfc6749)第 5.2 部分中定义的错误代码值。下表描述了 Azure AD 返回的错误代码。 |
| error\_description | 错误的更详细说明。此消息不是最终用户友好的。 |
| timestamp | 错误发生的日期和时间，采用协调世界时 (UTC)。 |
| trace\_id | 标识错误跟踪的 ID。 |
| correlation\_id | 	客户端生成的值。当发送到令牌颁发终结点的请求的 `client-request-id` 标头中包含此值时，此 ID 将包含在 JSON 错误文档中。其他调用可能会在同一会话中使用此 ID。 |
| error\_codes | 包含映射到不同服务条件的错误代码的列表。不要在应用程序逻辑中使用这些代码。但是，可以使用它们来诊断问题并按错误代码在线搜索信息。 |

### JSON 文档错误代码值

| 错误代码 | 说明 | 客户端操作 |
|------------|-------------|---------------|
| invalid\_request | 协议错误，例如，缺少必需的参数。 | 修复并重新提交请求。 |
| invalid\_grant | 权限授予（或其参数）或刷新令牌无效、过期或被吊销。 | 修复并重新提交请求。 |
| unauthorized\_client | 经过身份验证的客户端无权使用此权限授予类型。 | 客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| invalid\_client | 客户端身份验证失败。 | 客户端凭据无效。若要修复，应用程序管理员应更新凭据。 |
| unsupported\_grant\_type | 授权服务器不支持权限授予类型。 | 更改请求中的授权类型。这种类型的错误应该只在开发过程中发生，并且应该在初始测试过程中检测到。 |
| invalid\_resource | 目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。 | 这表示未在租户中配置该资源（如果存在）。应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| interaction\_required | 请求需要用户交互。例如，需要额外的身份验证步骤。 | 使用同一资源重试请求。 |
| temporarily\_unavailable | 服务器暂时繁忙，无法处理请求。 | 重试请求。客户端应用程序可以向用户说明，其响应由于临时状态而延迟。|

## 源自受保护资源的错误

这是当受保护资源（例如 Web API）实现 OAuth 2.0 授权框架的 [RFC 6750](http://tools.ietf.org/html/rfc6750) 规范时可能返回的错误。

实现 RFC 6750 的受保护资源会发出 HTTP 状态代码。如果请求不包含身份验证凭据或缺少令牌，则响应将包含 `WWW-Authenticate` 标头。当某个请求失败时，资源服务器将使用 HTTP 状态代码和错误代码做出响应。

下面是不成功的请求和响应的示例，其中，客户端请求不包含持有者令牌：
		
		
	HTTP/1.1 401 Unauthorized
	WWW-Authenticate: Bearer authorization_uri="https://login.window.net/contoso.com/oauth2/authorize",  error="invalid_token",  error_description="The access token is missing.",


## 错误参数

| 参数 | 说明 |
|-----------|-------------|
| authorization\_uri | 授权服务器的 URI（物理终结点）。此值还用作查找键，从一个发现终结点中获取有关服务器的详细信息。<p><p>客户端必须验证授权服务器是否受信任。由 Azure AD 对资源进行保护时，只需验证 URL 是否以 Azure AD 支持的 https://login.chinacloudapi.cn 或其他主机名开头即可。特定于租户的资源应始终返回特定于租户的授权 URI。 |
| error | [OAuth 2.0 授权框架](http://tools.ietf.org/html/rfc6749)第 5.2 部分中定义的错误代码值。|
| error\_description | 错误的更详细说明。此消息不是最终用户友好的。|
| resource\_id | 返回资源的唯一标识符。客户端应用程序在请求资源的令牌时，可以使用此标识符作为 `resource` 参数的值。<p><p>客户端应用程序必须验证此值，否则，恶意服务可能会引发**提升权限**攻击<p><p>若要防止攻击，建议的策略是验证 `resource_id` 是否与要访问的 Web API URL 基相匹配。例如，如果要访问的是 https://service.contoso.com/data，则 `resource_id` 可以是 htttps://service.contoso.com/。客户端应用程序必须拒绝不以基 URL 开头的 `resource_id`，除非存在可靠的替代方法来验证该 ID。 |

## 持有者方案错误代码

RFC 6750 规范为在响应中使用 WWW-Authenticate 标头和持有者方案的资源定义了以下错误。

| HTTP 状态代码 | 错误代码 | 说明 | 客户端操作 |
|------------------|------------|-------------|---------------|
| 400 | invalid\_request | 请求格式不正确。例如，请求可能缺少某个参数或者使用了同一参数两次。 | 修复错误并重试请求。这种类型的错误应该只在开发过程中发生，并且应该在初始测试中检测到。 |
| 401 | invalid\_token | 访问令牌缺失、无效或被吊销。error\_description 参数的值提供了更多详细信息。 | 从授权服务器请求新令牌。如果新令牌失败，则表示发生了意外的错误。向用户发送一条错误消息，然后在随机延迟后重试。 |
| 403 | insufficient\_scope | 访问令牌不包含访问资源所需的模拟权限。 | 将新的授权请求发送到授权终结点。如果响应包含 scope 参数，则在对资源的请求中使用 scope 值。 |
| 403 | insufficient\_access | 令牌的使用者没有访问该资源所需的权限。 | 提示用户使用其他帐户或请求对指定资源的权限。 |

<!---HONumber=Mooncake_0725_2016-->