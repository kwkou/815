<properties
	pageTitle="我的 WebApi 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况 | Azure"
	description="描述当你使用 Visual Studio 连接到 Azure AD 时，你的 MVC 项目 WebApi 会发生什么情况"
	services="active-directory"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="03/28/2016"
	wacn.date="07/05/2016"/>

# 我的 WebApi 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况

> [AZURE.SELECTOR]
> - [入门](/documentation/articles/vs-active-directory-webapi-getting-started/)
> - [发生了什么情况](/documentation/articles/vs-active-directory-webapi-what-happened/)

##已添加引用

###NuGet 包引用

- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.ActiveDirectory`
- `Microsoft.Owin.Security.Jwt`
- `Microsoft.Owin.Security.OAuth`
- `Owin`
- `System.IdentityModel.Tokens.Jwt`

###.NET 引用

- `Microsoft.Owin`
- `Microsoft.Owin.Host.SystemWeb`
- `Microsoft.Owin.Security`
- `Microsoft.Owin.Security.ActiveDirectory`
- `Microsoft.Owin.Security.Jwt`
- `Microsoft.Owin.Security.OAuth`
- `Owin`
- `System.IdentityModel.Tokens.Jwt`

##代码更改

###代码文件已添加到您的项目

身份验证启动类 **App\_Start/Startup.Auth.cs**（包含 Azure AD 身份验证的启动逻辑）已添加到您的项目。

###启动代码已添加到您的项目

如果你的项目中已经有一个 Startup 类，**Configuration** 方法将进行更新，以包括对 `ConfigureAuth(app)` 的调用。否则，Startup 类已添加到您的项目。


###您的 app.config 或 web.config 文件具有新的配置值。

已添加以下配置条目。

	<appSettings>
    		<add key="ida:ClientId" value="ClientId from the new Azure AD App" />
    		<add key="ida:Tenant" value="Your selected Azure AD Tenant" />
    		<add key="ida:Audience" value="The App ID Uri from the wizard" />
	</appSettings>


###已创建 Azure AD 应用

已在您在向导中选定的目录内创建一个 Azure AD 应用程序。

[详细了解 Azure Active Directory](/services/active-directory/)

##如果我选中“禁用单个用户帐户身份验证”，则会对我的项目进行哪些额外的更改？
NuGet 包引用已删除，文件已删除和备份。根据你的项目的状态，你可能需要手动删除额外的引用或文件，或者根据需要修改代码。

###删除的 NuGet 包引用（针对已存在的）

- `Microsoft.AspNet.Identity.Core`
- `Microsoft.AspNet.Identity.EntityFramework`
- `Microsoft.AspNet.Identity.Owin`

###备份的和删除的代码文件（针对已存在的）

以下每个文件都已备份并从项目中删除。备份文件位于项目根目录的“Backup”文件夹中。

- `App_Start\IdentityConfig.cs`
- `Controllers\AccountController.cs`
- `Controllers\ManageController.cs`
- `Models\IdentityModels.cs`
- `Providers\ApplicationOAuthProvider.cs`

###备份的的代码文件（针对已存在的）

以下每个文件在替换之前已备份。备份文件位于项目根目录的“Backup”文件夹中。

- `Startup.cs`
- `App_Start\Startup.Auth.cs`

##如果我选中“读取目录数据”，则会对我的项目做出其他哪些更改？

###对 app.config 或 web.config 做出的其他更改

添加了以下附加配置条目。


	<appSettings>
	    <add key="ida:Password" value="Your Azure AD App's new password" />
	</appSettings>


###你的 Azure Active Directory 应用已更新
你的 Azure Active Directory 应用已更新为包括“读取目录数据”权限，并已创建一个附加密钥，该密钥随后已用作 `web.config` 文件中的 *ida:Password*。

[详细了解 Azure Active Directory](/services/active-directory/)

<!---HONumber=Mooncake_0620_2016-->