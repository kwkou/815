<properties title="Getting Started with Active Directory Authentication" pageTitle="" metaKeywords="Azure, Getting Started, Active Directory" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

### 发生了什么情况？

引用已添加到您的项目

###### NuGet 包引用

Microsoft.Owin, Microsoft.Owin.Host.SystemWeb, Microsoft.Owin.Security, Microsoft.Owin.Security.ActiveDirectory, Microsoft.Owin.Security.Jwt, Microsoft.Owin.Security.OAuth, Newtonsoft.Json, Owin, System.IdentityModel.Tokens.Jwt

###### .NET 引用

Microsoft.Owin, Microsoft.Owin.Host.SystemWeb, Microsoft.Owin.Security, Microsoft.Owin.Security.ActiveDirectory, Microsoft.Owin.Security.Jwt, Microsoft.Owin.Security.OAuth, Newtonsoft.Json, Owin, System.IdentityModel.Tokens.Jwt

###### 代码文件已添加到您的项目

身份验证启动类 App\_Start/Startup.Auth.cs（包含 Azure AD 身份验证的启动逻辑）已添加到您的项目。

###### 启动代码已添加到您的项目

如果您的项目中已经有一个 Startup 类，Configuration() 方法将进行更新，以包括对添加到该方法的 ConfigureAuth(app) 的调用。否则，Startup 类已添加到您的项目。

###### 您的 app.config 或 web.config 文件具有新的配置值。

已添加以下配置条目。

      <appSettings>
              <add key="ida:ClientId" value="ClientId from the new Azure AD App" />
              <add key="ida:Tenant" value="Your selected Azure AD Tenant" />
              <add key="ida:Audience" value="The App ID Uri from the wizard" />
      </appSettings> 

</p>
### 已创建 Azure Active Directory (AD) 应用

已在您在向导中选定的目录内创建一个 Azure AD 应用程序。

### Azure Active Directory (AD) 入门

下面是您可以对已添加的代码执行的操作。

###### 访问控制器需要身份验证

您项目中的所有控制器均带有 Authorize 属性。此属性要求用户先进行身份验证，然后才能访问由这些控制器定义的 API。若要允许匿名访问控制器，请从控制器删除此属性。如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。

