<properties 
	pageTitle="开始使用 Azure Active Directory 和 Visual Studio 连接服务（MVC 项目）| Azure" 
	description="通过 Visual Studio 连接服务连接到或创建 Azure AD 之后，如何在 MVC 项目中开始使用 Azure Active Directory" 
	services="active-directory" 
	documentationCenter="" 
	authors="TomArcher" 
	manager="douge" 
	editor=""/>
  
<tags 
	ms.service="active-directory"  
	ms.date="03/28/2016" 
	wacn.date="07/05/2016"/>

# 开始使用 Azure Active Directory 和 Visual Studio 连接服务（MVC 项目）

> [AZURE.SELECTOR]
> -   [入门](/documentation/articles/vs-active-directory-dotnet-getting-started/)
> -   [发生了什么情况](/documentation/articles/vs-active-directory-dotnet-what-happened/)

##访问控制器需要身份验证 

您项目中的所有控制器均带有 **Authorize** 属性。此属性要求用户先进行身份验证，然后才能访问这些控制器。若要允许匿名访问控制器，请从控制器删除此属性。如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。
 
##添加 SignIn/SignOut 控件 

若要将 SignIn/SignOut 控件添加到您的视图，您可以使用 **\_LoginPartial.cshtml** 分部视图将该功能添加到您的某个视图。下面是已添加到标准 **\_Layout.cshtml** 视图的功能的示例。（注意带有 navbar-collapse 类的 div 中的最后一个元素）：

<pre>
    &lt;!DOCTYPE html> 
     &lt;html> 
     &lt;head> 
         &lt;meta charset="utf-8" /> 
        &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"> 
        &lt;title>@ViewBag.Title - My ASP.NET Application&lt;/title> 
        @Styles.Render("~/Content/css") 
        @Scripts.Render("~/bundles/modernizr") 
    &lt;/head> 
    &lt;body> 
        &lt;div class="navbar navbar-inverse navbar-fixed-top"> 
            &lt;div class="container"> 
                &lt;div class="navbar-header"> 
                    &lt;button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse"> 
                        &lt;span class="icon-bar">&lt;/span> 
                        &lt;span class="icon-bar">&lt;/span> 
                        &lt;span class="icon-bar">&lt;/span> 
                    &lt;/button> 
                    @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" }) 
                &lt;/div> 
                &lt;div class="navbar-collapse collapse"> 
                    &lt;ul class="nav navbar-nav"> 
                        &lt;li>@Html.ActionLink("Home", "Index", "Home")&lt;/li> 
                        &lt;li>@Html.ActionLink("About", "About", "Home")&lt;/li> 
                        &lt;li>@Html.ActionLink("Contact", "Contact", "Home")&lt;/li> 
                    &lt;/ul> 
                    <span style="background-color:yellow">@Html.Partial("_LoginPartial")</span> 
                &lt;/div> 
            &lt;/div> 
        &lt;/div> 
        &lt;div class="container body-content"> 
            @RenderBody() 
            &lt;hr /> 
            &lt;footer> 
                &lt;p>&amp;copy; @DateTime.Now.Year - My ASP.NET Application&lt;/p> 
            &lt;/footer> 
        &lt;/div> 
        @Scripts.Render("~/bundles/jquery") 
        @Scripts.Render("~/bundles/bootstrap") 
        @RenderSection("scripts", required: false) 
    &lt;/body> 
    &lt;/html>
</pre>

[详细了解 Azure Active Directory](/home/features/identity/)

<!---HONumber=Mooncake_0620_2016-->