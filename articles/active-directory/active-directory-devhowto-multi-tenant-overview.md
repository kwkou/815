<properties
   pageTitle="如何构建可使任何 Azure Active Directory 用户登录的应用程序 | Azure"
   description="有关如何构建一个可使用户从任何 Azure Active Directory 租户登录的应用程序（也称为多租户应用程序）的分步说明。"
   services="active-directory"
   documentationCenter=""
   authors="skwan"
   manager="mbaldwin"
   editor=""/>  


<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="10/11/2016"
   ms.author="skwan;bryanla"
   wacn.date="10/31/2016"/>

# 如何使用多租户应用程序模式将任何 Azure Active Directory (AD) 用户登录
如果你向许多组织提供软件即服务应用程序，可以将应用程序配置为可接受来自任何 Azure AD 租户的登录。在 Azure AD 中，这称为使应用程序成为多租户应用程序。任何 Azure AD 租户中的用户在同意配合你的应用程序使用其帐户之后，便可登录到你的应用程序。

如果现有应用程序拥有自身的帐户系统，或支持来自其他云提供程序的其他类型的登录方式，则添加来自任何租户的 Azure AD 登录相当简单，只需注册应用程序、通过 OAuth2、OpenID Connect 或 SAML 添加登录代码，然后在应用程序中放置一个“登录到 Microsoft”按钮即可。单击下面的按钮详细了解如何为应用程序创建品牌。

[![登录按钮][AAD-Sign-In]][AAD-App-Branding]


本文假设你已熟悉如何为 Azure AD 构建单租户应用程序。如果你不熟悉，请返回[开发人员指南主页][AAD-Dev-Guide]，然后尝试学习其中一篇快速入门！

将应用程序转换成 Azure AD 多租户应用包括四个简单的步骤：

1.	将应用程序注册更新为多租户
2.	将代码更新为向 /common 终结点发送请求
3.	将代码更新为处理多个颁发者值
4.	了解用户和管理员的同意意向并进行适当的代码更改

让我们详细了解每个步骤。你也可以直接跳转到[此多租户示例列表][AAD-Samples-MT]。

## 将注册更新为多租户
Azure AD 中的 Web 应用/API 注册默认为单租户。可以将注册转换为多租户，方法是在 [Azure 经典管理门户][AZURE-classic-portal]中的应用程序注册配置页上，找出“应用程序是多租户的”参数并将其设置为“是”。

注意：在将某个应用程序转换为多租户之前，Azure AD 要求该应用程序的应用 ID URI 全局唯一。应用 ID URI 是在协议消息中标识应用程序的方式之一。就单租户应用而言，应用 ID URI 在该租户中保持唯一便已足够。就多租户应用程序而言，该 URI 必须全局唯一，以便 Azure AD 能够在所有租户中找到该应用程序。系统通过要求应用 ID URI 必须具有与已验证 Azure AD 租户域匹配的主机名，来强制实施全局唯一性。例如，如果租户的名称为 contoso.partner.onmschina.cn，则有效的应用 ID URI 为 `https://contoso.partner.onmschina.cn/myapp`。如果租户具有已验证的域 `contoso.com`，则有效的应用 ID URI 也是 `https://contoso.com/myapp`。如果应用 ID URI 不遵循此模式，则将应用程序设置为多租户就会失败。

默认情况下，本机客户端注册是多租户的。你不需要采取任何措施将本机客户端应用程序注册转换为多租户。

## 将代码更新为向 /common 发送请求
在单租户应用程序中，登录请求将发送到租户的登录终结点。以 contoso.partner.onmschina.cn 为例，终结点将是：

    https://login.microsoftonline.com/contoso.partner.onmschina.cn

发送到租户终结点的请求可以让该租户中的用户（或来宾）登录该租户中的应用程序。使用多租户应用程序时，应用程序事先并不知道用户来自哪个租户，因此无法将请求发送到租户的终结点。取而代之的是，请求将发送到在所有 Azure AD 租户之间多路复用的终结点：

    https://login.microsoftonline.com/common

当 Azure AD 在 /common 终结点上收到请求时，将会使用户登录，因而可以发现用户来自哪个租户。/common 终结点可与 Azure AD 支持的所有身份验证协议配合使用：OpenID Connect、OAuth 2.0、SAML 2.0 和 WS 联合身份验证。

然后，对应用程序做出的登录响应包含代表该用户的令牌。令牌中的颁发者值告知应用程序该用户来自哪个租户。从 /common 终结点返回响应时，令牌中的颁发者值将与用户的租户对应。请务必注意，/common 终结点不是租户，也不是颁发者，而只是一个多路复用器。使用 /common 时，需要更新应用程序中用于验证令牌的逻辑，以便考虑到这一点。

如前所述，多租户应用程序也应该为用户提供一致的登录体验，并遵循 Azure AD 应用程序的品牌指导原则。单击下面的按钮详细了解如何为应用程序创建品牌。

[![登录按钮][AAD-Sign-In]][AAD-App-Branding]

让我们更详细地了解 /common 终结点的用法和代码实现。

## 将代码更新为处理多个颁发者值
Web 应用程序和 Web API 接收并验证来自 Azure AD 的令牌。

> [AZURE.NOTE] 尽管本机客户端应用程序从 Azure AD 请求并接收令牌，但它们这样做是为了将令牌发送到 API 来进行验证。本机应用程序不会验证令牌，并且必须将它们视为不透明。

让我们看看应用程序如何验证它从 Azure AD 接收的令牌。单租户应用程序通常采用类似于下面的终结点值：

    https://login.microsoftonline.com/contoso.partner.onmschina.cn

并使用该值构造元数据 URL（在本例中为 OpenID Connect），例如：

    https://login.microsoftonline.com/contoso.partner.onmschina.cn/.well-known/openid-configuration

以下载用于验证令牌的两项关键信息：租户的签名密钥和颁发者值。每个 Azure AD 租户使用以下格式的唯一颁发者值：

    https://sts.chinacloudapi.cn/31537af4-6d77-4bb9-a681-d2394888ea26/

其中，GUID 值是租户的租户 ID 重命名安全版本。如果你单击上面的 `contoso.partner.onmschina.cn` 元数据链接，就可以在文档中看到此颁发者值。

单租户应用程序在验证令牌时，会根据元数据文档中的签名密钥检查令牌的签名，并确保令牌中的颁发者值与在元数据文档中找到的颁发者值匹配。

由于 /common 终结点既不对应于租户也不是颁发者，因此在检查 /common 的元数据中的颁发者值时，它拥有的是一个模板化的 URL 而不是实际值：

    https://sts.chinacloudapi.cn/{tenantid}/

因此，多租户应用程序无法仅通过将元数据中的颁发者值与令牌中的 `issuer` 值进行匹配来验证令牌。多租户应用程序需要一个逻辑来根据颁发者值的租户 ID 部分来确定哪些颁发者值有效、哪些颁发者值无效。

例如，如果多租户应用程序只允许从已注册其服务的特定租户登录，则必须检查令牌中的颁发者值或 `tid` 声明值，以确保该租户在其订户列表中。如果多租户应用程序只处理个人而不根据租户做出任何访问决策，则可以完全忽略颁发者值。

在本文末尾[相关内容](#related-content)部分所述的多租户示例中，为了使任何 Azure AD 租户都能登录，已禁用颁发者验证。

现在，让我们看看用户登录多租户应用程序时的体验。

## 了解用户和管理员同意
若要让用户登录 Azuer AD 中的某个应用程序，必须以用户租户的形式表示该应用程序。这样，组织便可以采取一些措施，例如，当其租户中的用户登录应用程序时应用唯一策略。对于单租户应用程序，此注册过程相当简单，也就是在 [Azure 经典管理门户][AZURE-classic-portal]中注册应用程序所采用的同一个过程。

对于多租户应用程序，应用程序的初始注册过程是在开发人员使用的 Azure AD 租户中进行的。当来自不同租户的用户首次登录应用程序时，Azure AD 将要求他们同意应用程序所请求的权限。如果他们同意，系统将在用户的租户中创建一个称为*服务主体*的应用程序表示形式，然后登录可继续进行。系统还会在记录用户对应用程序的同意意向的目录中创建委托。有关应用程序的 Application 和 ServicePrincipal 对象的详细信息，请参阅 [Application Objects and Service Principal Objects][AAD-App-SP-Objects]（应用程序对象和服务主体对象）。

![同意单层应用][Consent-Single-Tier]

这种同意体验受应用程序请求的权限的影响。Azure AD 支持两种类型的权限 - 仅限应用的权限和委托的权限：

- 委托的权限授权应用程序充当登录用户来执行该用户所能执行的一部分操作。例如，你可以向应用程序授予委托的权限来读取登录用户的日历。
- 仅限应用的权限直接授予给应用程序的标识。例如，你可以向应用程序授予仅限应用的权限来读取租户中的用户列表，并且无论是谁登录此应用程序，此应用程序都能够执行此操作。

有些权限可由普通用户同意，有些则需要租户管理员同意。

### 管理员同意
仅限应用的权限始终需要租户管理员的同意。如果应用程序请求仅限应用的权限，当普通用户尝试登录应用程序时，应用程序将收到错误消息，指出该用户无法同意。

有些委托的权限也需要租户管理员的同意。例如，若要能够以登录用户身份写回 Azure AD，就需要租户管理员的同意。与仅限应用的权限一样，如果普通用户尝试登录请求委托权限的应用程序，而该权限需要管理员同意，则应用程序会收到错误。权限是否需要管理员同意是由发布资源的开发人员决定的，可以在该资源的文档中找到相关信息。有关介绍 Azure AD 图形 API 和 Microsoft 图形 API 可用权限的主题链接，请参阅本文的[相关内容](#related-content)部分。

如果应用程序使用需要管理员同意的权限，则应用程序中需要提供某种表示，例如可供管理员发起操作的按钮或链接。应用程序针对此操作发送的请求是一个普通的 OAuth2/OpenID Connect 授权请求，但此请求同时也包含 `prompt=admin_consent` 查询字符串参数。在管理员同意且系统已在客户的租户中创建服务主体之后，后续登录请求就不再需要 `prompt=admin_consent` 参数。由于管理员已确定可接受请求的权限，因此从该时间点之后，不再提示租户中的任何其他用户同意。

如果应用程序请求的权限不需要管理员同意，但是想要提供一种体验，让租户管理员只需“注册”应用程序一次，之后就不再提示所有其他用户行使同意权，则这些应用程序也可以使用 `prompt=admin_consent` 参数。

如果应用程序需要管理员同意，当管理员登录应用程序但系统未发送 `prompt=admin_consent` 参数时，管理员将可成功地同意此应用程序，但他们只能代表其用户帐户同意。普通用户仍然无法登录并同意该应用程序。如果你想要让租户管理员浏览你的应用程序，然后允许其他用户访问，则这种机制就相当有用。

租户管理员可以禁用普通用户同意应用程序的能力。如果禁用此功能，则始终需要管理员同意，才能在租户中设置应用程序。如果想要在禁用普通用户同意的情况下测试应用程序，可以在 [Azure 经典管理门户][AZURE-classic-portal]的 Azure AD 租户配置部分中找到配置开关。

> [AZURE.NOTE] 某些应用程序想要提供一种体验，让普通用户能够一开始即表示同意，然后应用程序可让管理员参与操作并请求需要管理员同意的权限。目前在 Azure AD 中还没有任何办法可以使用单个应用程序注册来实现此目的。即将推出的 Azure AD v2 终结点可允许应用程序在运行时（而不是在注册时）请求权限，这样会使这种方案成为可能。有关详细信息，请参阅 [Azure AD App Model v2 Developer Guide][AAD-V2-Dev-Guide]（Azure AD 应用模型 v2 开发人员指南）。

### 同意和多层应用程序
应用程序可能有多个层，每一层由其自身在 Azure AD 中的注册来表示。例如，一个调用 Web API 的本机应用程序，或者一个调用 Web API 的 Web 应用程序。在这两种情况下，客户端（本机应用或 Web 应用）将请求调用资源 (Web API) 的权限。要让客户端成功获得客户同意添加到其租户中，请求权限的所有资源必须都已在于客户的租户中。如果不符合此条件，Azuer AD 将返回错误，指出必须先添加资源。

如果逻辑应用程序包含两个或更多个应用程序注册（例如独立的客户端和资源），这可能造成问题。如何先将资源添加到客户租户中？ Azure AD 通过以单个步骤对客户端和资源行使同意权的方式来处理此情况，其中用户在同意页上看到客户端和资源两者所请求的权限的总和。若要启用此行为，资源的应用程序注册必须在其应用程序清单中以 `knownClientApplications` 形式包含客户端的应用 ID。例如：

    knownClientApplications": ["94da0930-763f-45c7-8d26-04d5938baab2"]

可以通过资源[应用程序的清单][AAD-App-Manifest]更新此属性，本文末尾的[相关内容](#related-content)部分中的多层本机客户端调用 Web API 示例中也提供了此属性的相关演示。下图提供了同意多层应用的概览：

![同意已知的多层客户端应用][Consent-Multi-Tier-Known-Client]

如果在不同的租户中注册不同的应用程序层，将发生类似的情况。例如，考虑构建一个调用 Office 365 Exchange Online API 的本机客户端应用程序的情况。若要开发该本机应用程序，然后让该本机应用程序在客户的租户中运行，必须存在 Exchange Online 服务主体。在此情况下，客户必须购买 Exchange Online，如此才能在其租户中创建服务主体。对于非 Microsoft 组织构建的 API，API 的开发人员必须提供一个可供其客户同意将其应用程序添加到客户租户中的方式，例如一个使用本文所述机制促成同意的网页。在租户中创建服务主体后，本机应用程序可以获取 API 的令牌。

下图提供了同意不同租户中注册的多层应用的概览：

![同意多层多方应用][Consent-Multi-Tier-Multi-Party]

### 吊销同意
用户和管理员可以随时吊销对应用程序的同意：

- 用户可通过将单个应用程序从其[访问面板应用程序][AAD-Access-Panel]列表中删除，来吊销对这些应用程序的访问权限。
- 管理员可以通过使用 [Azure 经典管理门户][AZURE-classic-portal]的 Azure AD 管理部分将应用程序从 Azure AD 中删除，来吊销对这些应用程序的访问权限。

如果是由管理员代表租户中的所有用户对应用程序行使同意权，用户就不能单独吊销访问权限。只有管理员才能吊销访问权限，并且只能针对整个应用程序吊销。

### 同意和协议支持
在 Azure AD 中，通过 OAuth、OpenID Connect、WS 联合身份验证和 SAML 协议来支持同意。由于 SAML 和 WS 联合身份验证协议不支持 `prompt=admin_consent` 参数，因此管理员只能通过 OAuth 和 OpenID Connect 同意。

## 多租户应用程序和缓存访问令牌
多租户应用程序也可以获取访问令牌来调用受 Azure AD 保护的 API。配合多租户应用程序使用 Active Directory 身份验证库 (ADAL) 时经常会出现一个错误，就是一开始即使用 /common 来为用户请求令牌、接收响应，然后同样使用 /common 来为同一用户请求后续令牌。由于从 Azure AD 返回的响应来自租户而不同 /common，因此 ADAL 缓存令牌时将它视为来自租户。后续为了为用户获取访问令牌而执行的 /common 调用将错过缓存项，因此系统会再次提示用户登录。为了避免缓存未命中，请确保后续为登录用户执行的调用是针对租户的终结点发出的。

## 相关内容

- [Multi-tenant application samples（多租户应用程序示例）][AAD-Samples-MT]
- [适用于应用程序的品牌准则][AAD-App-Branding]
- [Azure AD Developer's Guide（Azure AD 开发人员指南）][AAD-Dev-Guide]
- [应用程序对象和服务主体对象][AAD-App-SP-Objects]
- [将应用程序与 Azure Active Directory 集成][AAD-Integrating-Apps]
- [同意框架概述][AAD-Consent-Overview]
- [Microsoft Graph API Permission Scopes（Microsoft 图形 API 权限范围）][MSFT-Graph-AAD]
- [Azure AD Graph API Permission Scopes（Azure AD 图形 API 权限范围）][AAD-Graph-Perm-Scopes]

欢迎使用以下 Disqus 意见部分提供反馈，帮助我们改进与制作内容。

<!--Reference style links IN USE -->
[AAD-Access-Panel]: https://myapps.microsoft.com
[AAD-App-Branding]: /documentation/articles/active-directory-branding-guidelines/
[AAD-App-Manifest]: /documentation/articles/active-directory-application-manifest/
[AAD-App-SP-Objects]: /documentation/articles/active-directory-application-objects/
[AAD-Auth-Scenarios]: /documentation/articles/active-directory-authentication-scenarios/
[AAD-Consent-Overview]: /documentation/articles/active-directory-integrating-applications/#overview-of-the-consent-framework/
[AAD-Dev-Guide]: /documentation/articles/active-directory-developers-guide/
[AAD-Graph-Overview]: /documentation/articles/active-directory-graph-api/
[AAD-Graph-Perm-Scopes]: https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes
[AAD-Integrating-Apps]: /documentation/articles/active-directory-integrating-applications/
[AAD-Samples-MT]: https://azure.microsoft.com/documentation/samples/?service=active-directory&term=multitenant
[AAD-Why-To-Integrate]: /documentation/articles/active-directory-how-to-integrate/
[AZURE-classic-portal]: https://manage.windowsazure.cn
[MSFT-Graph-AAD]: https://graph.microsoft.io/zh-cn/docs/authorization/permission_scopes

<!--Image references-->
[AAD-Sign-In]: ./media/active-directory-devhowto-multi-tenant-overview/sign-in-with-microsoft-light.png
[Consent-Single-Tier]: ./media/active-directory-devhowto-multi-tenant-overview/consent-flow-single-tier.png
[Consent-Multi-Tier-Known-Client]: ./media/active-directory-devhowto-multi-tenant-overview/consent-flow-multi-tier-known-clients.png
[Consent-Multi-Tier-Multi-Party]: ./media/active-directory-devhowto-multi-tenant-overview/consent-flow-multi-tier-multi-party.png

<!--Reference style links -->

[AAD-App-Manifest]: /documentation/articles/active-directory-application-manifest/
[AAD-App-SP-Objects]: /documentation/articles/active-directory-application-objects/
[AAD-Auth-Scenarios]: /documentation/articles/active-directory-authentication-scenarios/
[AAD-Integrating-Apps]: /documentation/articles/active-directory-integrating-applications/
[AAD-Dev-Guide]: /documentation/articles/active-directory-developers-guide/
[AAD-Graph-Perm-Scopes]: https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes
[AAD-Graph-App-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity
[AAD-Graph-Sp-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity
[AAD-Graph-User-Entity]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity
[AAD-How-To-Integrate]: /documentation/articles/active-directory-how-to-integrate/
[AAD-Security-Token-Claims]:/documentation/articles/active-directory-authentication-scenarios/#claims-in-azure-ad-security-tokens/
[AAD-Tokens-Claims]: /documentation/articles/active-directory-token-and-claims/
[AAD-V2-Dev-Guide]: /documentation/articles/active-directory-appmodel-v2-overview/
[AZURE-classic-portal]: https://manage.windowsazure.cn
[Duyshant-Role-Blog]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/
[JWT]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
[O365-Perm-Ref]: https://msdn.microsoft.com/zh-cn/office/office365/howto/application-manifest
[OAuth2-Access-Token-Scopes]: https://tools.ietf.org/html/rfc6749#section-3.3
[OAuth2-AuthZ-Code-Grant-Flow]: https://msdn.microsoft.com/zh-cn/library/azure/dn645542.aspx
[OAuth2-AuthZ-Grant-Types]: https://tools.ietf.org/html/rfc6749#section-1.3
[OAuth2-Client-Types]: https://tools.ietf.org/html/rfc6749#section-2.1
[OAuth2-Role-Def]: https://tools.ietf.org/html/rfc6749#page-6
[OpenIDConnect]: http://openid.net/specs/openid-connect-core-1_0.html
[OpenIDConnect-ID-Token]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken

<!---HONumber=Mooncake_1024_2016-->
