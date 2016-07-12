<properties
	pageTitle="Azure AD v2.0 终结点更改 | Azure"
	description="介绍了要对应用模型 v2.0 公共预览协议进行的更改。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/28/2016"/>

# v2.0 身份验证协议的重要更新
开发人员请注意！ 在接下来两周，我们会对 v2.0 身份验证协议进行一些更新，这些更新对于你在我们的预览期间编写的任何应用都可能是重大更改。

## 哪些人会受到影响？
任何已编写为使用 v2.0 聚合身份验证终结点的应用，


		https://login.microsoftonline.com/common/oauth2/v2.0/authorize


有关 v2.0 终结点的详细信息可以在[此处](/documentation/articles/active-directory-appmodel-v2-overview/)找到。

如果你已经通过直接编码到 v2.0 协议，使用 v2.0 终结点构建了应用，并使用我们的任意 OpenID Connect 或 OAuth Web 中间件，或使用其他第三方库来执行身份验证，则应该准备好测试你的项目，并且根据需要进行更改。

## 哪些人不会受到影响？
任何已根据生产 Azure AD 身份验证终结点编写的应用，


		https://login.microsoftonline.com/common/oauth2/authorize


此协议一定都是如此，不会发生任何更改。

此外，如果你的应用**只**使用我们的 ADAL 库来执行身份验证，则不必更改任何项目。ADAL 已保护你的应用免遭更改。

## 所做的更改有哪些？
### 从 JWT 标头删除 x5t 值
v2.0 终结点大量使用 JWT 令牌，其中包含标头参数部分以及令牌的相关元数据。如果解码其中一个当前 JWT 的标头，你会发现类似以下的情形：


		{ 
			"type": "JWT",
			"alg": "RS256",
			"x5t": "MnC_VZcATfM5pOYiJHMba9goEKY",
			"kid": "MnC_VZcATfM5pOYiJHMba9goEKY"
		}


“x5t”和“kid”属性都会识别从 OpenID Connect 元数据终结点检索的，应该用于验证令牌签名的公钥。

我们在这里进行的更改是要删除“x5t”属性。你可以继续使用相同的机制来验证令牌，但应该只依赖“kid”属性来检索正确的公钥，如 OpenID Connect 协议中所指定。

> [AZURE.IMPORTANT] **你的工作：确保应用不依赖于 x5t 值是否存在。**

### 删除 profile\_info
以前，v2.0 终结点一直在称为 `profile_info` 的令牌响应中返回 base64 编码的 JSON 对象。当通过向下列对象发送请求，从 v2.0 终结点请求访问令牌时：


		https://login.microsoftonline.com/common/oauth2/v2.0/token


响应看起来将类似于下列 JSON 对象：

		{ 
			"token_type": "Bearer",
			"expires_in": 3599,
			"scope": "https://outlook.office.com/mail.read",
			"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
			"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
			"profile_info": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
		}


`profile_info` 值包含登录到应用的用户的相关信息 — 其显示名称、名字、姓氏、电子邮件地址、标识符等等。`profile_info` 主要用于令牌缓存和显示用途。

我们现在删除 `profile_info` 值 — 不过别担心，我们仍然会在稍微不同的地方为开发人员提供此信息。v2.0 终结点现在会在每个令牌响应中返回 `id_token`，而不是 `profile_info`：


		{ 
			"token_type": "Bearer",
			"expires_in": 3599,
			"scope": "https://outlook.office.com/mail.read",
			"access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
			"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
			"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
		}
		

你可以解码并分析 id\_token，以检索你从 profile\_info 收到的相同信息。id\_token 是 JSON Web 令牌 (JWT)，其内容由 OpenID Connect 指定。用于执行此操作的代码应该非常类似 — 你只需要提取 id\_token 的中间段（主体），base64 会将其解码以在 JSON 对象中访问。

在接下来两周，你应该将应用编码为从 `id_token` 或 `profile_info`（以存在的那个为准）检索用户信息。如此一来，当进行更改时，你的应用可以无缝地处理从 `profile_info` 到 `id_token` 的转换而不会中断。

> [AZURE.IMPORTANT] **你的工作：确保应用不依赖于 `profile_info` 值是否存在。**

### 删除 id\_token\_expires\_in
与 `profile_info` 相似，我们同时也从响应中删除 `id_token_expires_in` 参数。以前，v2.0 终结点会返回 `id_token_expires_in` 的值以及每个 id\_token 响应，例如在授权响应中：


		https://myapp.com?id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...&id_token_expires_in=3599...


或令牌响应中：

		
		{ 
			"token_type": "Bearer",
			"id_token_expires_in": 3599,
			"scope": "openid",
			"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
			"refresh_token": "OAAABAAAAiL9Kn2Z27UubvWFPbm0gL...",
			"profile_info": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsI...",
		}


`id_token_expires_in` 值会指出 id\_token 保持有效的秒数。现在，我们完全删除 `id_token_expires_in` 值。你可以改为使用 OpenID Connect 标准 `nbf` 和 `exp` 声明来检查 id\_token 的有效性。有关这些声明的详细信息，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)。

> [AZURE.IMPORTANT] **你的工作：确保应用不依赖于 `id_token_expires_in` 值是否存在。**


### 更改 scope=openid 返回的声明
这项更改最为重要 — 事实上，它将影响使用 v2.0 终结点的几乎每个应用。许多应用程序使用 `openid` 范围将请求发送到 v2.0 终结点，例如：


		https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
		client_id=...
		&redirect_uri=...
		&response_mode=form_post
		&response_type=id_token
		&scope=openid offline_access https://outlook.office.com/mail.read


现在，当用户同意 `openid` 范围时，你的应用会在生成的 id\_token 中收到丰富的用户相关信息。这些声明可以包含用户的名称、首选用户名、电子邮件地址和对象 ID 等等。

在此更新中，我们会更改 `openid` 范围可让应用访问的信息，使其更符合 OpenID Connect 规范。`openid` 范围只允许应用将用户登录，并在 id\_token 的 `sub` 声明中接收用户的应用特定标识符。只被授予 `openid` 范围的 id\_token 中的声明将缺少所有个人身份信息。id\_token 声明示例为：


		{ 
			"aud": "580e250c-8f26-49d0-bee8-1c078add1609",
			"iss": "https://login.microsoftonline.com/b9410318-09af-49c2-b0c3-653adc1f376e/v2.0 ",
			"iat": 1449520283,
			"nbf": 1449520283,
			"exp": 1449524183,
			"nonce": "12345",
			"sub": "MF4f-ggWMEji12KynJUNQZphaUTvLcQug5jdF2nl01Q",
			"tid": "b9410318-09af-49c2-b0c3-653adc1f376e",
			"ver": "2.0",
		}


如果你想要获取有关应用程序中的用户的个人标识信息 (PII)，应用程序必须向用户请求其他权限。我们将从 OpenID Connect 规范引入对两个新范围（`email` 和 `profile` 范围）的支持，这两个范围允许你执行此操作。

- `email` 范围非常简单，它可让应用通过 id\_token 中的 `email` 声明访问用户的主要电子邮件地址。请注意，`email` 声明不一定出现在 id\_tokens 中 — 只有在用户的配置文件中可用时才会包含。
- `profile` 范围可让应用访问用户的所有其他基本信息 — 其名称、首选用户名、对象 ID 等等。

这样，你就能够以最低泄漏的方式编码应用 — 只能向用户请求应用执行其作业所需的信息集。如果你想继续获取应用当前接收的完整用户信息集，则应该在授权请求中包含所有三个范围：


		https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
		client_id=...
		&redirect_uri=...
		&response_mode=form_post
		&response_type=id_token
		&scope=openid profile email offline_access https://outlook.office.com/mail.read
		

应用可以立即开始发送 `email` 和 `profile` 范围，v2.0 终结点会接受这两个范围，并且根据需要开始向用户请求权限。不过，对 `openid` 范围解释的更改几周之后才会生效。

> [AZURE.IMPORTANT] **你的工作：如果应用需要用户的相关信息，则添加 `profile` 和 `email` 范围。** 请注意，默认情况下，ADAL 将在请求中同时包含这些权限。

### 删除颁发者尾部斜杠。
以前，v2.0 终结点的令牌中显示的颁发者值采用以下格式：


		https://login.microsoftonline.com/{some-guid}/v2.0/


其中 guid 是颁发令牌的 Azure AD 租户的 tenantId。进行这些更改之后，这两个令牌和 OpenID Connect 发现文档中的颁发者值将变为


		https://login.microsoftonline.com/{some-guid}/v2.0 




> [AZURE.IMPORTANT] **你的工作：确保应用在颁发者验证期间接受包含或不含尾部斜杠的颁发者值。**

## 为何更改？
引进这些更改的主要目的是要符合 OpenID Connect 标准规范。我们希望通过符合 OpenID Connect，将与 Microsoft 标识服务集成以及与业内其他标识服务集成的差异降到最低。我们想让开发人员使用他们最爱的开源身份验证库，而不必更改库来适应 Microsoft 的差异。

## 你该怎么办？
目前，你可以开始进行上述所有更改。你应该立即：

1.	**删除 `x5t` 标头参数上的所有依赖项。**
2.	**妥善处理令牌响应中从 `profile_info` 到 `id_token` 的转换。**
3.  **删除 `id_token_expires_in` 响应参数上的所有依赖项。**
3.	**如果应用需要基本用户信息，则向应用添加 `profile` 和 `email` 范围。**
4.	**接受令牌中包含或不含尾部斜杠的颁发者值。**

我们的 [v2.0 协议文档](/documentation/articles/active-directory-v2-protocols/)已更新以反映这些更改，因此你可以将它作为参考，帮助你更新代码。

如果你对更改的范围还有别的疑问，欢迎在我们的 Twitter (@AzureAD) 上与我们联系。

## 协议多久进行一次更改？
我们无法预见身份验证协议任何进一步的重大更改。我们特意将这些更改捆绑到一个版本中，这样，你就不需要马上再经历一次这种更新过程。当然，我们将继续向聚合 v2.0 身份验证服务添加你可以充分利用的功能，但这些更改应该只是附加的，不会中断现有代码。

最后，感谢你在预览期间进行试用。迄今为止，早期采用者的深刻见解和体验为我们提供了非常宝贵的信息，希望你继续与我们分享你的意见和想法。

祝你编码愉快！

Microsoft 标识部门

<!---HONumber=Mooncake_0613_2016-->