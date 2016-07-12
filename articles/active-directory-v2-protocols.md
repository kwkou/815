<properties
	pageTitle="Azure AD v2.0 协议 | Azure"
	description="有关 Azure AD v2.0 终结点支持的协议的指南。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/20/2016"
	wacn.date="06/24/2016"/>

# v2.0 协议 - OAuth 2.0 和 OpenID Connect

v2.0 终结点可以使用 Azure AD，通过行业标准协议（OpenID Connect 与 OAuth 2.0）提供标识即服务。尽管此服务与标准兼容，但这些协议的两个实现之间仍然存在微妙的差异。如果你选择通过直接发送和处理 HTTP 请求，或使用第三方开放源代码库来编写代码，而不是使用我们的其中一个开放源代码库，则可以参考此处提供的有用信息。
<!-- TODO: Need link to libraries above -->

> [AZURE.NOTE]
	v2.0 终结点并不支持所有 Azure Active Directory 方案和功能。若要确定是否应使用 v2.0 终结点，请阅读 [v2.0 限制](/documentation/articles/active-directory-v2-limitations/)。

## 基础知识
几乎在所有的 OAuth 和 OpenID Connect 流中，都有四个参与交换的对象：

![OAuth 2.0 角色](./media/active-directory-v2-flows/protocols_roles.png)

- **授权服务器**是 v2.0 终结点。它负责确保用户的标识、授予和吊销对资源的访问权限，以及颁发令牌。它也称为标识提供者：安全处理与用户信息、用户访问权，以及流中合作对象彼此间信任关系有关的任何项目。
- **资源所有者**通常是用户。它是拥有数据的一方，并且有权允许第三方访问该数据或资源。
- **OAuth 客户端**是你的应用，按照其应用程序 ID 进行标识。它通常是与最终用户交互的对象，并向授权服务器请求令牌。客户端必须获得资源所有者授权才能访问资源。
- **资源服务器**是资源或数据所在的位置。它信任授权服务器安全验证和授权 OAuth 客户端，并使用 Bearer access\_token 来确保可以授予对资源的访问权限。


## 应用注册
所有使用 v2.0 终结点的应用都必须先在 [apps.dev.microsoft.com](https://apps.dev.microsoft.com) 上注册，然后才能使用 OAuth 或 OpenID Connect 进行交互。应用注册进程会收集一些值并将其分配到你的应用：

- 用于唯一标识应用的**应用程序 ID**
- 用于将响应定向回到应用的**重定向 URI** 或**包标识符**
- 其他一些特定于方案的值。

有关详细信息，请了解如何[注册应用](/documentation/articles/active-directory-v2-app-registration/)。

## 终结点
注册后，应用将通过向 v2.0 终结点发送请求来与 Azure AD 通信：


		https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
		https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token


其中 `{tenant}` 可以接受以下四个不同值之一：

| 值 | 说明 |
| ----------------------- | ------------------------------- |
| `common` | 允许用户使用个人的 Microsoft 帐户和工作/学校帐户从 Azure Active Directory 登录应用程序。 |
| `organizations` | 仅允许用户使用工作/学校帐户从 Azure Active Directory 登录应用程序。 |
| `consumers` | 仅允许用户使用个人的 Microsoft 帐户 (MSA) 登录应用程序。 |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` 或 `contoso.onmicrosoft.com` | 仅允许用户使用工作/学校帐户从特定的 Azure Active Directory 租户登录应用程序。可以使用 Azure AD 租户的友好域名或租户的 GUID 标识符。 |

有关如何与这些终结点交互的详细信息，请选择以下特定的应用类型。

## 令牌
OAuth 2.0 和 OpenID Connect 的 v2.0 实现广泛使用了持有者令牌，包括表示为 JWT 的持有者令牌。持有者令牌是一种轻型安全令牌，它授予对受保护资源的“持有者”访问权限。从这个意义上来说，“持有者”是可以提供令牌的任何一方。虽然某一方必须首先通过 Azure AD 的身份验证才能收到持有者令牌，但如果不采取必要的步骤在传输过程和存储中对令牌进行保护，令牌可能会被意外的某一方拦截并使用。虽然某些安全令牌具有内置机制来防止未经授权方使用它们，但是持有者令牌没有这一机制，因此必须在安全的通道（例如传输层安全 (HTTPS)）中进行传输。如果持有者令牌以明文传输，则恶意方可以利用中间人攻击来获得令牌并使用它来对受保护资源进行未经授权的访问。当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。请始终确保你的应用以安全的方式传输和存储持有者令牌。有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](http://tools.ietf.org/html/rfc6750)。

有关 v2.0 终结点中使用的不同类型令牌的详细信息，请参阅 [v2.0 终结点令牌参考](/documentation/articles/active-directory-v2-tokens/)。

## 协议

如果你已准备好查看部分示例请求，请从下列教程之一开始。每个教程对应于特定的身份验证方案。如果在确定适当的流时需要帮助，请查看[使用 v2.0 可以构建哪种类型的应用](/documentation/articles/active-directory-v2-flows/)。

- [使用 OAuth 2.0 构建移动和本机应用程序](/documentation/articles/active-directory-v2-protocols-oauth-code/)
- [使用 OpenID Connect 构建 Web 应用](/documentation/articles/active-directory-v2-protocols-oidc/)
- [使用 OAuth 2.0 隐式流构建单页应用](/documentation/articles/active-directory-v2-protocols-implicit/)
- 使用 OAuth 2.0 客户端凭据流构建守护程序或服务器端进程（敬请期待）
- 使用 OAuth 2.0 代理流在 Web API 中获取令牌（敬请期待）

<!-- - Get tokens using a username & password with the OAuth 2.0 Resource Owner Password Credentials Flow (coming soon) --> 

<!---HONumber=Mooncake_0516_2016-->