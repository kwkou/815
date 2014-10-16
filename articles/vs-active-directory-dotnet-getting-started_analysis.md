<properties title="Getting Started with Active Directory Authentication" pageTitle="" metaKeywords="Azure, Getting Started, Active Directory" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

### 发生了什么情况？

引用已添加到您的项目

###### NuGet 包引用

Microsoft.IdentityModel.Protocol.Extensions, Microsoft.Owin, Microsoft.Owin.Host.SystemWeb, Microsoft.Owin.Security, Microsoft.Owin.Security.Cookies, Microsoft.Owin.Security.OpenIdConnect, Owin, System.IdentityModel.Tokens.Jwt

###### .NET 引用

Microsoft.IdentityModel.Protocol.Extensions, Microsoft.Owin, Microsoft.Owin.Host.SystemWeb, Microsoft.Owin.Security, Microsoft.Owin.Security.Cookies, Microsoft.Owin.Security.OpenIdConnect, Owin, System, System.Data, System.Drawing, System.IdentityModel, System.IdentityModel.Tokens.Jwt, System.Runtime.Serialization

###### 代码文件已添加到您的项目

身份验证启动类 App\_Start/Startup.Auth.cs（包含 Azure AD 身份验证的启动逻辑）已添加到您的项目。此外，还添加了控制器类 Controllers/AccountController.cs，其中包含 SignIn() 和 SignOut() 方法。最后，添加了分部视图 Views/Shared/\_LoginPartial.cshtml（包含 SignIn/SignOut 的操作链接）。

###### 启动代码已添加到您的项目

如果您的项目中已经有一个 Startup 类，Configuration() 方法将进行更新，以包括对添加到该方法的 ConfigureAuth(app) 的调用。否则，Startup 类已添加到您的项目。

###### 您的 app.config 或 web.config 具有新的配置值

已添加以下配置条目。

    <appSettings>
        <add key="ida:ClientId" value="ClientId from the new Azure AD App" />
        <add key="ida:Tenant" value="Your selected Azure AD Tenant" />
        <add key="ida:AADInstance" value="https://login.windows.net/{0}" />
        <add key="Ida:PostLogoutRedirectURI" value="Your project start page" />
    </appSettings> 

</p>
###### 已创建 Azure Active Directory (AD) 应用

已在您在向导中选定的目录内创建一个 Azure AD 应用程序。

## Azure Active Directory (AD) 入门

下面是您可以对已添加的代码执行的操作。

###### 访问控制器需要身份验证

您项目中的所有控制器均带有 [Authorize] 属性。此属性要求用户先进行身份验证，然后才能访问这些控制器。若要允许匿名访问控制器，请从控制器删除此属性。

###### 添加 SignIn/SignOut 控件

若要将 SignIn/SignOut 控件添加到您的视图，您可以使用 \_LoginPartial.cshtml 分部视图将该功能添加到您的某个视图。下面是已添加到标准 \_Layout.cshtml 视图的功能的示例：

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>@ViewBag.Title - My ASP.NET Application</title>
        @Styles.Render("~/Content/css")
        @Scripts.Render("~/bundles/modernizr")
    </head>
    <body>
        <div class="navbar navbar-inverse navbar-fixed-top">
            <div class="container">
                <div class="navbar-header">
                    <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                    </button>
                    @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
                </div>
                <div class="navbar-collapse collapse">
                    <ul class="nav navbar-nav">
                        <li>@Html.ActionLink("Home", "Index", "Home")</li>
                        <li>@Html.ActionLink("About", "About", "Home")</li>
                        <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
                    </ul>
                    @Html.Partial("_LoginPartial")
                </div>
            </div>
        </div>
        <div class="container body-content">
            @RenderBody()
            <hr />
            <footer>
                <p>&copy; @DateTime.Now.Year - My ASP.NET Application</p>
            </footer>
        </div>
        @Scripts.Render("~/bundles/jquery")
        @Scripts.Render("~/bundles/bootstrap")
        @RenderSection("scripts", required: false)
    </body>
    </html> 
