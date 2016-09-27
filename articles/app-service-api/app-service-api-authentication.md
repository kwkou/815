<properties
	pageTitle="Azure 应用服务中 API 应用的身份验证和授权 | Azure"
	description="了解 Azure 应用服务为 API 应用提供的身份验证和授权服务。"
	services="app-service\api"
	documentationCenter=".net"
	authors="tdykstra"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-api"
	ms.date="05/23/2016"
	wacn.date="09/26/2016"/>

# Azure 应用服务中 API 应用的身份验证和授权

## 概述 

> [AZURE.NOTE] 本主题将会移入 [App Service Authentication / Authorization](/documentation/articles/app-service-authentication-overview/)（应用服务身份验证/授权）合并主题，其中涉及 Web 应用、移动应用和 API 应用。

Azure 应用服务提供内置的身份验证与授权服务，可实现 [OAuth 2.0](#oauth) 和 [OpenID Connect](#oauth)。本文介绍 Azure 应用服务中 API 应用可用的服务和选项。

下图演示了应用服务身份验证的几个重要特征：

* 预处理传入的 API 请求，这意味着，可以处理应用服务支持的任何语言或框架。
* 提供多个选项用于确定要在自身代码中完成多少身份验证工作。
* 适用于最终用户与服务帐户身份验证。
* 支持五个标识提供者：Azure Active Directory 和 Microsoft 帐户。
* 对 API 应用、Web 应用和移动应用的工作原理相同。

![](./media/app-service-api-authentication/api-apps-overview.png)

## 不限语言

应用服务身份验证处理在请求进入 API 应用之前进行，这意味着，无论 API 应用是以哪种语言或框架编写，都可以使用身份验证功能。API 可以基于 ASP.NET、Java、Node.js，或应用服务支持的任何框架。

应用服务在 HTTP 请求的 Authorization 标头中传递 JSON Web 令牌 (JWT)，以任何语言或框架编写的代码都可以从该令牌获取所需的信息。此外，应用服务通过设置一些特殊标头（如下所示）来方便访问最常用的声明：

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID
 
在 .NET API 中，可以使用 `Authorize` 属性，如果想要更精细的授权，可以基于声明轻松编写代码，因为 .NET 类中已经填充了声明信息。

## <a name="multiple-protection-options"></a>多个保护选项

应用服务可以防止匿名 HTTP 请求进入 API 应用、传递所有请求和验证请求中包含的令牌，或者不采取任何措施即放行所有请求：

1. 只允许经过身份验证的请求进入 API 应用。

	如果应用服务收到来自浏览器的匿名请求，会将其重定向到所选身份验证提供程序（Azure AD 等）的登录页。

	使用此选项时，不需要在应用中编写任何身份验证代码。由于 HTTP 标头中已提供最重要的声明，因此授权代码变得相当简单。

2. 允许所有请求进入 API 应用，但会验证经过身份验证的请求，并在 HTTP 标头中传递身份验证信息。

	使用此选项可以更灵活地处理匿名请求，但如果想要防止匿名用户使用 API，则必须编写代码。由于最常用的声明在 HTTP 请求的标头中传递，因此授权代码相对简单。
	
3. 允许所有请求进入 API，不处理请求中的身份验证信息。

	此选项将身份验证和授权任务全部交由应用程序代码来处理。

在 [Azure 门户预览](https://portal.azure.cn/)上的“身份验证/授权”边栏选项卡中选择所需的选项。

![](./media/app-service-api-authentication/authblade.png)

如果使用选项 1 和 2，请打开“应用服务身份验证”，然后在“请求未经身份验证时需执行的操作”下拉列表中，选择“登录”或“允许请求(无操作)”。如果选择“登录”，则必须选择身份验证提供程序并配置该提供程序。

![](./media/app-service-api-authentication/actiontotake.png)

有关如何配置身份验证的详细信息，请参阅 [How to configure your App Service application to use Azure Active Directory login](/documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/)（如何将应用服务应用程序配置为使用 Azure Active Directory 登录）。此文章适用于 API 应用和移动应用，并链接到有关其他身份验证提供程序的其他文章。
 
## <a id="internal"></a>服务帐户身份验证

应用服务身份验证适用于从某个 API 应用调用另一个 API 应用之类的内部方案。在此方案中，可以使用服务帐户凭据（而不是用户凭据）来获取令牌。在 Azure Active Directory 中，服务帐户也称为 *服务主体* ，使用此类帐户的身份验证也称为服务到服务方案。

对于服务到服务方案，请使用 Azure Active Directory 保护所调用的 API 应用，并在调用 API 应用时提供 AAD 服务主体授权令牌。通过提供客户端 ID 和客户端机密，可以从 AAD 应用程序获取令牌。不需要像过去处理移动服务 Zumo 令牌时那样使用仅限 Azure 的特殊代码。[Service principal authentication for API Apps](/documentation/articles/app-service-api-dotnet-service-principal-auth/)（API 应用的服务主体身份验证）教程讲解了这种使用 ASP.NET API 应用的方案示例。

若要处理服务到服务方案但不使用应用服务身份验证，请使用客户端证书或基本身份验证。有关 Azure 中客户端证书的信息，请参阅 [How To Configure TLS Mutual Authentication for Web Apps](/documentation/articles/app-service-web-configure-tls-mutual-auth/)（如何为 Web 应用配置 TLS 相互身份验证）。有关 ASP.NET 中基本身份验证的信息，请参阅 [Authentication Filters in ASP.NET Web API 2](http://www.asp.net/web-api/overview/security/authentication-filters)（ASP.NET Web API 2 中的身份验证筛选器）。

## 移动客户端身份验证

有关如何处理来自移动客户端的身份验证的信息，请参阅[有关移动应用身份验证的文档](/documentation/articles/app-service-mobile-ios-get-started-users/)。移动应用和 API 应用的应用服务身份验证运用相同的工作原理。
  
## 详细信息

有关 Azure 应用服务中的身份验证和授权的详细信息，请参阅以下资源：

* [Expanding App Service authentication / authorization](https://azure.microsoft.com/blog/announcing-app-service-authentication-authorization/)（扩展应用服务身份验证/授权）
* [How to configure your App Service application to use Azure Active Directory login](/documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/)（如何将应用服务应用程序配置为使用 Azure Active Directory 登录）（页面顶部提供了其他身份验证提供程序的链接）。

<a name="oauth"></a>

有关 OAuth 2.0、OpenID Connect 和 JSON Web 令牌 (JWT) 的详细信息，请参阅以下资源。

* [Getting started with OAuth 2.0](http://shop.oreilly.com/product/0636920021810.do "OAuth 2.0 入门")（OAuth 2.0 入门）
* [Introduction to OAuth2, OpenID Connect and JSON Web Tokens (JWT)](http://www.pluralsight.com/courses/oauth2-json-web-tokens-openid-connect-introduction)（OAuth2、OpenID Connect 和 JSON Web 令牌 (JWT) 简介）- PluralSight 课程
* [Building and Securing a RESTful API for Multiple Clients in ASP.NET - PluralSight course](http://www.pluralsight.com/courses/building-securing-restful-api-aspdotnet)（在 ASP.NET 中生成和保护多个客户端的 RESTful API）- PluralSight 课程

有关 Azure Active Directory 的详细信息，请参阅以下资源。

* [Azure AD scenarios](/documentation/articles/active-directory-authentication-scenarios/)（Azure AD 方案）
* [Azure AD developers' guide](/documentation/articles/active-directory-developers-guide/)（Azure AD 开发人员指南）
* [Azure AD 示例](https://github.com/azure-samples?query=active-directory)

## 后续步骤

本文说明了可用于 API 应用的应用服务身份验证和授权功能。入门系列教程的下一篇文章说明如何[在应用服务 API 应用中实现用户身份验证](/documentation/articles/app-service-api-dotnet-user-principal-auth/)。

<!---HONumber=Mooncake_0919_2016-->