<properties
   pageTitle="Azure Active Directory 身份验证库 | Microsoft Azure"
   description="通过 Azure AD 身份验证库 (ADAL)，客户端应用程序开发人员能够轻松利用云或本地 Active Directory (AD) 对用户进行身份验证，然后获取访问令牌，以进行安全的 API 调用。"
   services="active-directory"
   documentationCenter=""
   authors="msmbaldwin"
   manager="mbaldwin"
   editor="mbaldwin" />
<tags
   ms.service="active-directory"
   ms.date="02/25/2016"
   wacn.date="04/28/2016" />

# Azure Active Directory 身份验证库

通过 Azure AD 身份验证库 (ADAL)，客户端应用程序开发人员能够轻松利用云或本地 Active Directory (AD) 对用户进行身份验证，然后获取访问令牌，以进行安全的 API 调用。ADAL 提供许多可以方便开发人员进行身份验证的功能，例如，异步支持、用于存储访问令牌和刷新令牌的可配置令牌缓存、访问令牌过期时和提供刷新令牌时自动刷新令牌，等等。ADAL 可以应对大部分复杂情况，因而可以帮助开发人员集中处理其应用程序中的业务逻辑，并可轻松保护资源而不必成为安全方面的专家。

可在各种平台上使用 ADAL。

|平台|包名称|客户端/服务器|下载|源代码|文档和示例|
|---|---|---|---|---|---|
|.NET 客户端、Windows 应用商店、Windows Phone(8.1)|适用于 .NET 的 Active Directory 身份验证库 (ADAL)|客户端|[Microsoft.IdentityModel.Clients.ActiveDirectory (NuGet)](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory)|[适用于 .NET 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet)|[文档](https://msdn.microsoft.com/library/azure/mt417579.aspx)|
|JavaScript|适用于 JavaScript 的 Active Directory 身份验证库 (ADAL)|客户端|[适用于 JavaScript 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-js)|[适用于 JavaScript 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-js)|示例：[SinglePageApp-DotNet (Github)](https://github.com/AzureADSamples/SinglePageApp-DotNet)|
|OS X、iOS|适用于 Objective-C 的 Active Directory 身份验证库 (ADAL)|客户端|[适用于 Objective-C 的 ADAL (CocoaPods)](https://cocoapods.org/?q=adal%20io)|[适用于 Objective-C 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-objc)|示例：[NativeClient-iOS (Github)](https://github.com/AzureADSamples/NativeClient-iOS)|
|Android|适用于 Android 的 Active Directory 身份验证库 (ADAL)|客户端|[适用于 Android 的 ADAL（中央存储库）](http://search.maven.org/remotecontent?filepath=com/microsoft/aad/adal/)|[适用于 Android 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-android)|示例：[NativeClient-Android (Github)](https://github.com/AzureADSamples/NativeClient-Android)|
|Node.js|适用于 Node.js 的 Active Directory 身份验证库 (ADAL)|客户端|[适用于 Node.js 的 ADAL (npm)](https://www.npmjs.com/package/adal-node)|[适用于 Node.js 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs)|示例：[WebAPI-Nodejs (Github)](https://github.com/AzureADSamples/WebAPI-Nodejs)|
|Node.js|适用于 Node 的 Microsoft Azure Active Directory Passport 身份验证中间件|客户端|[适用于 Node.js 的 Azure Active Directory Passport (npm)](https://www.npmjs.com/package/passport-azure-ad)|[适用于 Node.js 的 Azure Active Directory (Github)](https://github.com/AzureAD/passport-azure-ad)||
|Java|适用于 Java 的 Active Directory 身份验证库 (ADAL)|客户端|[适用于 Java 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-java)|[适用于 Java 的 ADAL (Github)](https://github.com/AzureAD/azure-activedirectory-library-for-java)||
|.NET|适用于 Microsoft.NET Framework 4.5 的标识协议扩展|服务器|[Microsoft.IdentityModel.Protocol.Extensions (NuGet)](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocol.Extensions)|[适用于 .NET 的 Azure AD 标识模型扩展 (Github)](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet)||
|.NET|适用于 Microsoft.Net Framework 4.5 的 JSON Web 令牌处理程序|服务器|[System.IdentityModel.Tokens.Jwt (NuGet)](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt)|[适用于 .NET 的 Azure AD 标识模型扩展 (Github)](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet)||
|.NET|可让应用程序使用 Microsoft 技术进行身份验证的 OWIN 中间件。|服务器|[Microsoft.Owin.Security.ActiveDirectory (NuGet)](https://www.nuget.org/packages/Microsoft.Owin.Security.ActiveDirectory/)|[OWIN (CodePlex)](http://katanaproject.codeplex.com)||
|.NET|可让应用程序使用 OpenIDConnect 进行身份验证的 OWIN 中间件。|服务器|[Microsoft.Owin.Security.OpenIdConnect (NuGet)](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect)|[OWIN (CodePlex)](http://katanaproject.codeplex.com)|示例：[WebApp-OpenIDConnecty-DotNet (Github)](https://github.com/AzureADSamples/WebApp-OpenIDConnect-DotNet)|
|.NET|可让应用程序使用 WS-Federation 进行身份验证的 OWIN 中间件。|服务器|[Microsoft.Owin.Security.WsFederation (NuGet)](https://www.nuget.org/packages/Microsoft.Owin.Security.WsFederation)|[OWIN (CodePlex)](http://katanaproject.codeplex.com)|示例：[WebApp-WSFederation-DotNet (Github)](https://github.com/AzureADSamples/WebApp-WSFederation-DotNet)|

## 方案

以下是可以使用 ADAL 进行身份验证的三种常见方案。

### 使用远程资源对客户端应用程序的用户进行身份验证

在此方案中，开发人员有一个客户端（如 WPF 应用程序）需要访问由 Azure AD 保护的远程资源（如 Web API）。他有一个 Azure 订阅，知道如何调用下游 Web API，还知道 Web API 使用哪个 Azure AD 租户。因此，他可以通过将身份验证体验完全委托给 ADAL 或通过显式处理用户凭据，来使用 ADAL 实现通过 Azure AD 进行身份验证。使用 ADAL 可以轻松地对用户进行身份验证，从 Azure AD 获取访问令牌并刷新该令牌，然后使用访问令牌向 Web API 发出请求。

有关使用 Azure AD 身份验证演示此方案的代码示例，请参阅[本机客户端 WPF 应用程序到 Web API](https://github.com/azureadsamples/nativeclient-dotnet)。

### 使用远程资源对服务器应用程序进行身份验证

在此方案中，开发人员在服务器上有一个正在运行的应用程序需要访问由 Azure AD 保护的远程资源（如 Web API）。他有一个 Azure 订阅，知道如何调用下游服务，还知道 Web API 使用哪个 Azure AD 租户。因此，他可以通过显式处理应用程序的凭据，来使用 ADAL 实现通过 Azure AD 进行身份验证。使用 ADAL 可以轻松地使用应用程序的客户端凭据从 Azure AD 中检索令牌，然后使用该令牌向 Web API 发出请求。ADAL 还通过缓存访问令牌并在必要时续订它，来处理对访问令牌生存期的管理。有关演示此方案的代码示例，请参阅[控制台应用程序到 Web API](https://github.com/AzureADSamples/Daemon-DotNet)。

### 对代表用户访问远程资源的服务器应用程序进行身份验证

在此方案中，开发人员在服务器上有一个正在运行的应用程序需要访问由 Azure AD 保护的远程资源（如 Web API）。请求也需要代表 Azure AD 中的用户发出。他有一个 Azure 订阅，知道如何调用下游 Web API，还知道服务使用哪个 Azure AD 租户。用户通过 Web 应用的身份验证后，应用程序可以从 Azure AD 获取该用户的授权代码。然后， Web 应用可以使用该授权代码以及与应用程序关联的客户端凭据，代表用户通过 ADAL 从 Azure AD 中获取访问令牌和刷新令牌。 Web 应用拥有访问令牌后，就可以调用 Web API，直到该令牌过期。令牌过期后， Web 应用可以使用前面收到的刷新令牌，通过 ADAL 获取新的访问令牌。


## 另请参阅

[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)

[Azure Active directory 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)

[Azure Active Directory 代码示例](/documentation/articles/active-directory-code-samples/)

<!---HONumber=Mooncake_0613_2016-->