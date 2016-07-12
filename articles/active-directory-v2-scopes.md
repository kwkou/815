<properties
	pageTitle="Azure AD v2.0 的范围、权限和同意 | Azure"
	description="介绍 Azure AD v2.0 终结点中的授权，包括范围、权限和同意。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>

# v2.0 终结点中的范围、权限和同意

与 Azure AD 集成的应用程序遵循可让用户控制应用程序如何访问其数据的特定授权模型。此授权模型的 v2.0 实现已更新，其中更改了应用程序必须与 Azure AD 交互的方式。本主题涵盖此授权模型的基本概念，包括范围、权限和同意。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 范围和权限

Azure AD 实施 [OAuth 2.0](/documentation/articles/active-directory-v2-protocols/) 授权协议，此方法允许第三方应用程序代表用户访问 Web 托管的资源。任何与 Azure AD 集成的 Web 托管资源都有资源标识符或**应用程序 ID URI**。例如，Microsoft 的某些 Web 托管资源包括：

- Office 365 统一邮件 API：`https://outlook.office.com`
- Azure AD 图形 API：`https://graph.windows.net`
- Microsoft Graph：`https://graph.microsoft.com`

这也适用于已与 Azure AD 集成的任何第三方资源。这些资源还可以定义一组可用于将该资源的功能分区成较小区块的权限。例如，Microsoft Graph 定义了以下几个权限：

- 读取用户的日历
- 写入用户的日历
- 以用户身份发送邮件
- [更多](https://graph.microsoft.io)

通过定义这些权限，资源可以更细微地掌控其数据及如何对外界公开数据的方式。第三方应用程序接着可以向最终用户请求这些权限，而最终用户必须先批准这些权限，应用程序才可以代表他们执行操作。将资源的功能切割成较小的权限集，即可将第三方应用程序构建为只请求他们所需的特定权限，以便执行其职责。它还可让用户完全知道应用程序将如何使用其数据，如此他们才对应用程序的行为不受恶意攻击影响更具信心。

在 Azure AD 和 OAuth 中，这些权限也称为**范围**。你还可能看到它们被称为 **oAuth2Permissions**。在 Azure AD 中范围以字符串值表示。仍以 Microsoft Graph 为例，每个权限的范围值如下：

- 读取用户的日历：**Calendar.Read**
- 写入用户的日历：**Mail.ReadWrite**
- 以用户身份发送邮件：**Mail.Send**

如下所述，在对 v2.0 终结点的请求中指定范围，应用程序即可请求这些权限。


## Consent

在 [OpenID Connect 或 OAuth 2.0](/documentation/articles/active-directory-v2-protocols/) 授权请求中，应用程序可以使用 **scope** 查询参数来请求它所需的权限。例如，当用户登录应用程序时，应用程序发送如下所示的请求（包含换行符以便于阅读）：


		GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
		client_id=6731de76-14a6-49ae-97bc-6eba6914391e
		&response_type=code
		&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
		&response_mode=query
		&scope=
		https%3A%2F%2Fgraph.microsoft.com%2Fcalendar.read%20
		https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
		&state=12345


**scope** 参数是应用程序所请求的范围列表（以空格分隔）。将范围值附加到资源的标识符（应用程序 ID URI）可指示每个范围。上述请求表示应用程序需要相应的权限来读取用户的邮箱，以及以用户身分发送邮件。

在用户输入其凭据之后，v2.0 终结点将检查是否有匹配的**用户同意**记录。如果用户未曾同意所请求权限的任何一项，v2.0 终结点将请求用户授予请求的权限。

![工作帐户同意屏幕截图](./media/active-directory-v2-flows/work_account_consent.png)

当用户批准权限时，则记录同意，用户在后续登录时无需重新同意。

## 使用权限

在用户同意应用程序的权限之后，应用程序即可获取访问令牌，而这些令牌表示应用程序访问资源的权限。给定的访问令牌只能用于单个资源，但其内部编码是应用程序已获得该资源的特有权限。若要获取访问令牌，应用程序可以对 v2.0 令牌终结点发出请求：


		POST common/oauth2/v2.0/token HTTP/1.1
		Host: https://login.microsoftonline.com
		Content-Type: application/json
		
		{
			"grant_type": "authorization_code",
			"client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
			"scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
			"code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
			"redirect_uri": "https://localhost/myapp",
			"client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
		}


然后，生成的访问令牌可用于资源的 HTTP 请求 - 它可靠地指示应用程序具有适当权限可执行给定任务的资源。

有关 OAuth 2.0 协议以及如何获取访问令牌的详细信息，请参阅 [v2.0 终结点协议参考](/documentation/articles/active-directory-v2-protocols/)。

## OpenId Connect 范围

OpenID Connect 的 v2.0 实现有一些明确定义但未应用到任何特定资源的范围 - **openid**、 **email**、 **profile** 和 **offline_access**。

#### OpenId

如果应用程序使用 [OpenID Connect](/documentation/articles/active-directory-v2-protocols/#openid-connect-sign-in-flow) 执行登录，则必须请求 **openid** 范围。**openid** 范围在工作帐户同意屏幕上显示为“登录”权限，而在个人 Microsoft 帐户同意屏幕上显示为“查看你的配置文件并使用你的 Microsoft 帐户连接到应用程序和服务”权限。此权限可让应用程序访问 OpenID Connect 用户信息终结点，因此需要用户批准。 **openid** 范围还可用于 v2.0 令牌终结点来获取 id\_tokens，该令牌可用于保护应用程序的不同组件之间的 HTTP 调用。

#### 电子邮件

**email** 范围可以连同 openid 范围和任何其他范围一起包含。它以 **email** 声明的形式向应用程序提供用户主要电子邮件地址的访问权限。如果电子邮件地址与用户帐户关联（并非总是如此），则 **email** 声明只包含在令牌中。如果使用 **email** 范围，应用程序应该准备好处理 **email** 声明不存在于令牌中的情况。

#### 配置文件

**profile** 范围可以连同 **openid** 范围和任何其他范围一起包含。它提供大量用户信息的应用程序访问权限。这包括但不限于用户的名字、姓氏、首选用户名、对象 ID 等等。有关指定用户的 id\_tokens 中可用的配置文件声明的完整列表，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)。

#### Offline\_access

[`offline_access`范围](http://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess)允许应用程序在较长时间内代表用户访问资源。在公司帐户同意屏幕上，此范围显示为“随时访问你的数据”权限。在个人 Microsoft 帐户同意屏幕上，则显示为“随时访问你的信息”权限。用户批准 **offline_access** 范围后，应用程序将能够接收来自 v2.0 令牌终结点的刷新令牌。刷新令牌属于长效令牌，可让应用程序在旧的访问令牌过期时获取新的访问令牌。

如果应用未请求 **offline_access** 范围，则收不到 refresh\_tokens。这意味着，当在 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow)中兑换 authorization\_code 时，只从 /**token** 终结点接收 access\_token。该 access\_token 短时间维持有效（通常是一小时），但最后终将过期。到时，应用必须将用户重定向回到 /**authorize** 终结点以检索新的 authorization\_code。在此重定向期间，根据应用程序的类型，用户或许无需再次输入其凭据或重新同意权限。

有关如何获取及使用刷新令牌的详细信息，请参阅 [v2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)。

<!---HONumber=Mooncake_0418_2016-->