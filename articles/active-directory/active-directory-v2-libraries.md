<properties
    pageTitle="Azure Active Directory v2.0 身份验证库 | Azure"
    description="Azure Active Directory v2.0 终结点的兼容客户端库和服务器中间件库，以及相关的库、源代码和示例链接。"
    services="active-directory"
    documentationcenter=""
    author="skwan"
    manager="mbaldwin"
    editor="" />  

<tags
    ms.assetid="19cec615-e51f-4141-9f8c-aaf38ff9f746"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="11/15/2016"
    ms.author="skwan;bryanla" 
    wacn.date="12/12/2016"/>  


# Azure Active Directory v2.0 身份验证库
Azure Active Directory (Azure AD) v2.0 终结点支持行业标准 OAuth 2.0 和 OpenID Connect 1.0 协议。可以对 v2.0 终结点使用 Microsoft 和其他组织提供的各种库。

在构建使用 v2.0 终结点的应用程序时，建议使用协议领域的专家根据安全开发生命周期 (SDL) 方法（[例如 Microsoft 遵循的方法][Microsoft-SDL]）编写的库。如果决定手动编写协议支持，建议遵循 SDL 方法并认真对待每个协议的标准规范中的安全注意事项。

## 库的类型
Azure AD v2.0 适用于两类库：

- **客户端库**。本机客户端和服务器使用客户端库获取用于调用某个资源（例如 Microsoft Graph）的访问令牌。
- **服务器中间件库**。Web 应用使用服务器中间件库进行用户登录。Web API 使用服务器中间件库验证本机客户端或其他服务器发送的令牌。

## 库支持
可以在使用 v2.0 终结点时选择任何符合标准的库，此时必须知道可从何处寻求支持。有关库代码中的问题和功能请求，请联系库所有者。有关服务端协议实现中的问题和功能请求，请联系 Microsoft。

库的支持类型有两种：

- **Microsoft 支持**。Microsoft 为这些库提供修补程序，并对这些库进行 SDL 相关工作。
- **兼容**。Microsoft 已在基本方案中测试这些库并确认它们适用于 v2.0 终结点。Microsoft 不提供这些库的修复程序，且尚未审查这些库。问题和功能请求应重定向到库的开源项目。

有关适用于 V2.0 终结点的库列表，请参阅本文的后续部分。

## Microsoft 支持的客户端库
| 平台 | 库名称 | 下载 | 源代码 | 示例 |
|:---:|:---:|:---:|:---:|:---:|
| .NET、Windows 应用商店、Xamarin |适用于 .NET 的 Microsoft 身份验证库 (MSAL) |[Microsoft.Identity.Client (NuGet)][ClientLib-NET-Lib] |[适用于 .NET 的 MSAL (GitHub)][ClientLib-NET-Repo] |[Windows 桌面本机客户端示例][ClientLib-NET-Sample] |
| Node.js |Azure Active Directory Passport.js 插件 |[Passport-Azure-AD (npm)][ClientLib-Node-Lib] |[Passport-Azure-AD (GitHub)][ClientLib-Node-Repo] |即将支持 |

<!--- COMMENTING OUT UNTIL THEY ARE READY
| iOS, Mac | Microsoft Authentication Library (MSAL) for ObjC | In development | In development | In development |
| Android | Microsoft Authentication Library (MSAL) for Android | In development | In development | In development |
| JavaScript | Microsoft Authentication Library (MSAL) for JavaScript | In development | In development | In development |
 -->


## Microsoft 支持的服务器中间件库
| 平台 | 库名称 | 下载 | 源代码 | 示例 |
|:---:|:---:|:---:|:---:|:---:|
| .NET 4.x |适用于 ASP.NET 的 OWIN OpenID Connect 中间件 |[Microsoft.Owin.Security.OpenIdConnect (NuGet)][ServerLib-Net4-Owin-Oidc-Lib] |[Katana 项目 (CodePlex)][ServerLib-Net4-Owin-Oidc-Repo] |[Web 应用示例][ServerLib-Net4-Owin-Oidc-Sample] |
| .NET 4.x |适用于 ASP.NET 的 OWIN OAuth Bearer 中间件 |[Microsoft.Owin.Security.OAuth (NuGet)][ServerLib-Net4-Owin-Oauth-Lib] |[Katana 项目 (CodePlex)][ServerLib-Net4-Owin-Oauth-Repo] |[Web API 示例][ServerLib-Net4-Owin-Oauth-Sample] |
| .NET Core |适用于 .NET Core 的 OWIN OpenID Connect 中间件 |[Microsoft.AspNetCore.Authentication.OpenIdConnect (NuGet)][ServerLib-NetCore-Owin-Oidc-Lib] |[ASP.NET 安全性 (GitHub)][ServerLib-NetCore-Owin-Oidc-Repo] |[Web 应用示例][ServerLib-NetCore-Owin-Oidc-Sample] |
| .NET Core |适用于 .NET Core 的 OWIN OAuth Bearer 中间件 |[Microsoft.AspNetCore.Authentication.OAuth (NuGet)][ServerLib-NetCore-Owin-Oauth-Lib] |[ASP.NET 安全性 (GitHub)][ServerLib-NetCore-Owin-Oauth-Repo] |即将支持 |
| Node.js |Azure Active Directory Passport.js 插件 |[Passport-Azure-AD (npm)][ServerLib-Node-Lib] |[Passport-Azure-AD (GitHub)][ServerLib-Node-Repo] |[Web 应用示例][ServerLib-Node-Sample] |

<!--- COMMENTING UNTIL SAMPLE IS AVAILABLE
| .NET 4.x, .NET Core | JSON Web Token Handler for .NET | [System.IdentityModel.Tokens.Jwt (NuGet)][ServerLib-Net-Jwt-Lib] | [Azure AD identity model extensions for .NET (GitHub)][ServerLib-Net-Jwt-Repo] | Coming soon |
--->

## 兼容的客户端库
| 平台 | 库名称 | 测试的版本 | 源代码 | 示例 |
|:---:|:---:|:---:|:---:|:---:|
| Android |[OIDCAndroidLib](https://github.com/kalemontes/OIDCAndroidLib/wiki) |0\.2.1 |[OIDCAndroidLib](https://github.com/kalemontes/OIDCAndroidLib) |[本机应用示例](/documentation/articles/active-directory-v2-devquickstarts-android/) |
| iOS |[NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) |1\.2.8 |[NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) |[本机应用示例](/documentation/articles/active-directory-v2-devquickstarts-ios/) |
| Java | [Scribe Java scribejava](https://github.com/scribejava/scribejava) | [版本 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [ScribeJava](https://github.com/scribejava/scribejava/archive/scribejava-3.2.0.zip) | 即将支持 |
| JavaScript |[Hello.js](https://adodson.com/hello.js/) |1\.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |即将支持 |
| PHP | [The PHP League oauth2-client](https://github.com/thephpleague/oauth2-client) | [版本 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/archive/1.4.2.zip) | 即将支持 |
| Python-Flask |[Flask-OAuthlib](https://github.com/lepture/flask-oauthlib) |0\.9.3 |[Flask-OAuthlib](https://github.com/lepture/flask-oauthlib) |即将支持 |
| Ruby |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |omniauth:1.3.1</br>omniauth-oauth2:1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)</br>[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |即将支持 |

<!--- REMOVING BRANDON'S FOR NOW
|  |  |  |  |  |
| Android | [OAuth2 Client](https://github.com/wuman/android-oauth-client) |   | [OAuth2 Client](https://github.com/wuman/android-oauth-client)  | Coming soon  |
| Java | [WSO2 Identity Server](https://docs.wso2.com/display/IS500/Introducing+the+Identity+Server) | [Version 5.2.0](http://wso2.com/products/identity-server/) | [Source](https://docs.wso2.com/display/IS500/Building+from+Source) | [Samples index](https://docs.wso2.com/display/IS500/Samples)  |
| Java | [Java Gluu Server](https://gluu.org/docs/) |   | [oxAuth](https://github.com/GluuFederation/oxAuth)  | Coming soon |
| Node.js | [NPM passport-openidconnect](https://www.npmjs.com/package/passport-openidconnect) | 0.0.1  | [Passport-OpenID Connect](https://github.com/jaredhanson/passport-openidconnect) | Coming soon  |
| PHP | [OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP) |   | [OpenID Connect Basic Client](https://github.com/jumbojett/OpenID-Connect-PHP)  | Coming soon  |
-->


## 兼容的服务器中间件库
即将支持

## 相关内容
有关 Azure AD v2.0 终结点的详细信息，请参阅 [Azure AD 应用模型 v2.0 概述][AAD-App-Model-V2-Overview]。

阅读完本文后，请使用 Disqus 意见功能提供反馈，帮助我们改进内容。

<!--Image references-->


<!--Reference style links -->

[AAD-App-Model-V2-Overview]: /documentation/articles/active-directory-appmodel-v2-overview/
[ClientLib-NET-Lib]: http://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-NET-Sample]: /documentation/articles/active-directory-v2-devquickstarts-wpf/
[ClientLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ClientLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad
[ClientLib-Node-Sample]: [ClientLib-Iosmac-Lib]:
[ClientLib-Iosmac-Repo]: [ClientLib-Iosmac-Sample]:
[ClientLib-Android-Lib]: [ClientLib-Android-Repo]:
[ClientLib-Android-Sample]: [ClientLib-Js-Lib]:
[ClientLib-Js-Repo]: [ClientLib-Js-Sample]:

[Microsoft-SDL]: http://www.microsoft.com/sdl/default.aspx
[ServerLib-Net4-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/
[ServerLib-Net4-Owin-Oidc-Repo]: http://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oidc-Sample]: /documentation/articles/active-directory-v2-devquickstarts-dotnet-web/
[ServerLib-Net4-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth/
[ServerLib-Net4-Owin-Oauth-Repo]: http://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oauth-Sample]: /documentation/articles/active-directory-v2-devquickstarts-dotnet-api/
[ServerLib-Net-Jwt-Lib]: https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt
[ServerLib-Net-Jwt-Repo]: https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
[ServerLib-Net-Jwt-Sample]: /
[ServerLib-NetCore-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OpenIdConnect/
[ServerLib-NetCore-Owin-Oidc-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oidc-Sample]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore-v2
[ServerLib-NetCore-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OAuth/
[ServerLib-NetCore-Owin-Oauth-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oauth-Sample]: /
[ServerLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ServerLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad/
[ServerLib-Node-Sample]: /documentation/articles/active-directory-v2-devquickstarts-node-web/

<!---HONumber=Mooncake_1205_2016-->
