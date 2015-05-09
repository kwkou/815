<properties pageTitle="Azure Active Directory 代码示例" description="Azure Active Directory 代码示例的索引，按方案进行了组织。" services="active-directory" documentationCenter="dev-center-name" authors="msmbaldwin" manager="mbaldwin" editor=""/>
   
<tags ms.service="active-directory" ms.date="02/20/2015" wacn.date="04/11/2015"/>


# Azure Active Directory 代码示例 

你可以使用 Microsoft Azure Active Directory (Azure AD) 向你的 web 应用程序和 web API 添加身份验证和授权。本部分提供了指向代码示例的链接，这些代码示例将向你展示它的工作原理以及你可以在你的应用程序中使用的代码片段。在代码示例页上，你可以找到在要求、安装和设置方面提供了帮助的详细自述主题。并且代码带有注释，可以帮助你理解关键部分。

若要了解每个示例类型的基本方案，请参阅"Azure AD 的身份验证方案"。

为我们在 GitHub 上的示例做贡献：[Microsoft Azure Active Directory 示例和文档](https://github.com/AzureADSamples)。

## Web 浏览器到 Web 应用程序 

这些示例展示了如何编写 web 应用程序来对用户的浏览器进行定向以使用户登录到 Azure AD。



| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-OpenIDConnect-DotNet](http://github.com/AzureADSamples/WebApp-OpenIDConnect-DotNet) | 使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从一个 Azure AD 租户对用户进行身份验证。
| C#/.NET | [WebApp-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet) | 一个多租户 .NET MVC web 应用程序，使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从多个 Azure AD 租户对用户进行身份验证。
| C#/.NET | [WebApp-WSFederation-DotNet](https://github.com/AzureADSamples/WebApp-WSFederation-DotNet) | 使用 WS-Federation（ASP.Net WS-Federation OWIN 中间件）从一个 Azure AD 租户对用户进行身份验证。

## 单页面应用程序 (SPA)

此示例展示了如何编写受 Azure AD 保护的单页面应用程序。  

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| JavaScript, C#/.NET | [SinglePageApp-DotNet](https://github.com/AzureADSamples/SinglePageApp-DotNet) | 使用适用于 JavaScript 的 ADAL 和 Azure AD 保护基于 AngularJS 且实施了 ASP.NET web API 后端的单页面应用程序。


## 本机应用程序到 Web API
 
这些代码示例展示了如何构建本机应用程序来调用受 Azure AD 保护的 web API。它们使用 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 和 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/zh-CN/library/azure/dn645545.aspx)。
 
| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [NativeClient-DotNet](http://github.com/AzureADSamples/NativeClient-DotNet) | 一个 .NET WPF 应用程序，它调用了使用 Azure AD 保护的一个 web API。
| C#/.NET | [NativeClient-WindowsStore](http://github.com/AzureADSamples/NativeClient-WindowsStore) | 一个 Windows 应用商店应用程序，它调用了由 Azure AD 保护的一个 web API。
| C#/.NET | [NativeClient-WebAPI-MultiTenant-WindowsStore](https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore) | 一个 Windows 应用商店应用程序，它调用了由 Azure AD 保护的一个多租户 web API。
| C#/.NET | [WebAPI-OnBehalfOf-DotNet](http://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet) | 一个本机客户端应用程序，它调用了一个 web API，该 web API 代表原始用户获取一个令牌，然后使用该令牌调用另一个 web API。
| C#/.NET | [NativeClient-WindowsPhone8.1](https://github.com/AzureADSamples/NativeClient-WindowsPhone8.1) | 一个适用于 Windows Phone 8.1 的 Windows 应用商店应用程序，它调用了由 Azure AD 保护的一个 web API。
| ObjC | [NativeClient-iOS](http://github.com/AzureADSamples/NativeClient-iOS) | 一个 iOS 应用程序，它调用了要求 Azure AD 进行身份验证的一个 web API。
| C#/.NET | [WebAPI-ManuallyValidateJwt-DotNet](https://github.com/AzureADSamples/WebAPI-ManuallyValidateJwt-DotNet) | 一个本机客户端应用程序，其中包含了用来在 web API 中（而非使用 OWIN 中间件）处理 JWT 令牌的逻辑。
| C#/Xamarin | [NativeClient-Xamarin-Android](https://github.com/AzureADSamples/NativeClient-Xamarin-Android) | 到适用于 Android 库的本机 Azure AD 身份验证库 (ADAL) 的一个 Xamarin 绑定。
| C#/Xamarin | [NativeClient-Xamarin-iOS](http://github.com/AzureADSamples/NativeClient-Xamarin-iOS) | 到适用于 iOS 库的本机 Azure AD 身份验证库 (ADAL) 的一个 Xamarin 绑定。
| C#/Xamarin | [NativeClient-MultiTarget-DotNet](http://github.com/AzureADSamples/NativeClient-MultiTarget-DotNet) | 一个 Xamarin 项目，它以五个平台为目标并且调用了由 Azure AD 保护的一个 web API。
| C#/.NET | [NativeClient-Headless-DotNet](http://github.com/AzureADSamples/NativeClient-Headless-DotNet) | 一个本机应用程序，它执行非交互身份验证并调用了一个由 Azure AD 保护的 web API。

   

## Web 应用程序到 Web API

这些代码示例展示了如何使用 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/zh-CN/library/azure/dn645545.aspx) 构建 web 应用程序来调用由 Azure AD 保护的 web API。

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-WebAPI-OpenIDConnect-DotNet](http://github.com/AzureADSamples/WebApp-WebAPI-OpenIDConnect-DotNet) | 使用已注册用户的权限调用一个 web API。
|  C#/.NET | [WebApp-WebAPI-OAuth2-AppIdentity-DotNet](http://github.com/AzureADSamples/WebApp-WebAPI-OAuth2-AppIdentity-DotNet) | 使用应用程序的权限调用一个 web API。
| C#/.NET | [WebApp-WebAPI-OAuth2-UserIdentity-Dotnet](http://github.com/AzureADSamples/WebApp-WebAPI-OAuth2-UserIdentity-Dotnet) | 使用 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/zh-CN/library/azure/dn645545.aspx) 向现有 web 应用程序添加授权以便它能够更调用 web API。
| JavaScript | [WebAPI-Nodejs](http://github.com/AzureADSamples/WebAPI-Nodejs) | 设置一个与 Azure AD 集成的 REST API 服务以提供 API 保护。使用 Web API 包括一个 Node.js 服务器。
| C#/.NET | [WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet) | 一个多租户 MVC web 应用程序，使用 OpenID Connect（ASP.Net OpenID Connect OWIN 中间件）从一个 Azure AD 租户对用户进行身份验证。使用授权代码来调用 Graph API。

## 服务器或守护程序应用程序到 Web API

这些代码示例展示了如何构建守护程序或服务器应用程序来使用 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 和 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/zh-CN/library/azure/dn645545.aspx)通过一个 web API 获取资源。

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [Daemon-DotNet](http://github.com/AzureADSamples/Daemon-DotNet) | 一个控制台应用程序，它调用了一个 web API。客户端凭据是一个密码。
| C#/.NET | [Daemon-CertificateCredential-DotNet](http://github.com/AzureADSamples/Daemon-CertificateCredential-DotNet) | 一个控制台应用程序，它调用了一个 web API。客户端凭据是一个证书。


## 调用 Azure AD Graph API

这些代码示例展示了如何构建应用程序来调用 Azure AD Graph API 以读取和写入目录数据。

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| Java | [WebApp-GraphAPI-Java](http://github.com/AzureADSamples/WebApp-GraphAPI-Java) | 一个 web 应用程序，它使用 Graph API 访问 Azure AD 目录数据。
| PHP | [WebApp-GraphAPI-PHP](http://github.com/AzureADSamples/WebApp-GraphAPI-PHP) | 一个 web 应用程序，它使用 Graph API 访问 Azure AD 目录数据。
| C#/.NET | [WebApp-GraphAPI-DotNet](http://github.com/AzureADSamples/WebApp-GraphAPI-DotNet) | 一个 web 应用程序，它使用 Graph API 访问 Azure AD 目录数据。
| C#/.NET | [ConsoleApp-GraphAPI-DotNet](https://github.com/AzureADSamples/ConsoleApp-GraphAPI-DotNet) | 这是一个控制台应用程序，它展示了对 Graph API 的常用读取和写入调用，并展示了如何执行用户许可证分配以及更新用户的缩略图照片和链接。
| C#/.NET | [ConsoleApp-GraphAPI-DiffQuery-DotNet](https://github.com/AzureADSamples/ConsoleApp-GraphAPI-DiffQuery-DotNet) | 一个控制台应用程序，它使用 Graph API 中的差异查询来获取对 Azure AD 中的用户对象的定期更改。
| C#/.NET | [WebApp-GraphAPI-DirectoryExtensions-DotNet](https://github.com/AzureADSamples/WebApp-GraphAPI-DirectoryExtensions-DotNet) | 一个 MVC 应用程序，它使用 Graph API 查询生成简单的公司组织图。
| PHP | [WebApp-GraphAPI-DirectoryExtensions-PHP](https://github.com/AzureADSamples/WebApp-GraphAPI-DirectoryExtensions-PHP) | 一个 PHP 应用程序，它调用 Graph API 来注册一个扩展，然后读取、更新和删除扩展属性中的值。


## 授权

这些代码示例展示了如何使用 Azure AD 进行授权。

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [WebApp-GroupClaims-DotNet](https://github.com/AzureADSamples/WebApp-GroupClaims-DotNet) | 在与 Azure AD 集成的应用程序中执行使用 Azure Active Directory 组声明的基于角色的访问控制 (RBAC)。
| C#/.NET | [WebApp-RoleClaims-DotNet](https://github.com/AzureADSamples/WebApp-RoleClaims-DotNet) | 在与 Azure AD 集成的应用程序中执行使用 Azure Active Directory 应用程序角色的基于角色的访问控制 (RBAC)。


## 旧时演练

这些演练使用稍旧的技术，但可能仍会受关注。

| 语言/平台 | 示例 | 说明
| ----------------- | ------ | -----------
| C#/.NET | [Windows Azure AD 应用程序中基于角色的和基于 ACL 的授权](http://code.msdn.microsoft.com/Role-Based-and-ACL-Based-86ad71a1) | 在与 Azure AD 集成的应用程序中执行基于角色的授权 (RBAC) 和基于 ACL 的授权。
| C#/.NET |  [AAL - Windows 应用商店应用程序到 REST 服务 - 身份验证](http://code.msdn.microsoft.com/windowsapps/AAL-Windows-Store-app-to-2430e331) | 使用适用于 Windows 应用商店 Beta 版的 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266)（以前的 AAL） 向 Windows 应用商店应用程序添加用户身份验证功能。
| C#/.NET | [ADAL - 本机应用程序到 REST 服务 - 通过浏览器对话框使用 AAD 进行身份验证](http://code.msdn.microsoft.com/AAL-Native-Application-to-fd648dcf) | 使用 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 向 WPF 客户端添加用户身份验证功能。
| C#/.NET | [ADAL - 本机应用程序到 REST 服务 - 通过浏览器对话框使用 ACS 进行身份验证](http://code.msdn.microsoft.com/AAL-Native-App-to-REST-de57f2cc) | 使用 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 和[访问控制服务 2.0 (ACS)](https://msdn.microsoft.com/zh-CN/library/azure/hh147631.aspx) 向 WPF 客户端添加用户身份验证功能。
| C#/.NET | [ADAL - 服务器到服务器身份验证](http://code.msdn.microsoft.com/AAL-Server-to-Server-9aafccc1) | 使用 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 保护从服务器端进程到 MVC4 Web API REST 服务的服务调用。
| C#/.NET | [使用 Azure AD 向应用程序添加登录功能](https://msdn.microsoft.com/zh-CN/library/azure/dn151790.aspx) | 将 .NET 应用程序配置为根据你的 Azure AD 企业目录执行 web 单一登录。
| C#/.NET | [使用 Azure AD 开发多租户 Web 应用程序](https://msdn.microsoft.com/zh-CN/library/azure/dn151789.aspx) | 使用 Azure AD 向一个 .NET 应用程序添加单一登录和目录访问功能以便在多个组织中工作。
JAVA | [使用了 Azure AD Graph API 的 Java 示例应用程序](http://code.msdn.microsoft.com/Java-Sample-App-for-30d36d54) | 使用 Graph API 访问 Azure AD 中的目录数据。
PHP | [使用了 Azure AD Graph API 的 PHP 示例应用程序](http://code.msdn.microsoft.com/PHP-Sample-App-For-Windows-228c6ddb) | 使用 Graph API 访问 Azure AD 中的目录数据。
| C#/.NET | [使用了 Azure AD Graph API 的示例应用程序](http://code.msdn.microsoft.com/Write-Sample-App-for-79e55502) | 使用 Graph API 访问 Azure AD 中的目录数据。
| C#/.NET | [使用了 Azure AD Graph 差异查询的示例应用程序](http://code.msdn.microsoft.com/Sample-App-for-Windows-97eaec90) | 使用 Graph API 中的差异查询获取对 Azure AD 中的用户对象的定期更改。
| C#/.NET | [用于为 Azure AD 集成多租户云应用程序的示例应用程序](http://code.msdn.microsoft.com/Multi-Tenant-Cloud-8015b84b) | 将一个多租户应用程序集成到 Azure AD 中。
| C#/.NET | [使用 Azure AD（预览版）保护 Windows 应用商店应用程序和 REST Web 服务](https://msdn.microsoft.com/zh-CN/library/azure/dn169448.aspx) | 创建一个使用 Azure AD 和 [Azure AD 身份验证库 (ADAL)](https://msdn.microsoft.com/zh-CN/library/jj573266) 的简单 web API 资源和 Windows 应用商店客户端应用程序。
| C#/.NET| [使用 Graph API 查询 Azure AD](https://msdn.microsoft.com/zh-CN/library/azure/dn151791.aspx) | 将 Microsoft .NET 应用程序配置为使用 Azure AD Graph API 访问 Azure AD 租户目录中的数据。

## 另请参阅

##### 其他资源


[Azure AD Graph API 帮助程序库](http://code.msdn.microsoft.com/Windows-Azure-AD-Graph-API-a8c72e18) 

[开发使用 OAuth 和 Active Directory 联合身份验证服务的现代应用程序](https://msdn.microsoft.com/zh-CN/library/dn633593.aspx)





<!--HONumber=51-->