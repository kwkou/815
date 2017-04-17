<properties
    pageTitle="Visual Studio MVC 项目中的 Azure AD 入门 | Azure"
    description="通过 Visual Studio 连接服务连接到或创建 Azure AD 之后，如何在 MVC 项目中开始使用 Azure Active Directory"
    services="active-directory"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />

<tags
    ms.assetid="1c8b6a58-5144-4965-a905-625b9ee7b22b"
    ms.service="active-directory"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/01/2017"
    ms.author="tarcher"
    translationtype="Human Translation"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="6904e5e8511e4eb6b65d0fb94819cf8ecb17522b"
    ms.lasthandoff="04/07/2017"
    wacn.date="04/17/2017" />



# <a name="getting-started-with-azure-active-directory-and-visual-studio-connected-services-mvc-projects"></a>开始使用 Azure Active Directory 和 Visual Studio 连接服务（MVC 项目）
> [AZURE.SELECTOR]
>- [入门](/documentation/articles/vs-active-directory-dotnet-getting-started/)
>- [发生了什么情况](/documentation/articles/vs-active-directory-dotnet-what-happened/)

## <a name="requiring-authentication-to-access-controllers"></a>访问控制器需要身份验证
你项目中的所有控制器均带有 **Authorize** 属性。 此属性要求用户先进行身份验证，然后才能访问这些控制器。 若要允许匿名访问控制器，请从控制器删除此属性。 如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。

## <a name="adding-signin--signout-controls"></a>添加 SignIn/SignOut 控件
若要将 SignIn/SignOut 控件添加到视图，可使用 **_LoginPartial.cshtml** 分部视图将该功能添加到某个视图。 下面是已添加到标准 **_Layout.cshtml** 视图的功能示例。 （注意带有 navbar-collapse 类的 div 中的最后一个元素）：

<pre>
    &lt;!DOCTYPE html&gt; 
     &lt;html&gt; 
     &lt;head&gt; 
         &lt;meta charset="utf-8" /&gt; 
        &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt; 
        &lt;title&gt;@ViewBag.Title - My ASP.NET Application&lt;/title&gt; 
        @Styles.Render("~/Content/css") 
        @Scripts.Render("~/bundles/modernizr") 
    &lt;/head&gt; 
    &lt;body&gt; 
        &lt;div class="navbar navbar-inverse navbar-fixed-top"&gt; 
            &lt;div class="container"&gt; 
                &lt;div class="navbar-header"&gt; 
                    &lt;button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse"&gt; 
                        &lt;span class="icon-bar"&gt;&lt;/span&gt; 
                        &lt;span class="icon-bar"&gt;&lt;/span&gt; 
                        &lt;span class="icon-bar"&gt;&lt;/span&gt; 
                    &lt;/button&gt; 
                    @Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" }) 
                &lt;/div&gt; 
                &lt;div class="navbar-collapse collapse"&gt; 
                    &lt;ul class="nav navbar-nav"&gt; 
                        &lt;li&gt;@Html.ActionLink("Home", "Index", "Home")&lt;/li&gt; 
                        &lt;li&gt;@Html.ActionLink("About", "About", "Home")&lt;/li&gt; 
                        &lt;li&gt;@Html.ActionLink("Contact", "Contact", "Home")&lt;/li&gt; 
                    &lt;/ul&gt; 
                    <span style="background-color:yellow">@Html.Partial("_LoginPartial")</span> 
                &lt;/div&gt; 
            &lt;/div&gt; 
        &lt;/div&gt; 
        &lt;div class="container body-content"&gt; 
            @RenderBody() 
            &lt;hr /&gt; 
            &lt;footer&gt; 
                &lt;p&gt;&amp;copy; @DateTime.Now.Year - My ASP.NET Application&lt;/p&gt; 
            &lt;/footer&gt; 
        &lt;/div&gt; 
        @Scripts.Render("~/bundles/jquery") 
        @Scripts.Render("~/bundles/bootstrap") 
        @RenderSection("scripts", required: false) 
    &lt;/body&gt; 
    &lt;/html&gt;
</pre>

## <a name="next-steps"></a>后续步骤
- [详细了解 Azure Active Directory](/home/features/identity/)
<!-- Update_Description: wording update -->
