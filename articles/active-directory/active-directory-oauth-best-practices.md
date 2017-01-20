<properties
	pageTitle="Azure AD 中 OAuth 2.0 的最佳实践 | Azure"
	description="本文介绍在开发使用 Azure Active Directory 中的 OAuth 2.0 的应用程序时运用的最佳实践。"
	services="active-directory"
	documentationCenter=".net"
	authors="priyamohanram"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/31/2016"
	wacn.date="08/22/2016"/>


# Azure Active Directory 中 OAuth 2.0 的最佳实践

## 使用 State 参数

`state` 参数是客户端在请求中发送的、随机生成的不可重用值（通常是一个 GUID）。它是一个可选参数，但强烈建议在授权代码的请求中使用。响应中也包含相同的 `state` 值，在使用响应之前，应用程序必须验证状态值是否完全相同。

`state` 参数可帮助检测针对客户端的跨站点请求伪造 (CSRF) 攻击。有关 CSRF 攻击的详细信息，请参阅“OAuth 2.0 Authorization Framework”（OAuth 2.0 授权框架）中的 [Cross-Site Request Forgery（跨站点请求伪造）](https://tools.ietf.org/html/rfc6749#section-10.12)。

## 缓存访问令牌

为了尽量减少来自客户端应用程序的网络调用及其相关延迟，客户端应用程序应该根据 OAuth 2.0 响应中指定的令牌生存期缓存访问令牌。若要确定令牌生存期，请使用 `expires_in` 或 `expires_on` 参数值。

如果 Web API 资源返回 `invalid_token` 错误代码，则可能表示该资源确定令牌已过期。如果客户端和资源时钟时间不同（称为“时间偏差”），该资源可能会将令牌视为已过期，随后从客户端缓存中清除。如果发生这种情况，请从缓存中清除令牌，即使它仍处于其计算生存期内。

## 处理刷新令牌

刷新令牌没有指定的生存期。通常，刷新令牌的生存期相对较长。但是，在某些情况下，刷新令牌会过期、被吊销，或缺少执行所需操作的足够权限。客户端应用程序需要正确预期和处理令牌颁发终结点返回的错误。

当你收到具有刷新令牌错误的响应时，请丢弃当前刷新令牌，并请求新的授权代码或访问令牌。尤其是在授权代码授予流中使用刷新令牌时，如果收到具有 `interaction_required` 或 `invalid_grant` 错误代码的响应，请丢弃刷新令牌，并请求新的授权代码。

## 验证资源 ID 参数

客户端应用程序必须验证响应中 `resource_id` 参数的值。否则，恶意服务可能会造成权限提升攻击。

 防止攻击的建议策略是验证 resource\_id 是否与正在访问的 Web API URL 基部分相匹配。例如，如果正在访问 https://service.contoso.com/data， 则 `resource_id` 可以是 https://service.contoso.com/。 客户端应用程序必须拒绝不以基 URL 开头的 `resource_id`，除非存在可靠的替代方法来验证该 ID。

## 令牌颁发和重试指导原则

Azure AD 支持客户端可查询的多种令牌颁发终结点。使用以下指导原则来针对这些终结点实现重试逻辑以处理意外网络故障或服务器故障。

对于向 Azure AD 终结点发出的请求，响应重试失败通常返回 HTTP 500 系列的错误代码。在某些方案中，客户端是向 Azure AD 发出自动请求的应用程序或服务。在其他方案（如使用 WS-Federation 协议的基于 Web 的联合身份验证）中，客户端是 Web 浏览器，最终用户必须手动重试。

### 基于 HTTP 500 系列错误响应实现重试逻辑

如果 Active Directory 访问控制服务 (ACS) 返回 HTTP 500 系列错误，强烈建议使用重试逻辑。这包括：

- HTTP 错误 500 - 内部服务器错误
- HTTP 错误 502 - 网关错误
- HTTP 错误 503 - 服务不可用
- HTTP 错误 504 - 网关超时

虽然可以在重试逻辑中显式列出单个 HTTP 代码，但在返回任何 HTTP 500 系列错误的情况下，只需调用重试逻辑。

通常，在返回 HTTP 400 系列错误代码时，不建议使用重试逻辑。来自 ACS 的 400 系列 HTTP 错误响应代码意味着请求无效，需要修改。

重试逻辑应由 HTTP 错误代码（如 HTTP 504（外部服务器超时））或 HTTP 500 错误代码系列触发，而不是由 ACS 错误代码（如 ACS90005）触发。ACS 错误代码仅供参考，随时可能会更改。

### 重试应使用退避计时器以实现最佳流量控制

当客户端收到 HTTP 500 系列错误时，应等待一段时间后重试请求。为获得最佳结果，建议对于每次后续重试，增加此时间段的长度。使用此方法可优化需要较长时间解决的暂时性网络或服务器问题的请求速率，并可快速解决暂时性错误。

例如，使用指数退避计时器时，每个实例在重试之前的延迟以指数方式增加，例如，第 1 次重试：1 秒，第 2 次重试：2 秒，第 3 次重试：4 秒，依此类推。

根据用户体验需求调整重试次数和每次重试的间隔时间。但是，我们建议在五分钟的时间内最多重试五次。超时导致的失败需要较长的时间才能解决。

## 后续步骤

[Active Directory 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/)

<!---HONumber=AcomDC_0718_2016-->