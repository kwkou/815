<properties
    pageTitle="Azure Active Directory v2.0 令牌参考 | Azure"
    description="Azure AD v2.0 终结点发出的令牌和声明类型"
    services="active-directory"
    documentationcenter=""
    author="dstrockis"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="dc58c282-9684-4b38-b151-f3e079f034fd"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/13/2017"
    ms.author="dastrock" />  


# Azure Active Directory v2.0 令牌参考
Azure Active Directory (Azure AD) v2.0 终结点在每个[身份验证流](/documentation/articles/active-directory-v2-flows/)中发出多种安全令牌。本参考文档介绍每种令牌的格式、安全特征和内容。

> [AZURE.NOTE]
v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。
>
>

## 令牌类型  <a name="types-of-tokens"></a>
v2.0 终结点支持 [OAuth 2.0 授权协议](/documentation/articles/active-directory-v2-protocols/)，该协议使用访问令牌和刷新令牌。v2.0 终结点还支持通过[OpenID Connect](/documentation/articles/active-directory-v2-protocols/) 进行身份验证和登录。OpenID Connect 引入了第三种类型的令牌：ID 令牌。其中每种令牌都表示为*持有者*令牌。

持有者令牌是一种轻型安全令牌，可授予对受保护资源的持有者访问权限。持有者是可以提供令牌的任何一方。尽管某一方必须通过 Azure AD 的身份验证才能收到持有者令牌，但如果不采取措施在传输和存储过程中对令牌进行保护，令牌可能会被意外的某一方拦截并使用。某些安全令牌具有防止未授权方使用令牌的内置机制，但持有者令牌并不具有这种机制。持有者令牌必须在安全通道（例如传输层安全性 (HTTPS)）中传输。如果传输持有者令牌时不采取这种安全措施，则恶意方可能会利用“中间人攻击”来获取令牌，并使用它对受保护的资源进行未经授权的访问。当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。请始终确保应用以安全的方式传输和存储持有者令牌。有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](http://tools.ietf.org/html/rfc6750)。

v2.0 终结点颁发的许多令牌都以 JSON Web 令牌 (JWT) 的方式实现。JWT 是一种精简的 URL 安全方法，可在两方之间传输信息。JWT 中的信息称为*声明*。这些信息是有关令牌持有者和主题的断言信息。JWT 中的声明是为了传输而编码和序列化的 JavaScript 对象表示法 (JSON) 对象。由于 v2.0 终结点所颁发的 JWT 已签名但未加密，因此可以轻松地检查 JWT 的内容以便调试。有关 JWT 的详细信息，请参阅 [JWT 规范](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。

### ID 令牌 <a name="id_tokens"></a>
ID 令牌是应用使用 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/) 执行身份验证时收到的一种登录安全令牌形式。ID 令牌以 [JWT](#types-of-tokens) 表示，包含可让用户登录应用的声明。可以多种方式使用 ID 令牌中的声明。通常情况下，管理员使用 ID 令牌显示帐户信息或者在应用中做出访问控制决策。v2.0 终结点只颁发一种类型的 ID 令牌，无论登录的用户类型，都具有一组一致的声明。个人 Microsoft 帐户用户的 ID 令牌格式和内容与工作或学校帐户相同。

目前，ID 令牌已签名但未加密。应用收到 ID 令牌时，必须[验证签名](#validating-tokens)以证明令牌的真实性，并验证令牌中的几个声明来证明其有效性。应用验证的声明根据方案要求而有所不同，但在每种情况下，应用都必须执行一些[常见声明验证](#validating-tokens)。

以下部分提供了有关 ID 令牌中的声明的完整详细信息，以外还提供了一个示例 ID 令牌。请注意，ID 令牌中的声明不按特定的顺序返回。此外，可随时将新的声明引入 ID 令牌。引入新声明时，应用应该不会中断。以下列表包含应用当前能够可靠解释的声明。可在 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)中找到更多详细信息。

#### 示例 ID 令牌

	eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiI2NzMxZGU3Ni0xNGE2LTQ5YWUtOTdiYy02ZWJhNjkxNDM5MWUiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vYjk0MTk4MTgtMDlhZi00OWMyLWIwYzMtNjUzYWRjMWYzNzZlL3YyLjAiLCJpYXQiOjE0NTIyODUzMzEsIm5iZiI6MTQ1MjI4NTMzMSwiZXhwIjoxNDUyMjg5MjMxLCJuYW1lIjoiQmFiZSBSdXRoIiwibm9uY2UiOiIxMjM0NSIsIm9pZCI6ImExZGJkZGU4LWU0ZjktNDU3MS1hZDkzLTMwNTllMzc1MGQyMyIsInByZWZlcnJlZF91c2VybmFtZSI6InRoZWdyZWF0YmFtYmlub0BueXkub25taWNyb3NvZnQuY29tIiwic3ViIjoiTUY0Zi1nZ1dNRWppMTJLeW5KVU5RWnBoYVVUdkxjUXVnNWpkRjJubDAxUSIsInRpZCI6ImI5NDE5ODE4LTA5YWYtNDljMi1iMGMzLTY1M2FkYzFmMzc2ZSIsInZlciI6IjIuMCJ9.p_rYdrtJ1oCmgDBggNHB9O38KTnLCMGbMDODdirdmZbmJcTHiZDdtTc-hguu3krhbtOsoYM2HJeZM3Wsbp_YcfSKDY--X_NobMNsxbT7bqZHxDnA2jTMyrmt5v2EKUnEeVtSiJXyO3JWUq9R0dO-m4o9_8jGP6zHtR62zLaotTBYHmgeKpZgTFB9WtUq8DVdyMn_HSvQEfz-LWqckbcTwM_9RNKoGRVk38KChVJo4z5LkksYRarDo8QgQ7xEKmYmPvRr_I7gvM2bmlZQds2OeqWLB1NSNbFZqyFOCgYn3bAQ-nEQSKwBaA36jYGPOVG2r2Qv1uKcpSOxzxaQybzYpQ


> [AZURE.TIP]
练习时，若要检查示例 ID 令牌中的声明，请将示例 ID 令牌粘贴到 [calebb.net](http://calebb.net/) 中。
>
>

#### ID 令牌中的声明
| 名称 | 声明 | 示例值 | 说明 |
| --- | --- | --- | --- |
| 受众 |`aud`|`6731de76-14a6-49ae-97bc-6eba6914391e`|标识令牌的目标接收方。在 ID 令牌中，受众是在应用程序注册门户中分配给应用的应用程序 ID。应用应该验证此值，如果此值不匹配，应该拒绝该令牌。 |
| 颁发者 |`iss`|`https://login.microsoftonline.com/b9419818-09af-49c2-b0c3-653adc1f376e/v2.0 `|标识可构造并返回令牌的安全令牌服务 (STS)，以及对用户进行身份验证的 Azure AD 租户。应用应该验证颁发者声明，以确保令牌来自 v2.0 终结点。它也可以使用声明的 GUID 部分来限制可登录到应用的租户集。表示用户是来自 Microsoft 帐户的使用者用户的 GUID 为 `9188040d-6c67-4c5b-b112-36a304b66dad`。 |
| 颁发时间 |`iat`|`1452285331`|颁发令牌的时间，以纪元时间表示。 |
| 过期时间 |`exp`|`1452289231`|令牌失效的时间，以纪元时间表示。应用应该使用此声明来验证令牌生存期的有效性。 |
| 不早于 |`nbf`|`1452285331`|令牌生效的时间，以纪元时间表示。通常与颁发时间相同。应用应该使用此声明来验证令牌生存期的有效性。 |
| 版本 |`ver`|`2.0`|Azure AD 定义的 ID 令牌版本。对于 v2.0 终结点，该值为 `2.0`。 |
| 租户 ID |`tid`|`b9419818-09af-49c2-b0c3-653adc1f376e`|表示用户所属的 Azure AD 租户的 GUID。对于工作和学校帐户，该 GUID 就是用户所属组织的不可变租户 ID。对于个人帐户，该值为 `9188040d-6c67-4c5b-b112-36a304b66dad`。需要 `profile` 范围才能接收此声明。 |
| 代码哈希 |`c_hash`|`SGCPtt01wxwfgnYZy2VJtQ`|仅当 ID 令牌随 OAuth 2.0 授权代码一起颁发时，代码哈希才包含在 ID 令牌中。它可用于验证授权代码的真实性。有关执行此验证的详细信息，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)。 |
| 访问令牌哈希 |`at_hash`|`SGCPtt01wxwfgnYZy2VJtQ`|仅当 ID 令牌随 OAuth 2.0 访问令牌一起颁发时，访问令牌哈希才包含在 ID 令牌中。它可用于验证访问令牌的真实性。有关执行此验证的详细信息，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html)。 |
| nonce |`nonce`|`12345`|Nonce 是缓和令牌重放攻击的策略。应用可通过使用 `nonce` 查询参数，在授权请求中指定 nonce。在请求中提供的值将在 ID 令牌的 `nonce` 声明中发出（未经修改）。应用可根据在请求上指定的值验证此值，使应用的会话与特定的 ID 令牌相关联。应用应该在 ID 令牌验证过程中执行这项验证。 |
| 名称 |`name`|`Babe Ruth`|名称声明提供标识令牌使用者的人工可读值。该值不一定唯一，而且可变，只能用于显示目的。需要 `profile` 范围才能接收此声明。 |
| 电子邮件 |`email`|`thegreatbambino@nyy.partner.onmschina.cn`|与用户帐户关联的主要电子邮件地址（如果有）。其值是可变的，可能随时改变。需要 `email` 范围才能接收此声明。 |
| 首选用户名 |`preferred_username`|`thegreatbambino@nyy.partner.onmschina.cn`|表示 v2.0 终结点中用户的主要用户名。可以是电子邮件地址、电话号码或未指定格式的一般用户名。其值是可变的，可能随时改变。需要 `profile` 范围才能接收此声明。 |
| subject |`sub`|`MF4f-ggWMEji12KynJUNQZphaUTvLcQug5jdF2nl01Q`|令牌针对其断言信息的主体，例如应用的用户。此值固定不变，无法重新分配或重复使用。可以使用它来安全地执行授权检查，例如，当使用令牌访问资源时。由于使用者始终会在 Azure AD 颁发的令牌中存在，我们建议在通用授权系统中使用此值。 |
| 对象 ID |`oid`|`a1dbdde8-e4f9-4571-ad93-3059e3750d23`|Azure AD 系统中工作或学校帐户的对象 ID。不会针对个人 Microsoft 帐户颁发此声明。需要 `profile` 范围才能接收此声明。 |

### 访问令牌
目前，v2.0 终结点颁发的访问令牌仅供 Microsoft 服务使用。在任何目前支持的方案中，应用应该不需要执行任何的访问令牌验证或检查。可将访问令牌视为完全不透明。这些令牌只不过是应用可在 HTTP 请求中传递给 Microsoft 的字符串。

在不久的将来，v2.0 终结点将引入让应用从其他客户端接收访问令牌的功能。到时，本参考主题中的信息将会更新，包含应用执行访问令牌验证和其他类似任务时所需的信息。

当向 v2.0 终结点请求访问令牌时，v2.0 终结点还会返回有关访问令牌的元数据供应用使用。此信息包括访问令牌的到期时间及其有效范围。应用可使用此元数据对访问令牌执行智能缓存，而无需分析访问令牌本身。

### 刷新令牌
刷新令牌是应用可用来在 OAuth 2.0 流中获取新访问令牌的安全令牌。应用可以使用刷新令牌代表用户长期访问资源，而无需与用户交互。

刷新令牌属于多资源令牌。在一个资源的令牌请求期间收到的刷新令牌可以兑换完全不同资源的访问令牌。

为了在令牌响应中接收刷新令牌，应用必须请求并获得 `offline_acesss` 范围。若要了解有关 `offline_access` 范围的详细信息，请参阅有关[许可和范围](/documentation/articles/active-directory-v2-scopes/)的文章。

刷新令牌始终对应用完全不透明。它们由 Azure AD v2.0 终结点颁发，只能由 v2.0 终结点检查和解释。它们属于长效令牌，但不应将应用编写成预期刷新令牌将持续任何一段时间。刷新令牌可能出于各种原因而随时失效。让应用知道刷新令牌是否有效的唯一方式就是对 v2.0 终结点发出令牌请求以尝试兑换。

使用刷新令牌兑换新的访问令牌（而且应用已获得 `offline_access` 范围）时，将在令牌响应中收到新的刷新令牌。请保存新颁发的刷新令牌，替换请求中使用的刷新令牌。这可以保证刷新令牌尽可能长时间保持有效。

## 验证令牌  <a name="validating-tokens"></a>
目前，应用必须执行的唯一令牌验证就是验证 ID 令牌。若要验证 ID 令牌，应用应该同时验证 ID 令牌签名和 ID 令牌中的声明。

<!-- TODO: Link -->

Microsoft 提供了演示如何轻松处理令牌验证的库和代码示例。后续部分将介绍基础过程。此外，还可使用多个第三方开源库进行 JWT 验证。几乎每种平台和语言都至少有一个相应的库选项。

### 验证签名
JWT 包含三个段（以 `.` 字符分隔）。第一个段称为*标头*，第二个段称为*主体*，第三个段称为*签名*。签名段可用于验证 ID 令牌的真实性，使其获得应用的信任。

ID 令牌使用行业标准非对称式加密算法（例如 RSA 256）进行签名。ID 令牌标头包含用于签名令牌的密钥和加密方法的相关信息。例如：

	{
	  "typ": "JWT",
	  "alg": "RS256",
	  "kid": "MnC_VZcATfM5pOYiJHMba9goEKY"
	}


`alg` 声明指示用来为令牌签名的算法。`kid` 声明指示用来为牌签名的公钥。

v2.0 终结点随时都可以使用任何一组特定的公钥-私钥对来为 ID 令牌签名。v2.0 终结点会定期滚动更新一组可能的密钥，因此应将应用编写成自动处理这些密钥更改。对 v2.0 终结点所用公钥的更新进行检查的合理频率为 24 小时一次。

可以使用位于以下位置的 OpenID Connect 元数据文档来获取验证签名所需的签名密钥数据：


	https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration


> [AZURE.TIP]
请在浏览器中尝试打开此 URL！
>
>

此元数据文档是一个 JSON 对象，包含一些有用的信息，例如执行 OpenID Connect 身份验证所需的各种终结点的位置。该文档还包含 *jwks\_uri*，提供用于对令牌进行签名的公钥集的位置。位于 jwks\_uri 中的 JSON 文档包含当前正在使用的所有公钥信息。应用可以使用 JWT 标头中的 `kid` 声明选择此文档中已用于对令牌进行签名的公钥。然后可以使用正确的公钥和指定的算法来执行签名验证。

执行签名验证超出了本文档的介绍范围。有许多开源库可帮助实现此目的。

### 验证声明
如果应用在用户登录时收到 ID 令牌，则还应对 ID 令牌中的声明执行一些检查。这些检查包括但不限于：

- **受众**声明：验证该 ID 令牌是否计划提供给应用
- **不早于**和**过期时间**声明：验证 ID 令牌是否未过期
- **颁发者**声明：验证令牌是否由 v2.0 终结点颁发给应用
- **nonce**：缓解令牌重放攻击

有关应用应执行的声明验证的完整列表，请参阅 [OpenID Connect 规范](http://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation)。

上面的 [ID 令牌](#id_tokens) 部分包含了这些声明的预期值详细信息。

## 令牌生存期
下面提供的令牌生存期仅供参考。开发和调试应用时，这些信息可能会有所帮助。不应将应用编写成预期其中任何一个生存期都会维持不变。令牌生存期可以并且将会随时更改。

| 令牌 | 生存期 | 说明 |
| --- | --- | --- |
| ID 令牌（工作或学校帐户） |1 小时 |ID 令牌的有效期通常为 1 小时。Web 应用可以使用这一相同的生存期来维护自身与用户的会话（建议），或者你可以选择完全不同的会话生存期。如果应用需要获取新的 ID 令牌，需要对 v2.0 授权终结点发出新的登录请求。如果用户与 v2.0 终结点之间有有效的浏览器会话，可能无需再次输入凭据。 |
| ID 令牌（个人帐户） |24 小时 |个人帐户的 ID 令牌有效期通常为 24 小时。Web 应用可以使用这一相同的生存期来维护自身与用户的会话（建议），或者你可以选择完全不同的会话生存期。如果应用需要获取新的 ID 令牌，需要对 v2.0 授权终结点发出新的登录请求。如果用户与 v2.0 终结点之间有有效的浏览器会话，可能无需再次输入凭据。 |
| 访问令牌（工作或学校帐户） |1 小时 |在令牌响应中指定为令牌元数据的一部分。 |
| 访问令牌（个人帐户） |1 小时 |在令牌响应中指定为令牌元数据的一部分。可针对不同的生存期配置代表个人帐户颁发的访问令牌，但生存期通常为 1 小时。 |
| 刷新令牌（工作或学校帐户） |最长 14 天 |单个刷新令牌的有效期最长为 14 天。但是，刷新令牌可能出于各种原因而随时失效，因此应用应该继续尝试使用刷新令牌直到失败，或直到应用以新的刷新令牌替换它为止。如果用户输入其凭据已 90 天，则刷新令牌也会失效。 |
| 刷新令牌（个人帐户） |最长 1 年 |单个刷新令牌的有效期最长为 1 年。但是，刷新令牌可能出于各种原因而随时失效，因此应用应该继续尝试使用刷新令牌直到失败。 |
| 授权代码（工作或学校帐户） |10 分钟 |授权代码的生存期有意缩短，应在收到时立即兑换访问令牌和刷新令牌。 |
| 授权代码（个人帐户） |5 分钟 |授权代码的生存期有意缩短，应在收到时立即兑换访问令牌和刷新令牌。代表个人帐户颁发的授权代码只能使用一次。 |

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: wording update-->