<properties
	pageTitle="Azure AD v2.0 终结点 | Azure"
	description="原始 Azure AD 终结点与 v2.0 终结点之间的比较。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>

# v2.0 终结点有什么不同？

如果你熟悉 Azure Active Directory 或过去已将应用与 Azure AD 集成，v2.0 终结点中可能有一些预料不到的差异。本文档汇总了这些差异来帮助你了解。

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。


## Microsoft 帐户和 Azure AD 帐户
v2.0 终结点允许开发人员编写可接受使用单一身份验证终结点同时从 Microsoft 帐户和 Azure AD 帐户登录的应用。这样，你就可以编写完全不可认知帐户的应用；可以忽略用户登录时所用的帐户类型。当然，你可以让应用程序知道特定会话中所使用的帐户类型，但不必这么做。

例如，如果应用调用 [Microsoft Graph](https://graph.microsoft.io)，则有一些额外的功能和数据可供企业用户使用，例如他们的 SharePoint 站点或目录数据。但在许多操作（例如[阅读用户的邮件](https://graph.microsoft.io/docs/api-reference/v1.0/resources/message)）中，针对 Microsoft 帐户和 Azure AD 帐户编写的代码可以完全相同。

将应用与 Microsoft 帐户和 Azure AD 帐户集成现在是一个简单的过程。你可以使用一组终结点、单个资源库和单个应用注册来进入消费者和企业的世界。若要深入了解 v2.0 终结点，请查看[概述](/documentation/articles/active-directory-appmodel-v2-overview/)。


## 新的应用注册门户
v2.0 终结点只能在下列新位置中注册：[apps.dev.microsoft.com](https://apps.dev.microsoft.com)。可以在这个门户获取应用程序 ID、自定义应用程序登录页面的外观等等。访问门户时唯一需要的是 Microsoft 支持的帐户 — 个人帐户或工作/学校帐户。

随着时间推移，我们会不断地将越来越多的功能添加到这个应用注册门户中。此门户的用途是成为可用于管理与 Microsoft 应用程序相关的所有一切的新位置。


## 所有平台使用一个应用 ID
在原始的 Azure Active Directory 服务中，你可能已针对单个项目注册了多个不同的应用。必须针对本机客户端和 Web 应用使用不同的应用注册：

![旧的应用程序注册 UI](./media/active-directory-v2-flows/old_app_registration.PNG)

例如，如果你同时构建了网站和 iOS 应用，就必须使用两个不同的应用程序 ID 分别注册它们。如果你有网站和后端 Web API，则可能已在 Azure AD 中将每个 API 注册为不同的应用程序。如果你有 iOS 应用程序和 Android 应用程序，则还可能已注册两个不同的应用程序。

<!-- You may have even registered different apps for each of your build environments - one for dev, one for test, and one for production. -->

现在，每个项目只需要单个应用程序注册和单一应用程序 ID。你可以将多个“平台”添加到每个项目，并为添加的每个平台提供相应的数据。当然，你可以根据需要创建任意数量的应用程序，但在大多数的情况下应该只需要一个应用程序 ID。

<!-- You can also label a particular platform as "production-ready" when it is ready to be published to the outside world, and use that same Application Id safely in your development environments. -->

我们的目标是促成更简化的应用程序管理及开发经验，并且为可能正在处理的单个项目创建更加集成的视图。


## 范围而非资源
在原始的 Azure AD 服务中，应用可充当**资源**或是令牌接收者。资源可以定义它所了解的许多**范围**或 **oAuth2Permissions**，让客户端应用得以针对一组特定范围请求该资源的令牌。请考虑以 Azure AD 图形 API 作为资源的示例：

- 资源标识符，或 `AppID URI`：`https://graph.chinacloudapi.cn/`
- 范围，或 `OAuth2Permissions`：`Directory.Read`、`Directory.Write` 等等。  

这一切都适用于 v2.0 终结点。应用程序仍可作为资源、定义范围并由 URI 标识。客户端应用程序仍可请求访问这些范围。但是，客户端用于请求这些权限的方式已改变。在过去，Azure AD 的 OAuth 2.0 授权请求可能如下所示：


		GET https://login.microsoftonline.com/common/oauth2/authorize?
		client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
		&resource=https%3A%2F%2Fgraph.windows.net%2F
		...


其中 **resource** 参数指示客户端应用请求授权的资源。Azure AD 根据 Azure 门户中的静态设置计算应用程序所需的权限，并据以发出令牌。现在，相同的 OAuth 2.0 授权请求如下所示：


		GET https://login.microsoftonline.com/common/v2.0/oauth2/authorize?
		client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
		&scope=https%3A%2F%2Fgraph.windows.net%2Fdirectory.read%20https%3A%2F%2Fgraph.windows.net%2Fdirectory.write
		...


其中 **scope** 参数指示应用请求授权的资源和权限。所需的资源仍是请求中最新的 - 它只包含在 scope 参数的每个值中。以此方式使用 scope 参数可让 v2.0 终结点更符合 OAuth 2.0 规范，并且更贴近常见的行业实践。它还可以让应用执行下一节中所述的[增量同意](#incremental-and-dynamic-consent)。

## 增量同意和动态同意
在正式版 Azure AD 服务中注册的应用程序必须于应用程序创建时在 Azure 门户中指定其所需的 OAuth 2.0 权限：

![权限注册 UI](../media/active-directory-v2-flows/app_reg_permissions.PNG)

应用所需的权限是**以静态方式**配置的。尽管这可让应用程序的设置存在于 Azure 门户中并使代码好用又简单，但开发人员面临一些问题：

- 应用程序必须在应用程序创建时知道可能需要的所有权限。随着时间添加权限是一个繁琐的过程。
- 应用程序必须事先知道可能访问的所有资源。很难创建能够访问任意数目的资源的应用程序。
- 应用程序必须在用户第一次登录时请求可能需要的权限。在某些情况下，这导致非常冗长的权限列表，而让用户在初始登录时打消审批应用程序访问权限的念头。

通过 v2.0 终结点，可以在运行时**动态**指定应用在一般使用期间所需的权限。为此，可以在授权请求的 `scope` 参数中包含范围，以便在任何指定的时间点指定应用所需的范围：


		GET https://login.microsoftonline.com/common/v2.0/oauth2/authorize?
		client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
		&scope=https%3A%2F%2Fgraph.chinacloudapi.cn%2Fdirectory.read%20https%3A%2F%2Fgraph.windows.net%2Fdirectory.write
		...


应用程序的上述请求权限可读取 Azure AD 用户的目录数据，以及将数据写入到其目录。如果用户过去曾针对此特定应用程序同意这些权限，他们只要输入其凭据并登录应用程序。如果用户未曾同意这些权限的任何一项，v2.0 终结点请求用户同意这些权限。若要了解详细信息，可以阅读[权限、同意和范围](/documentation/articles/active-directory-v2-scopes/)。

允许应用通过 `scope` 参数动态请求权限，即可完全掌控用户的体验。如果需要，你可以选择将同意体验提前，并在一个初始授权请求中请求所有的权限。或者，如果应用程序需要大量的权限，可以选择在尝试使用应用程序的某些功能时，以递增方式向用户收集这些权限。

## 已知的范围

#### 脱机访问
v2.0 终结点可能需要针对应用使用新的已知权限 — `offline_access` 范围。如果应用程序需要长期表示用户访问资源，则所有应用程序都需要请求此权限，即使用户可能不主动使用此应用程序亦然。在同意对话框中，`offline_access` 范围对用户显示为“脱机访问数据”，而用户必须同意。请求 `offline_access` 权限可让 Web 应用从 v2.0 终结点接收 OAuth 2.0 refresh\_tokens。Refresh\_tokens 属于长效令牌，可用于交换新的 OAuth 2.0 access\_tokens 以延长访问期间。

如果应用未请求 `offline_access` 范围，则收不到 refresh\_tokens。这意味着，当在 [OAuth 2.0 授权代码流](/documentation/articles/active-directory-v2-protocols/#oauth2-authorization-code-flow)中兑换 authorization\_code 时，只从 `/token` 终结点接收 access\_token。该 access\_token 短时间维持有效（通常是一小时），但最后终将过期。到时，应用必须将用户重定向回到 `/authorize` 终结点以检索新的 authorization\_code。在此重定向期间，根据应用程序的类型，用户或许无需再次输入其凭据或重新同意权限。

若要深入了解 OAuth 2.0、refresh\_tokens 和 access\_tokens，请查看 [v2.0 协议参考](/documentation/articles/active-directory-v2-protocols/)。

#### OpenID、profile 和 email

在原始 Azure Active Directory 服务中，最基本的 OpenID Connect 登录流在生成的 id\_token 中提供丰富的用户相关信息。id\_token 中的声明可以包含用户的名称、首选用户名、电子邮件地址和对象 ID 等等。

我们现在限制 `openid` 范围允许应用访问的信息。“openid”范围只允许应用将用户登录，并接收用户的应用特定标识符。如果你想要获取有关应用程序中的用户的个人标识信息 (PII)，应用程序必须向用户请求其他权限。我们引入了两个新范围（`email` 和 `profile` 范围）让你执行这项操作。

`email` 范围非常简单，它可让应用通过 id\_token 中的 `email` 声明访问用户的主要电子邮件地址。`profile` 范围可让应用访问用户的所有其他基本信息 — 其名称、首选用户名、对象 ID 等等。

这样，你就能够以最低泄漏的方式编码应用程序 – 只可以向用户请求应用程序执行其作业所需的信息集。有关这些范围的详细信息，请参阅 [v2.0 范围参考](/documentation/articles/active-directory-v2-scopes/)。


## 令牌声明
v2.0 终结点颁发的令牌中的声明与正式版 Azure AD 终结点颁发的令牌不同 - 迁移到新服务的应用程序不应假设特定的声明存在于 id\_tokens 或 access\_tokens 中。v2.0 终结点颁发的令牌与 OAuth 2.0 和 OpenID Connect 规范兼容，但可能遵循与正式版 Azure AD 服务不同的语义。

若要了解 v2.0 令牌中发出的特定声明，请参阅 [v2.0 令牌参考](/documentation/articles/active-directory-v2-tokens/)。

## 限制
使用 v2.0 终结点时有一些要注意的限制。请参阅 [v2.0 限制文档](/documentation/articles/active-directory-v2-limitations/)，以了解特定的方案是否存在任何限制。

<!---HONumber=Mooncake_0620_2016-->