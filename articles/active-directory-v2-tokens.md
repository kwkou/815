<properties
	pageTitle="Azure AD v2.0 令牌参考 | Azure"
	description="v2.0 终结点发出的令牌和声明类型"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>

# v2.0 令牌参考

v2.0 终结点在每个[身份验证流](/documentation/articles/active-directory-v2-flows/)的处理中发出多种安全令牌。本文档说明每种令牌的格式、安全特性和内容。本文档说明每种令牌的格式、安全特征和内容。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 类型的令牌

v2.0 终结点支持 [OAuth 2.0 授权协议](/documentation/articles/active-directory-v2-protocols/)，该协议使用 access\_token 与 refresh\_token。它还支持通过 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/#openid-connect-sign-in-flow) 进行身份验证和登录，其中引入了第三种类型的令牌 (id\_token)。每个令牌表示为“持有者令牌”。

持有者令牌是一种轻型安全令牌，它授予对受保护资源的“持有者”访问权限。从这个意义上来说，“持有者”是可以提供令牌的任何一方。虽然某一方必须首先通过 Azure AD 的身份验证才能收到持有者令牌，但如果不采取必要的步骤在传输过程和存储中对令牌进行保护，令牌可能会被意外的某一方拦截并使用。虽然某些安全令牌具有内置机制来防止未经授权方使用它们，但是持有者令牌没有这一机制，因此必须在安全的通道（例如传输层安全 (HTTPS)）中进行传输。如果持有者令牌以明文传输，则恶意方可以利用中间人攻击来获得令牌并使用它来对受保护资源进行未经授权的访问。当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。请始终确保你的应用以安全的方式传输和存储持有者令牌。有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](http://tools.ietf.org/html/rfc6750)。

v2.0 终结点颁发的许多令牌都实现为 Json Web 令牌或 JWT。JWT 是一种精简的 URL 安全方法，可在两方之间传输信息。JWT 中包含的信息也称为令牌持有者及使用者相关信息的“声明”或断言。JWT 中的声明是为了传输而编码和序列化的 JSON 对象。由于 v2.0 终结点所颁发的 JWT 已签名但未加密，因此你可以轻松地检查 JWT 的内容以便调试。有多个工具可以进行这项操作，例如 [calebb.net](http://jwt.calebb.net)。有关 JWT 的详细信息，请参阅 [JWT 规范](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。

## Id\_tokens

Id\_token 是应用使用 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/#openid-connect-sign-in-flow) 执行身份验证时收到的一种登录安全令牌形式。它表示为 [JWT](#types-of-tokens)，包含可让用户登录应用程序的声明。可以适时使用 id\_token 中的声明 - 通常用于显示帐户信息或在应用程序中进行访问控制决策。v2.0 终结点只颁发一种类型的 id\_token，而无论用户登录的类型为何，都具有一组一致的声明。也就是说，id\_token 的格式和内容与个人 Microsoft 帐户用户及工作或学校帐户的格式与内容相同。

Id\_token 已签名，但目前不会加密。当应用收到 id\_token 时，它必须[验证签名](#validating-tokens)以证明令牌的真实性，并验证令牌中的几个声明来证明其有效性。应用验证的声明根据方案要求而有所不同，但应用必须在每种方案中执行一些[常见的声明验证](#validating-tokens)。

下面提供了 id\_token 中声明的完整详细信息和示例 id\_token。请注意，id\_token 中的声明不按任何特定顺序返回。此外，新声明可随时引入 id\_token 中 - 应用不会在引入新声明时中断。以下列表包含本文发布时你的应用能够可靠解释的声明。如有需要，可以在 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)中找到更多详细信息。

#### 示例 id\_token
	
	eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiI2NzMxZGU3Ni0xNGE2LTQ5YWUtOTdiYy02ZWJhNjkxNDM5MWUiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vYjk0MTk4MTgtMDlhZi00OWMyLWIwYzMtNjUzYWRjMWYzNzZlL3YyLjAiLCJpYXQiOjE0NTIyODUzMzEsIm5iZiI6MTQ1MjI4NTMzMSwiZXhwIjoxNDUyMjg5MjMxLCJuYW1lIjoiQmFiZSBSdXRoIiwibm9uY2UiOiIxMjM0NSIsIm9pZCI6ImExZGJkZGU4LWU0ZjktNDU3MS1hZDkzLTMwNTllMzc1MGQyMyIsInByZWZlcnJlZF91c2VybmFtZSI6InRoZWdyZWF0YmFtYmlub0BueXkub25taWNyb3NvZnQuY29tIiwic3ViIjoiTUY0Zi1nZ1dNRWppMTJLeW5KVU5RWnBoYVVUdkxjUXVnNWpkRjJubDAxUSIsInRpZCI6ImI5NDE5ODE4LTA5YWYtNDljMi1iMGMzLTY1M2FkYzFmMzc2ZSIsInZlciI6IjIuMCJ9.p_rYdrtJ1oCmgDBggNHB9O38KTnLCMGbMDODdirdmZbmJcTHiZDdtTc-hguu3krhbtOsoYM2HJeZM3Wsbp_YcfSKDY--X_NobMNsxbT7bqZHxDnA2jTMyrmt5v2EKUnEeVtSiJXyO3JWUq9R0dO-m4o9_8jGP6zHtR62zLaotTBYHmgeKpZgTFB9WtUq8DVdyMn_HSvQEfz-LWqckbcTwM_9RNKoGRVk38KChVJo4z5LkksYRarDo8QgQ7xEKmYmPvRr_I7gvM2bmlZQds2OeqWLB1NSNbFZqyFOCgYn3bAQ-nEQSKwBaA36jYGPOVG2r2Qv1uKcpSOxzxaQybzYpQ


> [AZURE.TIP] 练习时，请试着将示例 id\_token 中的声明粘贴到 [calebb.net](https://calebb.net) 中进行检查。

#### Id\_tokens 中的声明
| Name | 声明 | 示例值 | 说明 |
| ----------------------- | ------------------------------- | ------------ | --------------------------------- |
| 目标受众 | `aud` | `6731de76-14a6-49ae-97bc-6eba6914391e` | 标识令牌的目标接收方。在 id\_tokens 中，受众是在应用注册门户中分配给应用的应用程序 ID。应用应该验证此值并拒绝不匹配的令牌。 |
| 颁发者 | `iss` | `https://login.microsoftonline.com/b9419818-09af-49c2-b0c3-653adc1f376e/v2.0 ` | 标识可构造并返回令牌的安全令牌服务 (STS)，以及对用户进行身份验证的 Azure AD 租户。应用应该验证颁发者声明，以确保令牌来自 v2.0 终结点。它也可以使用声明的 guid 部分限制允许登录此应用的租户集。表示用户是来自 Microsoft 帐户的使用者用户的 guid 为 `9188040d-6c67-4c5b-b112-36a304b66dad`。 |
| 颁发时间 | `iat` | `1452285331` | 颁发令牌的时间，以纪元时间表示。 |
| 过期日期 | `exp` | `1452289231` | 令牌失效的时间，以纪元时间表示。应用应该使用此声明来验证令牌生存期的有效性。 |
| 不早于 | `nbf` | `1452285331` | 令牌生效的时间，以纪元时间表示。通常与颁发时间相同。应用应该使用此声明来验证令牌生存期的有效性。 |
| 版本 | `ver` | `2.0` | Azure AD 定义的 id\_token 版本。对于 v2.0 终结点，此值为 `2.0`。 |
| 租户 ID | `tid` | `b9419818-09af-49c2-b0c3-653adc1f376e` | 表示用户所属的 Azure AD 租户的 guid。对于工作和学校帐户，该 guid 就是用户所属组织的不可变租户 ID。对于个人帐户，此值为 `9188040d-6c67-4c5b-b112-36a304b66dad`。需要 `profile` 范围才能接收此声明。 |
| 代码哈希 | `c_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | 仅当在 id\_token 中随 OAuth 2.0 授权代码一起颁发时，代码哈希才包含在 id\_token 中。它可用于验证授权代码的真实性。有关执行此验证的详细信息，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)。 |
| 访问令牌哈希 | `at_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | 仅当在 id\_token 中随 OAuth 2.0 访问令牌一起颁发时，访问令牌才包含在 id\_token 中。它可用于验证访问令牌的真实性。有关执行此验证的详细信息，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)。 |
| Nonce | `nonce` | `12345` | Nonce 是缓和令牌重放攻击的策略。你的应用可通过使用 `nonce` 查询参数，在授权请求中指定 nonce。在请求中提供的值将在 id\_token 的 `nonce` 声明中发出（未经修改）。这可让应用根据在请求上指定的值验证此值，使应用的会话与给定的 id\_token 相关联。应用应该在 id\_token 验证过程中执行这项验证。 |
| Name | `name` | `Babe Ruth` | 此名称声明提供了标识令牌使用者的人工可读值。此值不一定唯一，它是可变的，旨在仅用于显示目的。需要 `profile` 范围才能接收此声明。 |
| 电子邮件 | `email` | `thegreatbambino@nyy.onmicrosoft.com` | 与用户帐户关联的主要电子邮件地址（如果有）。其值可变，对给定的用户而言会随着时间改变。需要 `email` 范围才能接收此声明。 |
| 首选用户名 | `preferred_username` | `thegreatbambino@nyy.onmicrosoft.com` | 用于表示 v2.0 终结点中用户的主要用户名。它可以是电子邮件地址、电话号码或未指定格式的一般用户名。其值可变，对给定的用户而言会随着时间改变。需要 `profile` 范围才能接收此声明。 |
| 使用者 | `sub` | `MF4f-ggWMEji12KynJUNQZphaUTvLcQug5jdF2nl01Q` | 令牌针对其断言信息的主体，例如应用的用户。此值不可变且不能重新分配或重复使用，因此可以使用它来安全地执行授权检查，例如，当使用令牌访问资源时。因为使用者始终会在 Azure AD 颁发的令牌中存在，我们建议在通用授权系统中使用此值。 |
| ObjectId | `oid` | `a1dbdde8-e4f9-4571-ad93-3059e3750d23` | Azure AD 系统中工作或学校帐户的对象 ID。不会针对个人 Microsoft 帐户发出此声明。需要 `profile` 范围才能接收此声明。 |


## 访问令牌

v2.0 终结点颁发的访问令牌目前仅适用于 Microsoft 服务。在任何目前支持的方案中，应用应该无需要执行任何的访问令牌验证或检查。可以将访问令牌视为完全不透明 - 这些都是应用可以在 HTTP 请求中传递给 Microsoft 的字符串。

在不久的将来，v2.0 终结点将引入可让应用程序从其它客户端接收访问令牌的功能。到时，此信息将更新为应用程序执行访问令牌验证和其他类似任务所需的信息。

当向 v2.0 终结点请求访问令牌时，v2.0 终结点还向应用使用者返回一些有关访问令牌的元数据。此信息包括访问令牌的到期时间及其有效范围。这可让应用程序执行访问令牌的智能缓存，而无需分析访问令牌本身。

## 刷新令牌

刷新令牌是应用可用于在 OAuth 2.0 流中获取新访问令牌的安全令牌。它可让应用表示用户长期访问资源，而无需用户交互。

刷新令牌属于多资源令牌。也就是说，在一个资源的令牌请求期间收到的刷新令牌可以兑换完全不同资源的访问令牌。

若要在令牌响应中接收刷新令牌，应用必须请求并获得 `offline_acesss` 范围。若要深入了解 `offline_access` 范围，请参阅[此处有关同意和范围的文章](/documentation/articles/active-directory-v2-scopes/)。

刷新令牌永远对应用程序完全不透明。它们是由 Azure AD v2.0 终结点颁发，只能由 v2.0 终结点检查和解译。它们属于长效令牌，但你不应将应用编写成预期刷新令牌将持续任何一段时间。刷新令牌可能由于各种原因而随时失效。让应用知道刷新令牌是否有效的唯一方式就是对 v2.0 终结点发出令牌请求以尝试兑换。

当你使用刷新令牌兑换新的访问令牌（而且如果应用已获得 `offline_access` 范围）时，在令牌响应中将收到新的刷新令牌。你应该保存新颁发的刷新令牌，并替换请求中使用的刷新令牌。这将保证刷新令牌尽可能长期保持有效。

## 验证令牌

目前，应用程序必须执行的唯一令牌验证就是验证 id\_token。若要验证 id\_token，应用应该验证 id\_token 签名和 id\_token 中的声明。

<!-- TODO: Link -->
我们提供了库和代码示例用于演示如何轻松处理令牌验证 - 想要了解基本过程的用户可以参阅以下信息。另外还有多个第三方开放源代码库可用于 JWT 验证 - 几乎每个平台和语言都至少有一个选项。

#### 验证签名
JWT 包含三个段（以 `.` 字符分隔）。第一个段称为**标头**，第二个称为**主体**，第三个称为**签名**。签名段可用于验证 id\_token 的真实性，使应用信任它。

Id\_Token 使用行业标准非对称式加密算法（例如 RSA 256）进行签名。Id\_token 标头包含用于签名令牌的密钥和加密方法的相关信息：


		{
		  "typ": "JWT",
		  "alg": "RS256",
		  "kid": "MnC_VZcATfM5pOYiJHMba9goEKY"
		}


`alg` 声明表示用于对令牌进行签名的算法，而 `kid` 声明表示用于对令牌进行签名的特定公钥。

在任何给定时间点，v2.0 终结点可以使用一组特定公钥-私钥对中的一个对来签名 id\_token。v2.0 终结点定期换用一组可能的密钥，因此应将应用编写成自动处理这些密钥更改。检查 v2.0 终结点所用公钥的更新的合理频率大约为 24 小时。

可以使用位于以下位置的 OpenID Connect 元数据文档来获取验证签名所需的签名密钥数据：


		https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration


> [AZURE.TIP] 在浏览器中尝试打开此 URL！

此元数据文档是一个 JSON 对象，包含一些有用的信息，例如执行 OpenID Connect 身份验证所需的各种终结点的位置。

它还包含 `jwks_uri`，其提供用于对令牌进行签名的公钥集的位置。位于 `jwks_uri` 的 JSON 文档包含在该特定时间点使用的所有公钥信息。应用可以使用 JWT 标头中的 `kid` 声明选择本文档中已用于对特定令牌进行签名的公钥。然后可以使用正确的公钥和指定的算法来执行签名验证。

执行签名验证超出了本文档的范围 - 有许多开放源代码库可帮助这么做（如有必要）。

#### 验证声明
如果应用在用户登录时接收 id\_token，则还应该对 id\_token 中的声明执行一些检查。这些检查包括但不限于：

- **受众**声明 - 验证是否计划将 id\_token 提供给应用。
- **不早于**和**过期时间**声明 - 验证 id\_token 是否未过期。
- **颁发者**声明 - 验证令牌是否确实由 v2.0 终结点颁发给应用。
- **Nonce** - 缓和令牌重放攻击。
- 等等...

有关应用应该执行的声明验证的完整列表，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation)。

这些声明的预期值详细信息包含在上面的 [id\_token 部分](#id_tokens)中。


## 令牌生存期

以下令牌生存期有助于开发及调试应用程序，完全是为了让你了解而提供。你不应将应用编写成预期任何一个生存期维持不变 - 它们可以并将在任何给定的时间点更改。

| 令牌 | 生存期 | 说明 |
| ----------------------- | ------------------------------- | ------------ |
| Id\_Token（工作或学校帐户） | 1 小时 | Id\_Token 的有效期通常为 1 小时。Web 应用可以使用与此相同的生存期来维护自身与用户的会话（建议），或选择完全不同的会话生存期。如果应用需要获取新的 id\_token，它只需要对 v2.0 授权终结点发出新的登录请求。如果用户有 v2.0 终结点的有效浏览器会话，则可能无需再次输入其凭据。 |
| Id\_Token（个人帐户） | 24 小时 | 个人帐户的 Id\_Token 有效期通常为 24 小时。Web 应用可以使用与此相同的生存期来维护自身与用户的会话（建议），或选择完全不同的会话生存期。如果应用需要获取新的 id\_token，它只需要对 v2.0 授权终结点发出新的登录请求。如果用户有 v2.0 终结点的有效浏览器会话，则可能无需再次输入其凭据。 |
| 访问令牌（工作或学校帐户） | 1 小时 | 在令牌响应中指定为令牌元数据的一部分。 |
| 访问令牌（个人帐户） | 1 小时 | 在令牌响应中指定为令牌元数据的一部分。可针对不同的生存期配置代表个人帐户颁发的 Access\_token，但生存期通常为一小时。 |
| 刷新令牌（工作或学校帐户） | 最长 14 天 | 单个刷新令牌的有效期最长为 14 天。但是，刷新令牌可能由于任何原因而随时失效，因此应用应该继续尝试并使用刷新令牌直到失败，或直到应用程序以新的刷新令牌替换它为止。如果用户输入其凭据已 90 天，则刷新令牌也会失效。 |
| 刷新令牌（个人帐户） | 最长 1 年 | 单个刷新令牌的有效期最长为 1 年。但是，刷新令牌可能由于任何原因而随时失效，因此应用应该继续尝试并使用刷新令牌直到失败。 |
| 授权代码（工作或学校帐户） | 10 分钟 | 授权代码的生存期有意缩短，应在收到时立即兑换 access\_token 和 refresh\_token。 |
| 授权代码（个人帐户） | 5 分钟 | 授权代码的生存期有意缩短，应在收到时立即兑换 access\_token 和 refresh\_token。代表个人帐户颁发的授权代码也只能使用一次。 |

<!---HONumber=Mooncake_0418_2016-->