<properties title="Active Directory 身份验证入门" pageTitle="" metaKeywords="Azure, Getting Started, Active Directory" description="" services="active-directory" documentationCenter="" authors="ghogen, kempb" />

<tags ms.service="active-directory" ms.workload="web" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/8/2014" ms.author="ghogen, kempb"></tags>

> [AZURE.SELECTOR]
>
> -   [入门][入门]
> -   [发生了什么情况][发生了什么情况]

## Azure Active Directory 入门（.NET 项目）

##### 访问控制器需要身份验证

您项目中的所有控制器均带有 **Authorize** 属性。此属性要求用户先进行身份验证，然后才能访问这些控制器。若要允许匿名访问控制器，请从控制器删除此属性。如果您想要更详细地设置这些权限，请将该属性应用到需要身份验证的每个方法，而不是将它应用到控制器类。

##### 添加 SignIn/SignOut 控件

若要将 SignIn/SignOut 控件添加到您的视图，您可以使用 \*\*\_LoginPartial.cshtml\*\* 分部视图将该功能添加到您的某个视图。下面是已添加到标准 \*\*\_Layout.cshtml\*\* 视图的功能的示例。（注意带有 navbar-collapse 类的 div 中的最后一个元素）：

<pre class="prettyprint">
&lt;!DOCTYPE html&gt; 
&lt;html&gt; 
&lt;head&gt; 
&lt;meta charset=&quot;utf-8&quot; /&gt; 
&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt; 
&lt;title&gt;@ViewBag.Title - My ASP.NET Application&lt;/title&gt; 
@Styles.Render(&quot;~/Content/css&quot;) 
@Scripts.Render(&quot;~/bundles/modernizr&quot;) 
&lt;/head&gt; 
&lt;body&gt; 
&lt;div class=&quot;navbar navbar-inverse navbar-fixed-top&quot;&gt; 
&lt;div class=&quot;container&quot;&gt; 
&lt;div class=&quot;navbar-header&quot;&gt; 
&lt;button type=&quot;button&quot; class=&quot;navbar-toggle&quot; data-toggle=&quot;collapse&quot; data-target=&quot;.navbar-collapse&quot;&gt; 
&lt;span class=&quot;icon-bar&quot;&gt;&lt;/span&gt; 
&lt;span class=&quot;icon-bar&quot;&gt;&lt;/span&gt; 
&lt;span class=&quot;icon-bar&quot;&gt;&lt;/span&gt; 
&lt;/button&gt; 
@Html.ActionLink(&quot;Application name&quot;, &quot;Index&quot;, &quot;Home&quot;, new { area = &quot;&quot; }, new { @class = &quot;navbar-brand&quot; }) 
&lt;/div&gt; 
&lt;div class=&quot;navbar-collapse collapse&quot;&gt; 
&lt;ul class=&quot;nav navbar-nav&quot;&gt; 
&lt;li&gt;@Html.ActionLink(&quot;Home&quot;, &quot;Index&quot;, &quot;Home&quot;)&lt;/li&gt; 
&lt;li&gt;@Html.ActionLink(&quot;About&quot;, &quot;About&quot;, &quot;Home&quot;)&lt;/li&gt; 
&lt;li&gt;@Html.ActionLink(&quot;Contact&quot;, &quot;Contact&quot;, &quot;Home&quot;)&lt;/li&gt; 
&lt;/ul&gt; 
@Html.Partial(&quot;_LoginPartial&quot;) 
&lt;/div&gt; 
&lt;/div&gt; 
&lt;/div&gt; 
&lt;div class=&quot;container body-content&quot;&gt; 
@RenderBody() 
&lt;hr /&gt; 
&lt;footer&gt; 
&lt;p&gt;© @DateTime.Now.Year - My ASP.NET Application&lt;/p&gt; 
&lt;/footer&gt; 
&lt;/div&gt; 
@Scripts.Render(&quot;~/bundles/jquery&quot;) 
@Scripts.Render(&quot;~/bundles/bootstrap&quot;) 
@RenderSection(&quot;scripts&quot;, required:false) 
&lt;/body&gt; 
&lt;/html&gt;
</pre>

[详细了解 Azure Active Directory][详细了解 Azure Active Directory]

  [入门]: /documentation/articles/vs-active-directory-dotnet-getting-started/
  [发生了什么情况]: /documentation/articles/vs-active-directory-dotnet-what-happened/
  [详细了解 Azure Active Directory]: http://azure.microsoft.com/services/active-directory/
