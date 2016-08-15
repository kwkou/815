<properties
	pageTitle="我的 MVC 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况？| Azure"
	description="描述一下，当你使用 Visual Studio 连接服务连接到 Azure AD 时，你的 MVC 项目会发生什么情况"
	services="active-directory"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor=""/>
  
<tags 
	ms.service="active-directory" 
	ms.date="03/28/2016"
	wacn.date="07/05/2016"/>	

# 我的 MVC 项目（Visual Studio Azure Active Directory 连接服务）发生了什么情况？

> [AZURE.SELECTOR]
> - [入门](/documentation/articles/vs-active-directory-dotnet-getting-started/)
> - [发生了什么情况](/documentation/articles/vs-active-directory-dotnet-what-happened/)



## 已添加引用

### NuGet 包引用

- **Microsoft.IdentityModel.Protocol.Extensions**
- **Microsoft.Owin**
- **Microsoft.Owin.Host.SystemWeb**
- **Microsoft.Owin.Security**
- **Microsoft.Owin.Security.Cookies**
- **Microsoft.Owin.Security.OpenIdConnect**
- **Owin**
- **System.IdentityModel.Tokens.Jwt**

### .NET 引用

- **Microsoft.IdentityModel.Protocol.Extensions**
- **Microsoft.Owin**
- **Microsoft.Owin.Host.SystemWeb**
- **Microsoft.Owin.Security**
- **Microsoft.Owin.Security.Cookies**
- **Microsoft.Owin.Security.OpenIdConnect**
- **Owin**
- **System.IdentityModel**
- **System.IdentityModel.Tokens.Jwt**
- **System.Runtime.Serialization**

## 已添加代码

### 代码文件已添加到您的项目

身份验证启动类 **App\_Start/Startup.Auth.cs**（包含 Azure AD 身份验证的启动逻辑）已添加到您的项目。此外，还添加了控制器类 Controllers/AccountController.cs，其中包含 **SignIn()** 和 **SignOut()** 方法。最后，添加了分部视图 **Views/Shared/\_LoginPartial.cshtml**（包含 SignIn/SignOut 的操作链接）。

### 启动代码已添加到您的项目

如果你的项目中已经有一个 Startup 类，**Configuration** 方法将进行更新，以包括对 **ConfigureAuth(app)** 的调用。否则，Startup 类已添加到您的项目。

### 您的 app.config 或 web.config 具有新的配置值

已添加以下配置条目。


	<appSettings>
	    <add key="ida:ClientId" value="ClientId from the new Azure AD App" />
	    <add key="ida:AADInstance" value="https://login.microsoftonline.com/" />
	    <add key="ida:Domain" value="The selected Azure AD Domain" />
	    <add key="ida:TenantId" value="The Id of your selected Azure AD Tenant" />
	    <add key="ida:PostLogoutRedirectUri" value="Your project start page" />
	</appSettings>

### 已创建 Azure Active Directory (AD) 应用
已在您在向导中选定的目录内创建一个 Azure AD 应用程序。

##如果我选中“禁用单个用户帐户身份验证”，则会对我的项目进行哪些额外的更改？
NuGet 包引用已删除，文件已删除和备份。根据你的项目的状态，你可能需要手动删除额外的引用或文件，或者根据需要修改代码。

### 删除的 NuGet 包引用（针对已存在的）

- **Microsoft.AspNet.Identity.Core**
- **Microsoft.AspNet.Identity.EntityFramework**
- **Microsoft.AspNet.Identity.Owin**

### 备份的和删除的代码文件（针对已存在的）

以下每个文件都已备份并从项目中删除。备份文件位于项目根目录的“Backup”文件夹中。

- **App\_Start\\IdentityConfig.cs**
- **Controllers\\ManageController.cs**
- **Models\\IdentityModels.cs**
- **Models\\ManageViewModels.cs**

### 备份的的代码文件（针对已存在的）

以下每个文件在替换之前已备份。备份文件位于项目根目录的“Backup”文件夹中。

- **Startup.cs**
- **App\_Start\\Startup.Auth.cs**
- **Controllers\\AccountController.cs**
- **Views\\Shared\_LoginPartial.cshtml**

## 如果我选中“读取目录数据”，则会对我的项目做出其他哪些更改？

添加了其他引用。

###其他 NuGet 包引用

- **EntityFramework**
- **Microsoft.Azure.ActiveDirectory.GraphClient**
- **Microsoft.Data.Edm**
- **Microsoft.Data.OData**
- **Microsoft.Data.Services.Client**
- **Microsoft.IdentityModel.Clients.ActiveDirectory**
- **System.Spatial**

###其他 .NET 引用

- **EntityFramework**
- **EntityFramework.SqlServer**
- **Microsoft.Azure.ActiveDirectory.GraphClient**
- **Microsoft.Data.Edm**
- **Microsoft.Data.OData**
- **Microsoft.Data.Services.Client**
- **Microsoft.IdentityModel.Clients.ActiveDirectory**
- **Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms**
- **System.Spatial**

###其他代码文件已添加到你的项目

添加了两个支持令牌缓存的文件：**Models\\ADALTokenCache.cs** 和 **Models\\ApplicationDbContext.cs**。额外添加了一个控制器和视图，以演示如何使用 Azure 图形 API 访问用户配置文件信息。这些文件是 **Controllers\\UserProfileController.cs** 和 **Views\\UserProfile\\Index.cshtml**。

###其他启动代码已添加到你的项目

在 **startup.auth.cs** 文件中，新的 **OpenIdConnectAuthenticationNotifications** 对象已添加到 **OpenIdConnectAuthenticationOptions** 的 **Notifications** 成员中。这是为了能够接收 OAuth 代码，并使用该对象来交换访问令牌。

###对 app.config 或 web.config 做出的其他更改

添加了以下附加配置条目。

	<appSettings>
	    <add key="ida:ClientSecret" value="Your Azure AD App's new client secret" />
	</appSettings>

添加了以下配置节和连接字符串。

	<configSections>
	    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
	    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
	</configSections>
	<connectionStrings>
	    <add name="DefaultConnection" connectionString="Data Source=(localdb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-[AppName + Generated Id].mdf;Initial Catalog=aspnet-[AppName + Generated Id];Integrated Security=True" providerName="System.Data.SqlClient" />
	</connectionStrings>
	<entityFramework>
	    <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
	      <parameters>
	        <parameter value="mssqllocaldb" />
	      </parameters>
	    </defaultConnectionFactory>
	    <providers>
	      <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
	    </providers>
	</entityFramework>


###你的 Azure Active Directory 应用已更新
你的 Azure Active Directory 应用已更新为包括读取目录数据权限，并已创建一个附加密钥，该密钥随后已用作 **Web.config** 文件中的 *ida:ClientSecret*。

[详细了解 Azure Active Directory](/documentation/services/identity/)
 
<!---HONumber=Mooncake_0620_2016-->