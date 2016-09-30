<properties
	pageTitle="Azure 应用服务中的身份验证和授权 | Azure"
	description="概念性参考和概述：Azure 应用服务的身份验证/授权功能"
	services="app-service"
	documentationCenter=""
	authors="mattchenderson"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="04/25/2016"
	wacn.date="09/26/2016"/>

# Azure 应用服务中的身份验证和授权

## 什么是应用服务身份验证/授权？

应用服务身份验证/授权是一项功能，方便应用程序登录用户，避免在应用后端更改代码。该功能可以方便地保护应用程序和处理每个用户的数据。

应用服务使用联合标识，即通过第三方标识提供者来存储帐户并进行用户身份验证。应用程序依赖于提供者的标识信息，因此不需自行存储该信息。应用服务目前支持五大提供者，其中包括 Azure Active Directory 和 Microsoft Account。应用可以使用任意数目的此类标识提供者，为用户提供登录方式选项。可以集成其他标识提供者或[自己的自定义标识解决方案][custom-auth]，扩大内置支持。

参阅以下教程之一即可立即入门：

- [Add authentication to your iOS app][iOS]（向 iOS 应用添加身份验证）（或者其他针对 [Android]、[Windows]、[Xamarin.iOS]、[Xamarin.Android]、[Xamarin.Forms] 或 [Cordova] 应用的教程）
- [User authentication for API Apps in Azure App Service][apia-user]（Azure 应用服务中 API 应用的用户身份验证）
- [Get started with Azure App Service - Part 2][web-getstarted]（Azure 应用服务入门 - 第 2 部分）

## 应用服务中的身份验证机制

若要使用其中某个标识提供者进行身份验证，首先需对标识提供者进行配置，使之了解应用程序。然后，标识提供者会提供 ID 和机密，再由用户将其提供给应用服务。这样即可确立信任关系，使应用服务能够验证标识提供者提供的用户声明，例如身份验证令牌。

若要使用其中某个提供者来登录用户，必须将用户重定向到一个终结点，通过该终结点来登录该提供者的用户。如果客户使用的是 Web 浏览器，则可让应用服务将所有未经身份验证的用户自动定向到允许用户登录的终结点。否则需将客户定向到 `{your App Service base URL}/.auth/login/<provider>`，其中 `<provider>` 的值为 aad 或 microsoft。本文后面部分介绍移动方案和 API 方案。

通过 Web 浏览器与应用程序交互的用户会设置一个 Cookie，因此在浏览应用程序时其身份可以保持已验证状态。对于其他客户端类型，例如移动客户端类型，则会向客户端颁发 JSON Web 令牌 (JWT)，后者应存在于 `X-ZUMO-AUTH` 标头中。移动应用客户端 SDK 会为用户处理这种情况。也可以在 `Authorization` 标头中以[持有者令牌](https://tools.ietf.org/html/rfc6750)的形式直接包含 Azure Active Directory 标识令牌或访问令牌。

应用服务会验证应用程序颁发的任何 Cookie 或令牌，对用户进行身份验证。若要限制访问应用程序的人员，请参阅本文后面的[授权](#authorization)部分。

### 通过提供者 SDK 进行移动身份验证

在后端配置完所有项目以后，即可修改移动客户端，通过应用服务进行登录。可以使用下述两种方式：

- 使用给定标识提供者发布的 SDK 来建立标识，然后即可获得应用服务的访问权限。
- 使用单行代码即可让移动应用客户端 SDK 登录用户。

>[AZURE.TIP] 大多数应用程序应使用提供者 SDK，这样可以让用户在登录时获得更一致的体验，可以使用刷新支持，还可以获得提供者指定的其他权益。

使用提供者 SDK 时，用户一登录即可获得与操作系统结合更紧密的体验，而应用程序就运行在该操作系统中。这同时还提供提供者令牌以及客户端上的某些用户信息，因此可以更轻松地使用图形 API 和自定义用户体验。在博客和论坛上，偶尔也会看到此过程被称为“客户端流”或“客户端定向流”，因为客户端代码可以登录用户，还可以访问提供者令牌。

获取提供者令牌后，需将其发送到应用服务进行验证。应用服务在验证令牌后会创建新的应用服务令牌，将其返回给客户端。移动应用客户端 SDK 提供的帮助器方法可用于管理此交换，并可自动将令牌附加到针对应用程序后端的所有请求。开发人员也可选择性地保留对提供者令牌的引用。

### 不通过提供者 SDK 进行移动身份验证

如果不希望设置提供者 SDK，可以利用 Azure 应用服务的移动应用功能进行登录。移动应用客户端 SDK 会针对所选提供者打开一个 Web 视图，方便用户登录。在博客和论坛上，偶尔也会看到此过程被称为“服务器流”或“服务器定向流”，因为服务器管理用户登录过程，而客户端 SDK 从来不会收到提供者令牌。

启动此流程的代码包括在每个平台的身份验证教程中。在流程结束时，客户端 SDK 将拥有一个应用服务令牌，该令牌自动附加到针对应用程序后端的所有请求。

### 服务到服务身份验证

虽然可以为用户提供应用程序访问权限，但是也可以委托其他应用程序来调用自己的 API。例如，可以让一个 Web 应用调用另一个 Web 应用中的 API。在这种情况下，可使用服务帐户凭据而非用户凭据来获取令牌。在 Azure Active Directory 用语中，服务帐户也称为“服务主体”，使用此类帐户的身份验证也称为“服务到服务方案”。

>[AZURE.IMPORTANT] 由于移动应用在客户设备上运行，因此移动应用程序_不_算可信应用程序，不应使用服务主体流，而应使用此前详细介绍过的用户流。

对于服务到服务方案，应用服务可通过 Azure Active Directory 保护应用程序。调用应用程序只需提供 Azure Active Directory 服务主体授权令牌即可，而该令牌可通过在 Azure Active Directory 中提供客户端 ID 和客户端机密来获取。[Service principal authentication for API Apps][apia-service]（API 应用的服务主体身份验证）教程介绍了使用 ASP.NET API 应用的此类方案的示例。

若要通过应用服务身份验证处理服务到服务方案，可使用客户端证书或基本身份验证。有关 Azure 中客户端证书的信息，请参阅 [How To Configure TLS Mutual Authentication for Web Apps](/documentation/articles/app-service-web-configure-tls-mutual-auth/)（如何为 Web 应用配置 TLS 相互身份验证）。有关 ASP.NET 中基本身份验证的信息，请参阅 [Authentication Filters in ASP.NET Web API 2](http://www.asp.net/web-api/overview/security/authentication-filters)（ASP.NET Web API 2 中的身份验证筛选器）。

## <a name="authorization"></a>应用服务中的授权机制

用户可以全权控制访问应用程序的请求。可以为应用服务身份验证/授权配置以下任意行为：

- 只允许经过身份验证的请求访问应用程序。

	如果浏览器收到匿名请求，应用服务会将其重定向到所选标识提供者的页面，方便用户登录。如果请求来自移动设备，则会返回“401 未授权”HTTP 响应。

	此选项根本不需在应用中编写任何身份验证代码。如需进行更细致的授权，可通过代码获取用户信息。

- 允许所有请求访问应用程序，但需验证经过身份验证的请求，在 HTTP 标头中传递身份验证信息。

	此选项将授权决策交由应用程序代码进行，可以更灵活地处理匿名请求，但需用户编写代码。

- 允许所有请求访问应用程序，不处理请求中的身份验证信息。

	这种情况下，身份验证/授权功能处于关闭状态。身份验证和授权任务完全由应用程序代码来完成。

前述行为由 Azure 门户中的“请求未经身份验证时需执行的操作”选项控制。如果选择“使用 *提供者名称* 登录”，则所有请求都需进行身份验证。“允许请求(无操作)”将授权决策交由代码进行，但仍提供身份验证信息。若要让代码来处理一切，可禁用身份验证/授权功能。

## 处理应用程序中的用户标识

应用服务通过特殊标头将某些用户信息传递给应用程序。外部请求禁止这些标头，仅在通过应用服务身份验证/授权进行设置的情况下存在。部分标头示例如下：

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

使用任何语言或框架编写的代码均可从这些标头获取所需信息。对于 ASP.NET 4.6 应用，**ClaimsPrincipal** 会自动设置为相应的值。

应用程序也可在其 `/.auth/me` 终结点上通过 HTTP GET 获取其他用户详细信息。如果请求中包含有效的令牌，则会返回 JSON 有效负载，并详细说明所使用的提供者、基础性提供者令牌，以及一些其他的用户信息。移动应用服务器 SDK 提供处理该数据的帮助器方法。有关详细信息，请参阅 [How to use the Azure Mobile Apps Node.js SDK](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#howto-tables-getidentity)（如何使用 Azure 移动应用 Node.js SDK）和 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#user-info)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

## 文档和其他资源

### 标识提供者
以下教程说明了如何将应用服务配置为使用不同的身份验证提供者：

- [How to configure your app to use Azure Active Directory login][AAD]（如何将应用配置为使用 Azure Active Directory 登录）
- [How to configure your app to use Microsoft Account login][MSA]（如何将应用配置为使用 Microsoft Account 登录）

若要使用此处未提供的其他标识系统，也可使用[移动应用 .NET 服务器 SDK 中的预览版自定义身份验证支持][custom-auth]，后者适用于 Web 应用、移动应用或 API 应用。

### Web 应用程序
以下教程介绍如何向 Web 应用程序添加身份验证：

- [Get started with Azure App Service - Part 2][web-getstarted]（Azure 应用服务入门 - 第 2 部分）

### 移动应用程序
以下教程介绍如何通过服务器定向流向移动客户端添加身份验证：

- [Add authentication to your iOS app][iOS]（向 iOS 应用添加身份验证）
- [Add Authentication to your Android app][Android]（向 Android 应用添加身份验证）
- [Add Authentication to your Windows app][Windows]（向 Windows 应用添加身份验证）
- [Add authentication to your Xamarin.iOS app][Xamarin.iOS]（向 Xamarin.iOS 应用添加身份验证）
- [Add authentication to your Xamarin.Android app][Xamarin.Android]（向 Xamarin.Android 应用添加身份验证）
- [Add authentication to your Xamarin.Forms app][Xamarin.Forms]（向 Xamarin.Forms 应用添加身份验证）
- [Add Authentication to your Cordova app][Cordova]（向 Cordova 应用添加身份验证）

若要使用 Azure Active Directory 的客户端定向流，请参阅以下资源：

- [使用适用于 iOS 的 Active Directory 身份验证库][ADAL-iOS]
- [使用适用于 Android 的 Active Directory 身份验证库][ADAL-Android]
- [使用适用于 Windows 和 Xamarin 的 Active Directory 身份验证库][ADAL-dotnet]

### API 应用程序
以下教程介绍了如何保护 API 应用：

- [User authentication for API Apps in Azure App Service][apia-user]（Azure 应用服务中 API 应用的用户身份验证）
- [Service principal authentication for API Apps in Azure App Service][apia-service]（Azure 应用服务中 API 应用的服务主体身份验证）









[apia-user]: /documentation/articles/app-service-api-dotnet-user-principal-auth/
[apia-service]: /documentation/articles/app-service-api-dotnet-service-principal-auth/

[web-getstarted]: /documentation/articles/app-service-web-get-started-2/#authenticate-your-users

[iOS]: /documentation/articles/app-service-mobile-ios-get-started-users/
[Android]: /documentation/articles/app-service-mobile-android-get-started-users/
[Xamarin.iOS]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-users/
[Xamarin.Android]: /documentation/articles/app-service-mobile-xamarin-android-get-started-users/
[Xamarin.Forms]: /documentation/articles/app-service-mobile-xamarin-forms-get-started-users/
[Windows]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-users/
[Cordova]: /documentation/articles/app-service-mobile-cordova-get-started-users/

[AAD]: /documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/
[Facebook]: /documentation/articles/app-service-mobile-how-to-configure-facebook-authentication/
[Google]: /documentation/articles/app-service-mobile-how-to-configure-google-authentication/
[MSA]: /documentation/articles/app-service-mobile-how-to-configure-microsoft-authentication/
[Twitter]: /documentation/articles/app-service-mobile-how-to-configure-twitter-authentication/

[custom-auth]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#custom-auth

[ADAL-Android]: /documentation/articles/app-service-mobile-android-how-to-use-client-library/#adal
[ADAL-iOS]: /documentation/articles/app-service-mobile-ios-how-to-use-client-library/#adal
[ADAL-dotnet]: /documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/#adal

<!---HONumber=Mooncake_0919_2016-->