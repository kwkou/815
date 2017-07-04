<properties
    pageTitle="Azure Active Directory 代码示例 | Azure"
    description="Azure Active Directory 代码示例的索引，按方案组织。"
    services="active-directory"
    documentationcenter="dev-center-name"
    author="priyamohanram"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="a242a5ff-7300-40c2-ba83-fb6035707433"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/08/2017"
    wacn.date="03/13/2017"
    ms.author="mbaldwin" />

# Azure Active Directory 代码示例
[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

可以使用 Azure Active Directory (Azure AD) 向 Web 应用程序和 Web API 添加身份验证与授权。本部分提供指向示例的链接，这些示例展示了它的工作原理以及可以在应用程序中使用的代码片段。在代码示例页上，可以找到在要求、安装和设置方面提供帮助的详细自述主题。并且代码带有注释，可以帮助你理解关键部分。

若要了解每个示例类型的基本方案，请参阅“Azure AD 的身份验证方案”。

为 GitHub 上的示例做出补充：[Azure Active Directory 示例和文档](https://github.com/Azure-Samples?page=3&query=active-directory)。

## Web 浏览器到 Web 应用程序 <a name="web-browser-to-web-application"></a>

这些示例展示了如何编写 web 应用程序来对用户的浏览器进行定向以使用户登录到 Azure AD。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| C#/.NET |[WebApp-OpenIDConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect) |使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从一个 Azure AD 租户对用户进行身份验证。 |
| C#/.NET |[WebApp-MultiTenant-OpenIdConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect) |一个多租户 .NET MVC web 应用程序，使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从多个 Azure AD 租户对用户进行身份验证。 |
| C#/.NET |[WebApp-WSFederation-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) |使用 WS-Federation（ASP.Net WS-Federation OWIN 中间件）从一个 Azure AD 租户对用户进行身份验证。 |

## 单页面应用程序 (SPA) <a name="single-page-application-spa"></a>

此示例展示了如何编写受 Azure AD 保护的单页面应用程序。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| JavaScript、C#/.NET |[SinglePageApp-DotNet](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) |使用适用于 JavaScript 的 ADAL 和 Azure AD 保护使用 ASP.NET Web API 后端实施的基于 AngularJS 的单页面应用。 |

## 本机应用程序到 Web API <a name="native-application-to-web-api"></a>

这些代码示例展示了如何构建本机应用程序来调用受 Azure AD 保护的 web API。它们使用 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/) 和 [Azure AD 中的 OAuth 2.0](/documentation/articles/active-directory-protocols-oauth-code/)。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| Javascript |[NativeClient-MultiTarget-Cordova](https://github.com/Azure-Samples/active-directory-cordova-multitarget) |使用 Apache Cordova 的 ADAL 插件构建一个可调用 Web API 并使用 Azure AD 进行身份验证的 Apache Cordova 应用。 |
| C#/.NET |[NativeClient-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-desktop) |一个 .NET WPF 应用程序，它调用了使用 Azure AD 保护的一个 Web API。 |
| C#/.NET |[NativeClient-WindowsStore](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) |一个 Windows 应用商店应用程序，它调用了使用 Azure AD 保护的一个 Web API。 |
| C#/.NET |[NativeClient-WebAPI-MultiTenant-WindowsStore](https://github.com/Azure-Samples/active-directory-dotnet-webapi-multitenant-windows-store) |一个 Windows 应用商店应用程序，它调用了使用 Azure AD 保护的多租户 Web API。 |
| C#/.NET |[WebAPI-OnBehalfOf-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof) |一个本机客户端应用程序，它调用了一个 web API，该 Web API 代表原始用户获取一个令牌，然后使用该令牌调用另一个 Web API。 |
| C#/.NET |[NativeClient-WindowsPhone8.1](https://github.com/Azure-Samples/active-directory-dotnet-windowsphone-8.1) |一个适用于 Windows Phone 8.1 的 Windows 应用商店应用程序，它调用了由 Azure AD 保护的一个 Web API。 |
| ObjC |[NativeClient-iOS](https://github.com/Azure-Samples/active-directory-ios) |一个 iOS 应用程序，它调用了要求 Azure AD 进行身份验证的一个 Web API。 |
| C#/.NET |[WebAPI-ManuallyValidateJwt-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapi-manual-jwt-validation) |一个本机客户端应用程序，其中包含了用来在 Web API 中（而非使用 OWIN 中间件）处理 JWT 令牌的逻辑。 |
| C#/Xamarin |[NativeClient-Xamarin-Android](https://github.com/Azure-Samples/active-directory-xamarin-android) |到适用于 Android 库的本机 Azure AD 身份验证库 (ADAL) 的一个 Xamarin 绑定。 |
| C#/Xamarin |[NativeClient-Xamarin-iOS](https://github.com/Azure-Samples/active-directory-xamarin-ios) |到适用于 iOS 的本机 Azure AD 身份验证库 (ADAL) 的一个 Xamarin 绑定。 |
| C#/Xamarin |[NativeClient-MultiTarget-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-multitarget) |一个 Xamarin 项目，它面向五个平台并且调用由 Azure AD 保护的 Web API。 |
| C#/.NET |[NativeClient-Headless-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-native-headless) |一个本机应用程序，它执行非交互身份验证并调用由 Azure AD 保护的 Web API。 |

## Web 应用程序到 Web API <a name="web-application-to-web-api"></a>

这些代码示例展示了如何使用 [Azure AD 中的 OAuth 2.0](/documentation/articles/active-directory-protocols-oauth-code/) 构建 web 应用程序来调用由 Azure AD 保护的 Web API。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| C#/.NET |[WebApp-WebAPI-OpenIDConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-openidconnect) |使用已注册用户的权限调用 Web API。 |
| C#/.NET |[WebApp-WebAPI-OAuth2-AppIdentity-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity) |使用应用程序的权限调用 Web API。 |
| C#/.NET |[WebApp-WebAPI-OAuth2-UserIdentity-Dotnet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-useridentity) |使用 [Azure AD 中的 OAuth 2.0](/documentation/articles/active-directory-protocols-oauth-code/) 向现有 Web 应用程序添加授权以便它能够更调用 Web API。 |
| JavaScript |[WebAPI-Nodejs](https://github.com/Azure-Samples/active-directory-node-webapi) |设置一个与 Azure AD 集成的 REST API 服务以提供 API 保护。在 Web API 中包含 Node.js 服务器。 |
| C#/.NET |[WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-multitenant-openidconnect) |一个多租户 MVC Web 应用程序，使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从 Azure AD 租户对用户进行身份验证。使用授权代码来调用图形 API。 |

## 服务器或守护程序应用程序到 Web API <a name="server-or-daemon-application-to-web-api"></a>

这些代码示例展示了如何构建守护程序或服务器应用程序来使用 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/) 和 [Azure AD 中的 OAuth 2.0](/documentation/articles/active-directory-protocols-oauth-code/) 通过一个 Web API 获取资源。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| C#/.NET |[Daemon-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-daemon) |控制台应用程序调用 Web API。客户端凭据是密码。 |
| C#/.NET |[Daemon-CertificateCredential-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential) |调用了 Web API 的一个控制台应用程序。客户端凭据是证书。 |

## 调用 Azure AD 图形 API
这些代码示例展示了如何构建应用程序来调用 Azure AD 图形 API 以读取和写入目录数据。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| Java |[WebApp-GraphAPI-Java](https://github.com/Azure-Samples/active-directory-java-graphapi-web) |一个 Web 应用程序，它使用图形 API 访问 Azure AD 目录数据。 |
| PHP |[WebApp-GraphAPI-PHP](https://github.com/Azure-Samples/active-directory-php-graphapi-web) |一个 Web 应用程序，它使用图形 API 访问 Azure AD 目录数据。 |
| C#/.NET |[WebApp-GraphAPI-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-web) |一个 Web 应用程序，它使用图形 API 访问 Azure AD 目录数据。 |
| C#/.NET |[ConsoleApp-GraphAPI-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-console) |这是一个控制台应用程序，它展示了对图形 API 的常用读取和写入调用，并展示了如何执行用户许可证分配以及更新用户的缩略图照片和链接。 |
| C#/.NET |[ConsoleApp-GraphAPI-DiffQuery-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-diffquery) |一个控制台应用程序，它使用图形 API 中的差异查询来获取对 Azure AD 中的用户对象的定期更改。 |
| C#/.NET |[WebApp-GraphAPI-DirectoryExtensions-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-directoryextensions-web) |一个 MVC 应用程序，它使用图形 API 查询生成简单的公司组织图。 |
| PHP |[WebApp-GraphAPI-DirectoryExtensions-PHP](https://github.com/Azure-Samples/active-directory-php-graphapi-directoryextensions-web) |一个 PHP 应用程序，它调用图形 API 来注册扩展，然后读取、更新和删除扩展属性中的值。 |

## 授权
这些代码示例展示了如何使用 Azure AD 进行授权。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| C#/.NET |[WebApp-GroupClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-groupclaims) |在与 Azure AD 集成的应用程序中执行使用 Azure Active Directory 组声明的基于角色的访问控制 (RBAC)。 |
| C#/.NET |[WebApp-RoleClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) |在与 Azure AD 集成的应用程序中执行使用 Azure Active Directory 应用程序角色的基于角色的访问控制 (RBAC)。 |

## 旧时演练
这些演练使用稍旧的技术，但可能仍值得关注。

| 语言/平台 | 示例 | 说明 |
| --- | --- | --- |
| C#/.NET |[Azure AD 应用程序中基于角色和基于 ACL 的授权](http://go.microsoft.com/fwlink/?LinkId=331694) |在与 Azure AD 集成的应用程序中执行基于角色的授权 (RBAC) 和基于 ACL 的授权。 |
| C#/.NET |[AAL - Windows 应用商店应用到 REST 服务 - 身份验证](http://go.microsoft.com/fwlink/?LinkId=330605) |使用适用于 Windows 应用商店 Beta 版的 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/)（以前称为 AAL）向 Windows 应用商店应用添加用户身份验证功能。 |
| C#/.NET |[ADAL - 本机应用到 REST 服务 - 通过浏览器对话框使用 AAD 进行身份验证](http://go.microsoft.com/fwlink/?LinkId=259814) |使用 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/) 向 WPF 客户端添加用户身份验证功能。 |
| C#/.NET | |使用 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/) |
| C#/.NET |[ADAL - 服务器到服务器的身份验证](http://go.microsoft.com/fwlink/?LinkId=259816) |使用 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/) 保护从服务器端进程到 MVC4 Web API REST 服务的服务调用。 |
| C#/.NET |[使用 Azure AD 将登录名添加到 Web 应用程序](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect) |将 .NET 应用程序配置为根据 Azure AD 企业目录执行 Web 单一登录。 |
| C#/.NET |[利用 Azure AD 开发多租户 Web 应用程序](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect) |使用 Azure AD 添加一个 .NET 应用程序的单一登录和目录访问功能，以便在多个组织中工作。 |
| JAVA |[Azure AD 图形 API 的 Java 示例应用](http://go.microsoft.com/fwlink/?LinkId=263969) |使用图形 API 访问 Azure AD 中的目录数据。 |
| PHP |[Azure AD 图形 API 的 PHP 示例应用](http://code.msdn.microsoft.com/PHP-Sample-App-For-Windows-228c6ddb) |使用图形 API 访问 Azure AD 中的目录数据。 |
| C#/.NET |[Azure AD 图形 API 的示例应用](http://go.microsoft.com/fwlink/?LinkID=262648) |使用图形 API 访问 Azure AD 中的目录数据。 |
| C#/.NET |[Azure AD Graph 差异查询的示例应用](http://go.microsoft.com/fwlink/?LinkId=275398) |使用图形 API 中的差异查询来获取对 Azure AD 租户中用户对象的定期更改。 |
| C#/.NET |[用于集成 Azure AD 的多租户云应用程序的示例应用](http://go.microsoft.com/fwlink/?LinkId=275397) |将多租户应用程序集成到 Azure AD 中。 |
| C#/.NET |[使用 Azure AD 保护 Windows 应用商店应用程序和 REST Web 服务](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) |使用 Azure AD 和 [Azure AD 身份验证库 (ADAL)](/documentation/articles/active-directory-authentication-libraries/)，创建简单的 Web API 资源和 Windows 应用商店客户端应用程序。 |
| C#/.NET |[使用图形 API 查询 Azure AD](https://github.com/Azure-Samples/active-directory-dotnet-graphapi-web) |将 Microsoft .NET 应用程序配置为使用 Azure AD 图形 API 访问 Azure AD 租户目录中的数据。 |

## 另请参阅
##### 其他资源
[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)

[Azure AD 图形 API 概念和参考](https://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx)

[Azure AD 图形 API 帮助程序库](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->