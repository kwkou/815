
<properties
   pageTitle="Azure AD 的身份验证方案 | Azure"
   description="Azure Active Directory (AAD) 的五个最常见身份验证方案概述"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="02/09/2016"
   wacn.date="06/03/2016"/>

# Azure AD 的身份验证方案

Azure Active Directory (Azure AD) 通过以下方式简化了对开发人员的身份验证：将标识提供为一项服务、支持行业标准协议（例如 OAuth 2.0 和 OpenID Connect），并提供用于不同平台的开源库来帮助你快速开始编码。本文档将帮助你了解 Azure AD 支持的各种方案并演示如何入门。具体内容划为以下几部分：

- [Azure AD 中的身份验证基本知识](#basics-of-authentication-in-azure-ad)

- [Azure AD 安全令牌中的声明](#claims-in-azure-ad-security-tokens)

- [在 Azure AD 中注册应用程序的基本知识](#basics-of-registering-an-application-in-azure-ad)

- [应用程序类型和方案](#application-types-and-scenarios)

  - [Web 浏览器到 Web 应用程序](#web-browser-to-web-application)

  - [单页面应用程序 (SPA)](#single-page-application-spa)

  - [本机应用程序到 Web API](#native-application-to-web-api)

  - [Web 应用程序到 Web API](#web-application-to-web-api)

  - [后台或服务器应用程序到 Web API](#daemon-or-server-application-to-web-api)



## <a name="basics-of-authentication-in-azure-ad"></a>Azure AD 中的身份验证基本知识

如果你不熟悉 Azure AD 中的身份验证基本概念，请阅读本部分。否则，你可能希望跳到[应用程序类型和方案](#application-types-and-scenarios)。

让我们考虑一下需要标识的最基本方案：Web 浏览器中的用户需要通过 Web 应用程序进行身份验证。此方案在 [Web 浏览器到 Web 应用程序](#web-browser-to-web-application)部分中有更详细的介绍，但可以在一开始的时候用来对 Azure AD 的功能进行说明，并通过概念对此方案的工作方式进行归纳。对于此方案，请参考以下示意图：

![Web 应用程序登录概述](./media/active-directory-authentication-scenarios/basics_of_auth_in_aad.png)

记住上面的图示，下面是你需要了解的其中的各种组件的相关信息：

- Azure AD 是标识提供程序，负责对组织的目录中存在的用户和应用程序的标识进行验证，并最终在那些用户和应用程序成功通过身份验证时颁发安全令牌。


- 希望将身份验证外包给 Azure AD 的应用程序必须在 Azure AD 中进行注册，Azure AD 将在目录中注册并唯一地标识该应用程序。


- 开发人员可以使用开源 Azure AD 身份验证库为你处理协议细节，方便你进行身份验证。有关详细信息，请参阅 [Azure Active Directory 身份验证库](/documentation/articles/active-directory-authentication-libraries/)。


• 在用户通过身份验证后，应用程序必须对用户的安全令牌进行验证以确保身份验证对于目标方是成功的。开发人员可以使用所提供的身份验证库来处理 Azure AD 提供的令牌的验证，包括 JSON Web 令牌 (JWT) 或 SAML 2.0。如果希望手动执行验证，请参阅 [JWT Token Handler](https://msdn.microsoft.com/library/dn205065.aspx)（JWT 令牌处理程序）文档。


> [AZURE.IMPORTANT] Azure AD 使用公钥加密对令牌进行签名以及验证它们是否有效。应用程序必须实施必要的逻辑才能确保始终使用最新密钥进行更新，此方面的详细信息，请参阅有关 [Azure AD 中签名密钥滚动更新的重要信息](https://msdn.microsoft.com/zh-cn/library/azure/dn641920.aspx)。


• 身份验证过程的请求和响应流是由所使用的身份验证协议（例如 OAuth 2.0、OpenID Connect、WS-Federation 或 SAML 2.0）决定的。[Azure Active Directory 身份验证协议](/documentation/articles/active-directory-authentication-protocols/)主题和下面的部分中更详细地讨论了这些协议。

> [AZURE.NOTE] Azure AD 支持 OAuth 2.0 和 OpenID Connect 标准，这些标准广泛使用持有者令牌，包括表示为 JWT 的持有者令牌。持有者令牌是一种轻型安全令牌，它授予对受保护资源的“持有者”访问权限。从这个意义上来说，“持有者”是可以提供令牌的任何一方。虽然某一方必须首先通过 Azure AD 的身份验证才能收到持有者令牌，但如果不采取必要的步骤在传输过程和存储中对令牌进行保护，令牌可能会被意外的某一方拦截并使用。虽然某些安全令牌具有内置机制来防止未经授权方使用它们，但是持有者令牌没有这一机制，因此必须在安全的通道（例如传输层安全 (HTTPS)）中进行传输。如果持有者令牌以明文传输，则恶意方可以利用中间人攻击来获得令牌并使用它来对受保护资源进行未经授权的访问。当存储或缓存持有者令牌供以后使用时，也应遵循同样的安全原则。请始终确保你的应用程序以安全的方式传输和存储持有者令牌。有关持有者令牌的更多安全注意事项，请参阅 [RFC 6750 第 5 部分](http://tools.ietf.org/html/rfc6750)。


现在你已概要了解了基本知识，请阅读下面几部分来了解在 Azure AD 中如何进行设置，以及 Azure AD 支持的常见方案。


## <a name="claims-in-azure-ad-security-tokens"></a>Azure AD 安全令牌中的声明

Azure AD 颁发的安全令牌包含与经过授权的使用者有关的信息的声明或断言。应用程序可将这些声明用于各种任务。例如，它们可以用于验证令牌、标识使用者的目录租户、显示用户信息、确定使用者的授权等等。任何给定安全令牌中存在的声明都依赖于令牌的类型、用于验证用户身份的凭据的类型和应用程序配置。下表提供了由 Azure AD 发出的每种声明的简要说明。有关详细信息，请参阅[支持的令牌和声明类型](/documentation/articles/active-directory-token-and-claims/)。


| 声明 | 说明 |
|-------|-------------|
| 应用程序 ID | 标识正在使用令牌的应用程序。
| 目标受众 | 标识令牌所针对的接收方资源。 |
| 应用程序身份验证上下文类引用 | 指示客户端的身份验证方式（公共客户端与机密客户端）。 |
| 即时身份验证 | 记录身份验证发生的日期和时间。 |
| 身份验证方法 | 指示令牌使用者的身份验证方式（密码、证书等）。 |
| 名字 | 按照 Azure AD 中的设置提供用户的给定名称。 |
| 组 | 包含用户所属 Azure AD 组的对象 ID。 |
| 标识提供程序 | 记录对令牌使用者进行身份验证的标识提供程序。 |
| 颁发时间 | 记录令牌的颁发时间，通常用于计算令牌新鲜度。 |
| 颁发者 | 标识发出令牌以及 Azure AD 租户的 STS。 |
| 姓氏 | 按照 Azure AD 中的设置提供用户的姓氏。 |
| Name | 提供了一个用户可读值，用于标识令牌使用者。 |
| 对象 ID | 包含 Azure AD 中的使用者的不可变、唯一标识符。 |
| 角色 | 包含授予用户的 Azure AD 应用程序角色的友好名称。 |
| 范围 | 指示授予客户端应用程序的权限。 |
| 使用者 | 指示令牌断言信息的主体。 |
| 租户 ID | 包含已颁发令牌的目录租户的不可变、唯一标识符。 |
| 令牌生存期 | 定义令牌保持有效状态的时间间隔。 |
| 用户主体名称 | 包含使用者的用户主体名称。 |
| 版本 | 包含令牌的版本号。 |


## <a name="basics-of-registering-an-application-in-azure-ad"></a>在 Azure AD 中注册应用程序的基本知识

将身份验证外包给 Azure AD 的任何应用程序都必须在目录中进行注册。此步骤涉及告诉 Azure AD 关于你的应用程序的情况，包括应用程序所在的 URL、在进行身份验证后要将回复发送到的 URL、用以标识你的应用程序的 URI，以及其他信息。该信息是必需的，有以下几个重要原因：

- 在处理登录或者交换令牌时，Azure AD 需要对等项才能与应用程序进行通信。其中包括：

  - 应用程序 ID URI：应用程序的标识符。此值在身份验证期间发送给 Azure AD 以指明调用者是希望为哪个应用程序获取令牌。另外，此值还包括在令牌中以便应用程序知道它是预定目标。


  - 回复 URL 和重定向 URI：对于 Web API 或 Web 应用程序，回复 URL 是当身份验证成功时 Azure AD 要将身份验证响应（包括令牌）发送到的位置。对于本机应用程序，重定向 URI 是一个唯一标识符，Azure AD 会将 OAuth 2.0 请求中的用户代理重定向到该标识符。


  - 客户端 ID：应用程序的 ID，这是在注册应用程序时由 Azure AD 生成的。当请求授权代码或令牌时，在身份验证期间会将客户端 ID 和密钥发送到 Azure AD。


  - 密钥：通过 Azure AD 进行身份验证以调用 Web API 时会随客户端 ID 一起发送的密钥。


- Azure AD 需要确保应用程序在访问你的目录数据、你组织中的其他应用程序等内容时，具有必要的权限

当你了解可以开发两类与 Azure AD 集成的应用程序时，设置工作将变得更为清晰：

- 单租户应用程序：单租户应用程序预定在单个组织中使用。它们通常是由企业开发人员编写的业务线 (LoB) 应用程序。单租户应用程序只需要供单个目录中的用户进行访问，因此，只需要将其设置在单个目录中。这些应用程序通常由组织中的开发人员进行注册。


- 多租户应用程序：多租户应用程序预定在许多组织中使用，而不仅是在单个组织中使用。它们通常是由独立软件供应商 (ISV) 编写的软件即服务 (SaaS) 应用程序。多租户应用程序需要设置在将使用它们的每个目录中，需要经过用户或管理员许可才能注册它们。当在目录中注册应用程序并向其授予对 Graph API 或者另一可能的 Web API 的访问权限时，此许可过程即已开始。当其他组织的用户或管理员注册使用应用程序时，会向他们显示一个对话框，其中显示了应用程序要求的权限。然后，用户或管理员可以许可应用程序的要求，这将向应用程序授予对指定数据的访问权限，并最终在其目录中注册该应用程序。

与开发单租户应用程序相比，当开发多租户应用程序时，会出现一些额外的注意事项。例如，如果要使你的应用程序可供多个目录中的用户使用，你需要一种机制来确定用户在哪个租户中。单租户应用程序只需要在其自己的目录中查找用户，而多租户应用程序需要从 Azure AD 中的所有目录来识别特定用户。为此，Azure AD 提供了一个任何多租户应用程序都可以在其中对登录请求进行定向的通用身份验证终结点，而不是提供特定于租户的终结点。对于 Azure AD 中的所有目录，该终结点都是 https://login.chinacloudapi.cn/common ，而特定于租户的终结点可能是 https://login.chinacloudapi.cn/contoso.onmicrosoft.com 。在开发应用程序时考虑通用终结点尤为重要，因为在登录、注销和令牌验证期间你将需要必要的逻辑来处理多租户。

如果你当前在开发单租户应用程序但希望使其可供许多组织使用，可以轻松地在 Azure AD 中更改该应用程序及其配置以使其支持多租户。此外，无论你是在单租户应用程序中还是在多租户应用程序中提供身份验证，Azure AD 都将为所有目录中的所有令牌使用相同的签名密钥。

本文档中列出的每个方案都包括一个小节，用以介绍其设置要求。
## <a name="application-types-and-scenarios"></a>应用程序类型和方案

本文档所述的每个方案都可以使用各种语言和平台进行开发，并且 [GitHub 上提供了每个方案的完整代码示例](https://github.com/AzureAD)。此外，如果你的应用程序需要某个端到端方案的特定片段，在大多数情况下都可以独立添加该功能。例如，如果你有一个调用某个 Web API 的本机应用程序，则你可以轻松添加也调用该 Web API 的网站。下面的图示介绍了这些方案和应用程序类型，以及可以如何添加各种组件：

![应用程序类型和方案](./media/active-directory-authentication-scenarios/application_types_and_scenarios.png)

下面是 Azure AD 支持的五种主要应用程序方案：

- [Web 浏览器到 Web 应用程序](#web-browser-to-web-application)：用户需要登录到由 Azure AD 保护的 Web 应用程序。

- [单页面应用程序 (SPA)](#single-page-application-spa)：用户需要登录到由 Azure AD 保护的单页面应用程序。

- [本机应用程序到 Web API](#native-application-to-web-api)：在手机、平板电脑或电脑上运行的本机应用程序需要对用户进行身份验证以从 Azure AD 所保护的 Web API 获取资源。

- [Web 应用程序到 Web API](#web-application-to-web-api)：Web 应用程序需要从 Azure AD 所保护的 Web API 获取资源。

- [后台或服务器应用程序到 Web API](#daemon-or-server-application-to-web-api)：没有 Web 用户界面的后台应用程序或服务器应用程序需要从 Azure AD 所保护的 Web API 获取资源。

### <a name="web-browser-to-web-application"></a>Web 浏览器到 Web 应用

本部分介绍了在 Web 浏览器到 Web 应用程序方案中对用户进行身份验证的应用程序。在此方案中，Web 应用程序指示用户的浏览器让用户登录到 Azure AD 中。Azure AD 通过用户的浏览器返回一个登录响应，该响应在一个安全令牌中包含了关于用户的声明。此方案支持使用 WS-Federation、SAML 2.0 和 OpenID Connect 协议进行登录。


#### 图表
![浏览器到 Web 应用程序的身份验证流](./media/active-directory-authentication-scenarios/web_browser_to_web_api.png)


#### 协议流的说明


1. 当用户访问应用程序并需要登录时，系统会通过一个登录请求将其重定向到 Azure AD 中的身份验证终结点。


2. 用户在登录页面上进行登录。


3. 如果身份验证成功，则 Azure AD 将创建一个身份验证令牌并将登录响应返回到应用程序的回复 URL（已在 Azure 经典管理门户中配置）。对于生产应用程序，此回复 URL 应当采用 HTTPS 格式。返回的令牌包括应用程序对该令牌进行验证所需的关于用户和 Azure AD 的声明。


4. 应用程序使用 Azure AD 的联合元数据文档中提供的公用签名密钥和颁发者信息对令牌进行验证。在应用程序对令牌进行验证后，Azure AD 将通过该用户启动一个新会话。该会话允许用户访问应用程序，直到会话过期。


#### 代码示例


请参阅 Web 浏览器到 Web 应用方案的代码示例。另外，请经常回来查看 - 我们会不时地添加新示例。[Web 浏览器到 Web 应用](/documentation/articles/active-directory-code-samples/#web-browser-to-web-application)。


#### 注册


- 单租户：如果你在构建仅供你的组织使用的应用程序，则必须使用 Azure 经典管理门户在你公司的目录中注册该应用程序。


- 多租户：如果你在构建可以由组织外部用户使用的应用程序，则必须在你公司的目录中注册该应用程序，并且还必须在要使用该应用程序的每个组织的目录中注册该应用程序。若要使你的应用程序在客户的目录中可用，你可以提供一个供客户使用的注册流程，让客户许可你的应用程序的要求。当他们针对你的应用程序进行注册时，系统会向他们显示一个对话框，其中显示了应用程序要求的权限，之后是表示许可的选项。可能会要求其他组织中的管理员表示许可，具体取决于所需的权限。当用户或管理员表示许可后，将在其目录中注册该应用程序。

#### 令牌过期

当 Azure AD 颁发的令牌的生存期过期时，用户的会话便过期。如果需要，你的应用程序可以缩短此时段，例如，根据用户处于不活动状态的时长注销用户。当会话过期时，将提示用户重新登录。





### <a name="single-page-application-spa"></a>单页面应用程序 (SPA)


本部分介绍使用 Azure AD 来保护其 Web API 后端的单页面应用程序的身份验证。通常将单页面应用程序构建为一个 JavaScript 表示层（前端），该表示层不仅在浏览器中运行，还在一个在服务器上运行并实现应用程序业务逻辑的 Web API 中运行。在此方案中，当用户登录时，JavaScript 前端使用 [JavaScript (ADAL.JS) 的 Active Directory 身份验证库](https://github.com/AzureAD/azure-activedirectory-library-for-js/tree/dev)预览版和 OAuth 2.0 隐式授权协议从 Azure AD 获取一个 ID 令牌 (id\_token)。该令牌随后被缓存，当客户端调用使用 OWIN 中间件进行保护的 Web API 后端时，客户端将该令牌作为持有者令牌附加到请求。


#### 图表

![单页面应用程序图示](./media/active-directory-authentication-scenarios/single_page_app.png)

#### 协议流的说明

1. 用户导航到 Web 应用程序。


2. 应用程序将 JavaScript 前端（表示层）返回到浏览器。


3. 用户通过单击登录链接等方式启动登录。浏览器将 GET 发送到 Azure AD 授权终结点来请求一个 ID 令牌。此请求将客户端 ID 和答复 URL 包含在查询参数中。


4. Azure AD 根据在 Azure 经典管理门户中配置的已注册答复 URL 来验证答复 URL。


5. 用户在登录页面上进行登录。


6. 如果身份验证成功，Azure AD 将创建一个 ID 令牌，并将其作为 URL 片段 (#) 返回到应用程序的答复 URL。对于生产应用程序，此回复 URL 应当采用 HTTPS 格式。返回的令牌包括应用程序对该令牌进行验证所需的关于用户和 Azure AD 的声明。


7. 在浏览器中运行的 JavaScript 客户端代码从用于安全调用应用程序的 Web API 后端的响应中提取令牌。


8. 浏览器使用 Authorization 标头中的访问令牌来调用应用程序的 Web API 后端。

#### 代码示例


请参阅单页面应用程序 (SPA) 方案的代码示例。请经常回来查看 - 我们会不时地添加新示例。[单页面应用程序 (SPA)](/documentation/articles/active-directory-code-samples/#single-page-application-spa)。


#### 注册


- 单租户：如果你在构建仅供你的组织使用的应用程序，则必须使用 Azure 经典管理门户在你公司的目录中注册该应用程序。


- 多租户：如果你在构建可以由组织外部用户使用的应用程序，则必须在你公司的目录中注册该应用程序，并且还必须在要使用该应用程序的每个组织的目录中注册该应用程序。若要使你的应用程序在客户的目录中可用，你可以提供一个供客户使用的注册流程，让客户许可你的应用程序的要求。当他们针对你的应用程序进行注册时，系统会向他们显示一个对话框，其中显示了应用程序要求的权限，之后是表示许可的选项。可能会要求其他组织中的管理员表示许可，具体取决于所需的权限。当用户或管理员表示许可后，将在其目录中注册该应用程序。。

注册应用程序之后，必须将其配置为使用 OAuth 2.0 隐式授予协议。默认情况下，应用程序禁用此协议。若要为你的应用程序启用 OAuth2 隐式授予协议，从 Azure 经典管理门户中下载该协议的应用程序清单，将“oauth2AllowImplicitFlow”值设置为 true，然后将该清单上载回到门户预览。


#### 令牌过期

当你使用 ADAL.js 来管理 Azure AD 身份验证时，你将从几个功能中获益，这些功能不仅有助于刷新过期的令牌，还有助于为可能被应用程序调用的其他 Web API 资源获取令牌。当用户成功向 Azure AD 进行身份验证时，将在浏览器与 Azure AD 之间建立一个通过 Cookie 进行保护的会话。请务必注意，此会话存在于用户与 Azure AD 之间，而非存在于用户与服务器上运行的网站之间。当一个令牌过期时，ADAL.js 将使用此会话以无提示方式获取另一个令牌。通过使用隐藏的 iFrame 来发送和接收使用 OAuth 隐式授予协议的请求来执行上述操作。对于应用程序调用的其他 Web API 资源，只要它们支持跨域资源共享 (CORS)，在用户的目录中注册，并在登录期间获得用户的所需许可，ADAL.js 就可以使用此相同的机制以无提示方式为这些资源从 Azure AD 中获取访问令牌。


### <a name="native-application-to-web-api"></a>本机应用程序到 Web API


本部分介绍了代表用户调用某个 Web API 的本机应用程序。此方案是基于带有公共客户端的 OAuth 2.0 授权代码授予类型构建的，如 [OAuth 2.0 规范的第 4.1 部分](http://tools.ietf.org/html/rfc6749)所述。本机应用程序使用 OAuth 2.0 协议为用户获取访问令牌。然后会在请求中将此访问令牌发送到 Web API，后者对用户进行授权并返回所需的资源。

#### 图表

![本机应用程序到 Web API 图示](./media/active-directory-authentication-scenarios/native_app_to_web_api.png)

#### 本机应用程序到 API 的身份验证流程

#### 协议流的说明


如果你使用 AD 身份验证库，则会替你处理下面所述的大多数协议细节，例如浏览器弹出窗口、令牌缓存以及对刷新令牌的处理。

1. 本机应用程序使用浏览器弹出窗口向 Azure AD 中的授权终结点发出请求。此请求包括客户端 ID 和本机应用程序的重定向 URI（与经典管理门户中显示的相同），以及 Web API 的应用程序 ID URI。如果用户尚未登录，同样将会提示他们登录


2. Azure AD 对用户进行身份验证。如果它是一个多租户应用程序并且需要许可才能使用应用程序，则用户将需要表示许可（如果他们尚未如此做）。在用户表示许可并成功进行身份验证后，Azure AD 会将一个授权代码响应发回客户端应用程序的重定向 URI。


3. 当 Azure AD 将授权代码响应发回重定向 URI 时，客户端应用程序将停止浏览器交互并从响应中提取授权代码。使用此授权代码，客户端应用程序向 Azure AD 的令牌终结点发送请求，请求中包括授权代码、关于客户端应用程序的详细信息（客户端 ID 和重定向 URI）以及所需的资源（Web API 的应用程序 ID URI）。


4. Azure AD 对授权代码和关于客户端应用程序和 Web API 的信息进行验证。当验证成功时，Azure AD 返回两个令牌：一个 JWT 访问令牌和一个 JWT 刷新令牌。此外，Azure AD 还将返回关于用户的基本信息，例如其显示名称和租户 ID。


5. 通过 HTTPS，客户端应用程序使用返回的 JWT 访问令牌在发往 Web API 的请求的 Authorization 标头中添加一个具有“Bearer”限定符的 JWT 字符串。然后，Web API 对 JWT 令牌进行验证，并且如果验证成功，则返回所需的资源。


6. 当访问令牌过期时，客户端应用程序将收到一个错误，指出用户需要重新进行身份验证。如果应用程序具有有效的刷新令牌，则可以使用它来获取新的访问令牌，系统不会提示用户重新登录。如果刷新令牌过期，则应用程序将再次需要以交互方式对用户进行身份验证。


> [AZURE.NOTE] Azure AD 颁发的刷新令牌可以用来访问多个资源。例如，如果你的某个客户端应用程序有权调用两个 Web API，则同样可以使用刷新令牌来获取其他 Web API 的访问令牌。


#### 代码示例


请参阅本机应用程序到 Web API 方案的代码示例。另外，请经常回来查看 - 我们会不时地添加新示例。[本机应用程序到 Web API](/documentation/articles/active-directory-code-samples/#native-application-to-web-api)。


#### 注册


- 单租户：本机应用程序和 Web API 必须在 Azure AD 的同一个目录中进行注册。可以对 Web API 进行配置以公开一组权限，然后使用这些权限来限制本机应用程序对其资源的访问。然后，客户端应用程序在 Azure 经典管理门户的“对其他应用程序的权限”下拉菜单中选择所需的权限。


- 多租户：首先，本机应用程序只在开发人员或发布者的目录中进行注册。其次，本机应用程序在配置后会指示它在正常运行时所需的权限。当目标目录中的用户或管理员表示许可应用程序的要求时（这将使应用程序可供其组织使用），此必需权限列表将显示在一个对话框中。某些应用程序只需要用户级权限，组织中的任何用户都可以表示许可。另外一些应用程序需要管理员级权限，组织中的用户无法表示许可。只有目录管理员可以对需要此级别的权限的应用程序表示许可。当用户或管理员表示许可后，才会在其目录中注册该 Web API。


#### 令牌过期


当本机应用程序使用其授权代码来获取 JWT 访问令牌时，它还会收到一个 JWT 刷新令牌。当访问令牌过期时，可以使用刷新令牌来重新对用户进行身份验证，不需要他们重新登录。然后将使用此刷新令牌对用户进行身份验证，这将生成新的访问令牌和刷新令牌。





### <a name="web-application-to-web-api"></a> Web 应用到 Web API

本部分介绍了需要从 Web API 获取资源的 Web 应用程序。在此方案中，Web 应用程序可以使用两种标识类型进行身份验证并调用 Web API：应用程序标识或委托用户标识。

应用程序标识：此方案使用 OAuth 2.0 客户端凭据授予作为应用程序进行身份验证并访问 Web API。当使用应用程序标识时，Web API 只能检测到 Web 应用程序在调用它，因为 Web API 不会收到关于用户的任何信息。如果应用程序收到关于用户的信息，则该信息将通过应用程序协议发送，并且 Azure AD 不会对其进行签名。Web API 相信 Web 应用程序已对用户进行了身份验证。因此，此模式称为受信任的子系统。

委派的用户标识：此方案可以通过两种方式完成：OpenID Connect 和带有机密客户端的 OAuth 2.0 代码授权。Web 应用程序为用户获取访问令牌，该令牌将向 Web API 证明用户已成功通过了 Web 应用程序的身份验证并且 Web 应用程序能够获取委托用户标识来调用 Web API。然后会在请求中将此访问令牌发送到 Web API，后者对用户进行授权并返回所需的资源。

#### 图表

![Web 应用程序到 Web API 图示](./media/active-directory-authentication-scenarios/web_app_to_web_api.png)



#### 协议流的说明

下面的流对应用程序标识类型和委托用户标识类型都进行了讨论。它们之间的主要区别是，委托用户标识必须先获取一个授权代码，然后用户才能登录并访问 Web API。

##### 带有 OAuth 2.0 客户端凭据授权的应用程序标识

1. 用户在 Web 应用程序中登录到 Azure AD（请参阅前面的 [Web 浏览器到 Web 应用程序](#web-browser-to-web-application)）。


2. Web 应用程序需要获取访问令牌，以便通过 Web API 进行身份验证并检索所需的资源。它向 Azure AD 的令牌终结点发出一个请求，在其中提供凭据、客户端 ID 以及 Web API 的应用程序 ID URI。


3. Azure AD 对应用程序进行身份验证并返回用来调用 Web API 的 JWT 访问令牌。


4. 通过 HTTPS，Web 应用程序使用返回的 JWT 访问令牌在发往 Web API 的请求的 Authorization 标头中添加一个具有“Bearer”限定符的 JWT 字符串。然后，Web API 对 JWT 令牌进行验证，并且如果验证成功，则返回所需的资源。

##### 采用 OpenID Connect 的委托用户标识

1. 用户使用 Azure AD 登录到 Web 应用程序（请参阅前面的 [Web 浏览器到 Web 应用程序](#web-browser-to-web-application)）。如果 Web 应用程序的用户尚未许可允许 Web 应用程序代表自己调用 Web API，则需要用户表示许可。应用程序将显示它要求的权限，并且如果这些权限中有任何一个是管理员级权限，则目录中的普通用户将无法表示许可。此许可过程仅适用于多租户应用程序，不适用于单租户应用程序，因为单租户应用程序那时已经具有了必需的权限。当用户登录后，Web 应用程序将收到一个 ID 令牌，其中包含关于用户的信息以及授权代码。


2. 使用由 Azure AD 颁发的授权代码，Web 应用程序向 Azure AD 的令牌终结点发送请求，请求中包括授权代码、关于客户端应用程序的详细信息（客户端 ID 和重定向 URI）以及所需的资源（Web API 的应用程序 ID URI）。


3. Azure AD 对授权代码和关于 Web 应用程序和 Web API 的信息进行验证。当验证成功时，Azure AD 返回两个令牌：一个 JWT 访问令牌和一个 JWT 刷新令牌。


4. 通过 HTTPS，Web 应用程序使用返回的 JWT 访问令牌在发往 Web API 的请求的 Authorization 标头中添加一个具有“Bearer”限定符的 JWT 字符串。然后，Web API 对 JWT 令牌进行验证，并且如果验证成功，则返回所需的资源。

##### 采用 OAuth 2.0 授权代码授权的委托用户标识

1. 用户已登录到 Web 应用程序，该应用程序的身份验证机制独立于 Azure AD。


2. Web 应用程序需要一个授权代码来获取访问令牌，因此，在成功进行身份验证后，它通过浏览器向 Azure AD 的授权终结点发出一个请求，其中提供了客户端 ID 和 Web 应用程序的重定向 URI。用户登录到 Azure AD。用户登录到 Azure AD。


3. 如果 Web 应用程序的用户尚未许可允许 Web 应用程序代表自己调用 Web API，则需要用户表示许可。应用程序将显示它要求的权限，并且如果这些权限中有任何一个是管理员级权限，则目录中的普通用户将无法表示许可。此许可过程仅适用于多租户应用程序，不适用于单租户应用程序，因为单租户应用程序那时已经具有了必需的权限。


4. 在用户表示许可后，Web 应用程序将收到它获取访问令牌所需的授权代码。


5. 使用由 Azure AD 颁发的授权代码，Web 应用程序向 Azure AD 的令牌终结点发送请求，请求中包括授权代码、关于客户端应用程序的详细信息（客户端 ID 和重定向 URI）以及所需的资源（Web API 的应用程序 ID URI）。


6. Azure AD 对授权代码和关于 Web 应用程序和 Web API 的信息进行验证。当验证成功时，Azure AD 返回两个令牌：一个 JWT 访问令牌和一个 JWT 刷新令牌。


7. 通过 HTTPS，Web 应用程序使用返回的 JWT 访问令牌在发往 Web API 的请求的 Authorization 标头中添加一个具有“Bearer”限定符的 JWT 字符串。然后，Web API 对 JWT 令牌进行验证，并且如果验证成功，则返回所需的资源。

#### 代码示例

请参阅 Web 应用到 Web API 方案的代码示例。另外，请经常回来查看 - 我们会不时地添加新示例。Web [应用程序到 Web API](/documentation/articles/active-directory-code-samples/#web-application-to-web-api)。


#### 注册

- 单租户：对于应用程序标识和委托用户标识这两种情况，Web 应用程序和 Web API 都必须在 Azure AD 的同一个目录中进行注册。可以对 Web API 进行配置以公开一组权限，然后使用这些权限来限制 Web 应用程序对其资源的访问。如果使用的是委托用户标识类型，则 Web 应用程序需要从 Azure 经典管理门户的“对其他应用程序的权限”下拉菜单中选择所需的权限。如果使用的是应用程序标识类型，则不需要此步骤。


- 多租户：首先，Web 应用程序在配置后会指示它在正常运行时所需的权限。当目标目录中的用户或管理员表示许可应用程序的要求时（这将使应用程序可供其组织使用），此必需权限列表将显示在一个对话框中。某些应用程序只需要用户级权限，组织中的任何用户都可以表示许可。另外一些应用程序需要管理员级权限，组织中的用户无法表示许可。只有目录管理员可以对需要此级别的权限的应用程序表示许可。当用户或管理员表示许可后，将在其目录中注册 Web 应用程序和 Web API。

#### 令牌过期

当 Web 应用程序使用其授权代码来获取 JWT 访问令牌时，它还会收到一个 JWT 刷新令牌。当访问令牌过期时，可以使用刷新令牌来重新对用户进行身份验证，不需要他们重新登录。然后将使用此刷新令牌对用户进行身份验证，这将生成新的访问令牌和刷新令牌。


### <a name="daemon-or-server-application-to-web-api"></a>后台或服务器应用程序到 Web API


本部分介绍了需要从 Web API 获取资源的后台或服务器应用程序。本部分包括两个适用的子方案：基于 OAuth 2.0 客户端凭据授权类型构建的需要调用 Web API 的后台应用程序；基于 OAuth 2.0 On-Behalf-Of 草案规范构建的需要调用 Web API 的服务器应用程序（例如 Web API）。

对于后台应用程序需要调用 Web API 的方案，了解以下几个事项非常重要。首先，用户无法与后台应用程序进行交互，因为这要求应用程序具有其自己的标识。在后台运行的批处理作业或操作系统服务都是后台应用程序的示例。此类型的应用程序通过以下方式来请求访问令牌：使用其应用程序标识并向 Azure AD 提供其客户端 ID、凭据（密码或证书）以及应用程序 ID URI。在身份验证成功后，后台应用程序会从 Azure AD 收到一个访问令牌，然后将使用该令牌来调用 Web API。

对于服务器应用程序需要调用 Web API 的方案，使用示例很有帮助。假定用户已在某个本机应用程序上通过了身份验证，并且此本机应用程序需要调用 Web API。Azure AD 颁发一个 JWT 访问令牌来调用 Web API。如果 Web API 需要调用另一个下游 Web API，则它可以使用 on-behalf-of 流来委托用户的标识并通过第二层 Web API 进行身份验证。

#### 图表

![后台或服务器应用程序到 Web API 图示](./media/active-directory-authentication-scenarios/daemon_server_app_to_web_api.png)

#### 协议流的说明

##### 带有 OAuth 2.0 客户端凭据授权的应用程序标识

1. 首先，服务器应用程序需要自行通过 Azure AD 进行身份验证，不涉及任何人为交互（例如交互式登录对话框）。它向 Azure AD 的令牌终结点发出一个请求，在其中提供凭据、客户端 ID 以及应用程序 ID URI。


2. Azure AD 对应用程序进行身份验证并返回用来调用 Web API 的 JWT 访问令牌。


3. 通过 HTTPS，Web 应用程序使用返回的 JWT 访问令牌在发往 Web API 的请求的 Authorization 标头中添加一个具有“Bearer”限定符的 JWT 字符串。然后，Web API 对 JWT 令牌进行验证，并且如果验证成功，则返回所需的资源。


##### 采用 OAuth 2.0 On-Behalf-Of 草案规范的委托用户标识

下面讨论的流假定用户已在另一应用程序（例如本机应用程序）上通过了身份验证，并且已使用其用户标识来获取第一层 Web API 的访问令牌。

1. 本机应用程序将访问令牌发送到第一层 Web API。


2. 第一层 Web API 向 Azure AD 的令牌终结点发送一个请求，其中提供了其客户端 ID 和凭据以及用户的访问令牌。此外，将随请求发送一个 on\_behalf\_of 参数，此参数指示 Web API 是在代表原始用户请求新令牌以调用下游 Web API。


3. Azure AD 验证第一层 Web API 是否有权访问第二层 Web API 并对请求进行验证，并将一个 JWT 访问令牌和一个 JWT 刷新令牌返回给第一层 Web API。


4. 然后，第一层 Web API 使用 HTTPS 通过将令牌字符串附加到请求的 Authorization 标头中来调用第二层 Web API。只要访问令牌和刷新令牌有效，第一层 Web API 就可以继续调用第二层 Web API。

#### 代码示例

请参阅后台或服务器应用程序到 Web API 方案的代码示例。另外，请经常回来查看 - 我们会不时地添加新示例。[服务器或守护程序应用程序到 Web API](/documentation/articles/active-directory-code-samples/#server-or-daemon-application-to-web-api)

#### 注册

- 单租户：对于应用程序标识和委托用户标识这两种情况，后台或服务器应用程序都必须在 Azure AD 的同一个目录中进行注册。可以对 Web API 进行配置以公开一组权限，然后使用这些权限来限制后台或服务器对其资源的访问。如果使用的是委托用户标识类型，则服务器应用程序需要从 Azure 经典管理门户的“对其他应用程序的权限”下拉菜单中选择所需的权限。如果使用的是应用程序标识类型，则不需要此步骤。


- 多租户：首先，后台或服务器应用程序在配置后会指示它在正常运行时所需的权限。当目标目录中的用户或管理员表示许可应用程序的要求时（这将使应用程序可供其组织使用），此必需权限列表将显示在一个对话框中。某些应用程序只需要用户级权限，组织中的任何用户都可以表示许可。另外一些应用程序需要管理员级权限，组织中的用户无法表示许可。只有目录管理员可以对需要此级别的权限的应用程序表示许可。当用户或管理员表示许可后，将在其目录中注册这两个 Web API。

#### 令牌过期

当第一个应用程序使用其授权代码来获取 JWT 访问令牌时，它还会收到一个 JWT 刷新令牌。当访问令牌过期时，可以使用刷新令牌来重新对用户进行身份验证，不会提示他们输入凭据。然后将使用此刷新令牌对用户进行身份验证，这将生成新的访问令牌和刷新令牌。

## 另请参阅

[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)

[Azure Active Directory 代码示例](/documentation/articles/active-directory-code-samples/)

[有关 Azure AD 中签名密钥滚动更新的重要信息](https://msdn.microsoft.com/zh-cn/library/azure/dn641920.aspx)

[Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/zh-cn/library/azure/dn645545.aspx)

<!---HONumber=Mooncake_0411_2016-->