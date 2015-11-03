<properties 
	pageTitle="在 Azure 网站中通过 Azure Active Directory 身份验证创建 .NET MVC Web 应用" 
	description="了解如何在使用 Azure Active Directory 进行身份验证的 Azure 网站中创建 ASP.NET MVC 业务线应用程序。" 
	services="app-service\web, active-directory" 
	documentationCenter=".net" 
	authors="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.date="07/07/2015" 
	wacn.date="08/29/2015"/>

# 在 Azure 网站中通过 Azure Active Directory 身份验证创建 .NET MVC Web 应用 #

在本文中，您将了解如何在 [Azure Active Directory](/documentation/services/identity/) 用作标识提供程序的 [Azure App Service Web Apps](/documentation/services/web-sites/) 中创建 ASP.NET MVC 业务线应用程序。您还将了解如何使用 [Azure Active Directory Graph 客户端库](http://blogs.msdn.com/b/aadgraphteam/archive/2014/06/02/azure-active-directory-graph-client-library-1-0-publish.aspx)查询应用程序中的目录数据。

使用的 Azure Active Directory 租户可以是仅限 Azure 的目录，或者与本地 Active Directory (AD) 进行目录同步，以便为本地或远程的辅助角色创建单一登录体验。

>[AZURE.NOTE]Azure 网站中的简单身份验证功能使您能够设置通过单击几下按钮即可轻松完成对 Azure Active Directory 租户的身份验证。有关详细信息，请参阅[使用 Active Directory 在 Azure 网站中进行身份验证](/documentation/articles/web-sites-authentication-authorization)。

- [要生成的项目](#bkmk_build)
- [所需的项目](#bkmk_need)
- [将示例应用程序用于 LOB 模板](#bkmk_sample)
- [运行示例应用程序](#bkmk_run)
- [将示例应用程序部署到 Azure 网站](#bkmk_deploy)
- [将 LOB 功能添加到示例应用程序](#bkmk_crud)
- [其他资源](#bkmk_resources)

<a name="bkmk_build"></a>
## 要生成的项目 ##

您将在 Azure 网站中生成用于跟踪工作项并具有以下功能的简单的业务线创建-读取-更新-删除 (CRUD) 应用程序：

- 根据 Azure Active Directory 对用户进行身份验证
- 实现登录和注销功能
- 使用 `[Authorize]` 授权用户执行不同 CRUD 操作
- 使用 [Azure Active Directory Graph API](http://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx) 查询 Azure Active Directory 数据
- 使用 [Microsoft.Owin](http://www.asp.net/aspnet/overview/owin-and-katana/an-overview-of-project-katana)（而不是 Windows Identity Foundation，即 WIF），它代表了 ASP.NET 的未来，与 WIF 相比，它的身份验证和授权设置要简单得多

<a name="bkmk_need"></a>
## 所需的项目 ##

[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]

若要完成本教程，您需要以下项目：

- 包含各组中的用户的 Azure Active Directory 租户
- 在 Azure Active Directory 租户上创建应用程序的权限
- Visual Studio 2013
- [Azure SDK 2.5.1](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids) 或更高版本

<a name="bkmk_sample"></a>
## 使用业务线模板的示例应用程序 ##

本教程中的示例应用程序 [WebApp-GroupClaims-DotNet](https://github.com/AzureADSamples/WebApp-GroupClaims-DotNet) 由 Azure Active Directory 团队创建，可用作模板来轻松创建新的业务线应用程序。该示例应用程序具有以下内置功能：

- 使用 [OpenID Connect](http://openid.net/connect/) 通过 Azure Active Directory 进行身份验证
- `Roles` 控制器包含 Azure Active Directory 搜索筛选器，并使您能够轻松地将 Azure Active Directory 用户或组映射到应用程序角色。
- 示例 `TaskTracker` 控制器演示了如何才能对应用程序中的特定操作授权不同的角色，其中包括 `[Authorize]` 的标准用法。 

![](./media/web-sites-dotnet-lob-application-azure-ad/role-management.png)

<a name="bkmk_run" />
## 运行示例应用程序 ##

1.	将 [WebApp-GroupClaims-DotNet](https://github.com/AzureADSamples/WebApp-GroupClaims-DotNet) 中的示例解决方案克隆或下载到本地目录。

2.	按照 [README.md](https://github.com/AzureADSamples/WebApp-GroupClaims-DotNet/blob/master/README.md) 中的说明设置 Azure Active Directory 应用程序和项目。

	<div class="wa-note">
	<span class="wa-icon-bulb"></span>
	<p>AAD 应用程序中配置的权限仅需要<strong>用户</strong>角色，而不是<strong>全局管理员</strong>。</p>
</div>
	
3.	完成配置应用程序后，键入 `F5` 运行该应用程序。

4.	加载应用程序之后，请单击**登录**。

5.	如果您已正确配置 Azure Active Directory 应用程序并在 Web.config 中设置相应的设置，您应被重定向到日志中。只需使用在 Azure 门户中创建 Azure Active Directory 应用程序时使用的帐户登录，因为它是 Azure Active Directory 应用程序默认的所有者。
	
	<div class="wa-note">
	<span class="wa-icon-bulb"></span>
	<p>在示例项目的 Startup.Auth.cs 中，请注意该应用程序具有一个名为 <code>AddOwnerAdminClaim</code> 的方法，用于将应用程序所有者添加到管理员角色。这样您便可以立即开始在<code>角色</code>控制器中管理应用程序角色。</p>
</div>
	
4.	登录后，单击**角色**管理应用程序角色。

5.	在**搜索用户/组**中，开始键入所需的用户名或组名，并注意，一个下拉列表显示了 Azure Active Directory 租户中的用户和/或组的筛选列表。

	![](./media/web-sites-dotnet-lob-application-azure-ad/select-user-group.png)

	<div class="wa-note">	<span class="wa-icon-bulb"></span>
		<p>在视图\角色\ Index.cshtml 中，你会看到该视图使用名为 <code>AadPicker</code> 的 JavaScript 对象（在脚本\AadPickerLibrary.js 中定义）访问 <code>Roles</code> 控制器中的 <code>Search</code> 操作。</p>
		<pre class="prettyprint">var searchUrl = window.location.protocol + "//" + window.location.host + "<mark>/Roles/Search</mark>";
	...
	var picker = new <mark>AadPicker (searchUrl, maxResultsPerPage, input, token, tenant)</mark>;</pre>在 	Controllers\\RolesController.cs 中，您将看到<code>搜索</code>操作，它将实际请求发送到 Azure Active 	Directory Graph API 并将响应返回到页面。稍后，您将使用相同的方法在应用程序中创建简单的功能。

6.	从下拉列表中选择用户或组，选择一个角色，然后单击**分配角色**。

<a name="bkmk_deploy"></a>
## 将示例应用程序部署到 Azure 网站

现在，您需要将应用程序发布到 Azure 网站中的 Web 应用。[README.md](https://github.com/AzureADSamples/WebApp-GroupClaims-DotNet/blob/master/README.md) 中已经提供了有关部署到 Azure 网站的说明，但这些步骤还取消了本地调试环境的配置。下面将介绍如何在保留调试配置的同时进行部署。

1. 右键单击您的项目并选择**发布**。

	![](./media/web-sites-dotnet-lob-application-azure-ad/publish-app.png)

2. 选择 **Windows Azure Web Apps**。

3. 如果您尚未登录到 Azure，请单击**登录**，然后使用 Azure 订阅的 Microsoft 帐户登录。

4. 登录后，单击**新建**在 Azure 中创建新的 Web 应用。

5. 填写所有必填字段。您需要使用此应用程序的数据库连接来存储角色映射、缓存的令牌以及任何应用程序数据。

	![](./media/web-sites-dotnet-lob-application-azure-ad/4-create-website.png)

6. 单击**创建**。创建 Web 应用后，将打开**发布 Web** 对话框。

7. 在**目标 URL** 中，将 **http** 更改为 **https**。将整个 URL 复制到文本编辑器。稍后将要用到它。然后单击**下一步**。

	![](./media/web-sites-dotnet-lob-application-azure-ad/5-change-to-https.png)

8. 清除**启用组织身份验证**复选框。

	![](./media/web-sites-dotnet-lob-application-azure-ad/6-disable-organizational-authentication.png)

9. 此时请不要单击**发布**转到 Web 发布过程，而是单击**关闭**。单击**是**保存对发布配置文件所做的更改。

2. 在 [Azure 管理门户](https://manage.windowsazure.cn) 转到您的 Azure Active Directory 租户，然后单击**应用程序**选项卡。

2. 单击页面底部的**添加**。

3. 选择 **Web 应用程序和/或 Web API**。

4. 为应用程序提供一个名称，然后单击**下一步**。

5. 在“应用程序属性”中，将**登录 URL** 设置为您前面保存的 Web 应用 URL（例如 `https://<site-name>.chinacloudsites.cn`），并将**应用 ID URI** 设置为 `https://<aad-tenanet-name>/<app-name>`。然后，单击**完成**。

	![](./media/web-sites-dotnet-lob-application-azure-ad/7-app-properties.png)

6. 创建应用程序后，单击**配置**。

7. 在**键**下，通过从下拉列表中选择 **1 年**创建新键。

8. 在 **Azure Active Directory** 条目的**对其他应用程序的权限**下，在**委派权限**下拉列表中选择**访问组织的目录**。

	<div class="wa-note">
	<span class="wa-icon-bulb"></span>
	<p>此处需要的确切权限取决于您的应用程序所需的功能。某些权限要求设置<strong>全局管理员</strong>角色，但本教程所需的权限仅需要<strong>用户</strong>角色。</p>
</div>

9.  单击**保存**。

10.  您通过导航离开已保存的配置页之前，请将以下信息复制到文本编辑器中。

	-	客户端 ID
	-	键（如果导航离开页面时，您将不能再次看到此键）

11. 在 Visual Studio 中，打开项目中的 **Web.Release.config**。将以下 XML 插入 `<configuration>` 标记，并将每个键的值替换为新的 Azure Active Directory 应用程序保存的信息。
	<pre class="prettyprint">
&lt;appSettings&gt;
   &lt;add key="ida:ClientId" value="<mark>[e.g. 82692da5-a86f-44c9-9d53-2f88d52b478b]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /&gt;
   &lt;add key="ida:AppKey" value="<mark>[e.g. rZJJ9bHSi/cYnYwmQFxLYDn/6EfnrnIfKoNzv9NKgbo=]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /&gt;
   &lt;add key="ida:PostLogoutRedirectUri" value="<mark>[e.g. https://mylobapp.chinacloudsites.cn/]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /&gt;
&lt;/appSettings&gt;</pre>

	请确保 ida:PostLogoutRedirectUri 的值以斜杠“/”结尾。

1. 右键单击您的项目并选择**发布**。

2. 单击**发布**以发布到 Azure 网站。

完成此操作后，将在 Azure 管理门户中配置两个 Azure Active Directory 应用程序，一个用于 Visual Studio 中的调试环境，另一个用于 Azure 中发布的 Web 应用。在调试期间，将使用 Web.config 中的应用设置来使**调试**配置适用于 Azure Active Directory，发布配置（默认情况下，会发布**版本**配置）时，将上载转换的 Web.config，其中包含 Web.Release.config 中的应用设置更改。

如果您想要将已发布的 Web 应用附加到调试器（即必须上载已发布的 Web 应用中的代码的调试符号），您可以创建一个调试配置的克隆进行 Azure 调试，但其自定义的 Web.config 转换（例如 Web.AzureDebug.config) 使用 Web.Release.config 中的 Azure Active Directory 设置。这样，您可以跨不同的环境维护静态配置。

<a name="bkmk_crud"></a>
## 将业务线功能添加到示例应用程序

在本教程的本部分中，您将学习如何基于示例应用程序生成所需的业务线功能。您将创建一个简单 CRUD 的工作项跟踪程序，类似于 TaskTracker 控制器，但使用标准的 CRUD 基架和设计模式。您还将使用包含的 Scripts\\AadPickerLibrary.js 使您的应用程序富有 Azure Active Directory Graph API 的数据。

5.	在 Models 文件夹中创建名为 WorkItem.cs 的新模型，并将代码替换为以下代码：

		using System.ComponentModel.DataAnnotations;
		
		namespace WebAppGroupClaimsDotNet.Models
		{
		    public class WorkItem
		    {
		        [Key]
		        public int ItemID { get; set; }
		        public string AssignedToID { get; set; }
		        public string AssignedToName { get; set; }
		        public string Description { get; set; }
		        public WorkItemStatus Status { get; set; }
		    }
		
		    public enum WorkItemStatus
		    {
		        Open, 
		        Investigating, 
		        Resolved, 
		        Closed
		    }
		}

6.	打开 DAL\\GroupClaimContext.cs 并添加突出显示的代码：
	<pre class="prettyprint">
    public class GroupClaimContext : DbContext
    {
        public GroupClaimContext() : base("GroupClaimContext") { }

        public DbSet&lt;RoleMapping&gt; RoleMappings { get; set; }
        public DbSet&lt;Task&gt; Tasks { get; set; }
        <mark>public DbSet&lt;WorkItem&gt; WorkItems { get; set; }</mark>
        public DbSet&lt;TokenCacheEntry&gt; TokenCacheEntries { get; set; }
    }</pre>

7.	生成项目，以便能够通过 Visual Studio 中的基架逻辑访问您的新模型。

8.	将新的基架项 `WorkItemsController` 添加到 Controllers 文件夹。为此，请右键单击**控制器**，指向**添加**，然后选择**新的基架项**。

9.	选择**使用实体框架的包含视图的 MVC 5 控制器**并单击**添加**。

10.	选择刚创建的模型并单击**添加**。

	![](./media/web-sites-dotnet-lob-application-azure-ad/8-add-scaffolded-controller.png)

9.	打开 Controllers\\WorkItemsController.cs

11. 将突出显示的 [Authorize] 修饰添加到下面的相应操作。
	<pre class="prettyprint">
	...

    <mark>[Authorize(Roles = "Admin, Observer, Writer, Approver")]</mark>
    public class WorkItemsController : Controller
    {
		...

        <mark>[Authorize(Roles = "Admin, Writer")]</mark>
        public ActionResult Create()
        ...

        <mark>[Authorize(Roles = "Admin, Writer")]</mark>
        public async Task&lt;ActionResult&gt; Create([Bind(Include = "ItemID,AssignedToID,AssignedToName,Description,Status")] WorkItem workItem)
        ...

        <mark>[Authorize(Roles = "Admin, Writer")]</mark>
        public async Task&lt;ActionResult&gt; Edit(int? id)
        ...

        <mark>[Authorize(Roles = "Admin, Writer")]</mark>
        public async Task&lt;ActionResult&gt; Edit([Bind(Include = "ItemID,AssignedToID,AssignedToName,Description,Status")] WorkItem workItem)
        ...

        <mark>[Authorize(Roles = "Admin, Writer, Approver")]</mark>
        public async Task&lt;ActionResult&gt; Delete(int? id)
        ...

        <mark>[Authorize(Roles = "Admin, Writer, Approver")]</mark>
        public async Task&lt;ActionResult&gt; DeleteConfirmed(int id)
        ...
	}</pre>

	因为您需要考虑角色控制器中的角色映射，您需要做是确保每个操作授予适当的角色。

	<div class="wa-note">
		<span class="wa-icon-bulb"></span>
		<p>您可能已经注意到有些操作中含有 <code>[ValidateAntiForgeryToken]</code> 修饰。由于 <a href="https://twitter.com/BrockLAllen">Brock Allen</a> 在 <a href="http://brockallen.com/2012/07/08/mvc-4-antiforgerytoken-and-claims/">MVC 4、AntiForgeryToken 和声明</a>中所描述的行为，HTTP POST 可能无法进行防伪令牌验证，因为：</p>
		<ul>
		<li>AAD 不会发送 http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider，默认情况下要求防伪标记执行此操作。</li>
		<li>如果 AAD 是使用 AD FS 同步的目录，默认情况下的 AD FS 信任不发送 http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider 声明，尽管您可以手动配置 AD FS 以发送此声明。</li>
		</ul>
		<p>您将在下一步对此进行处理。</p>
	</div>

12.  在 App_Start\\Startup.Auth.cs 中，将以下代码行添加到 `ConfigureAuth` 方法中：

		AntiForgeryConfig.UniqueClaimTypeIdentifier = ClaimTypes.NameIdentifier;
	
	`ClaimTypes.NameIdentifies` 指定声明 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier`（由 Azure Active Directory 提供）。既然您已注意了授权部分（严格来讲，这并不长），您可以将时间投入到操作的实际功能。

13.	在 Create() 和 Edit() 中添加下列代码以使某些变量可稍后供您的 JavaScript 使用：ViewData["token"] = GraphHelper.AcquireToken(ClaimsPrincipal.Current.FindFirst(Globals.ObjectIdClaimType).Value); ViewData["tenant"] = ConfigHelper.Tenant;

14.	在 Views\\WorkItems\\Create.cshtml（自动搭建基架的项）中，找到 `Html.BeginForm` 帮助器方法并对其进行如下修改：
	<pre class="prettyprint">@using (Html.BeginForm(<mark>"Create", "WorkItems", FormMethod.Post, new { id = "main-form" }</mark>))
	{
	    @Html.AntiForgeryToken()
	    
	    &lt;div class="form-horizontal"&gt;
	        &lt;h4&gt;WorkItem&lt;/h4&gt;
	        &lt;hr /&gt;
	        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
	
	        &lt;div class="form-group"&gt;
	            &lt;div class="col-md-10"&gt;
	                @Html.EditorFor(model =&gt; model.AssignedToID, new { htmlAttributes = new { @class = "form-control"<mark>, @type=&quot;hidden&quot;</mark> } })
	                @Html.ValidationMessageFor(model =&gt; model.AssignedToID, "", new { @class = "text-danger" })
	            &lt;/div&gt;
	        &lt;/div&gt;
	
	        &lt;div class="form-group"&gt;
	            @Html.LabelFor(model =&gt; model.AssignedToName, htmlAttributes: new { @class = "control-label col-md-2" })
	            &lt;div class="col-md-10"&gt;
	                @Html.EditorFor(model =&gt; model.AssignedToName, new { htmlAttributes = new { @class = "form-control" } })
	                @Html.ValidationMessageFor(model =&gt; model.AssignedToName, "", new { @class = "text-danger" })
	            &lt;/div&gt;
	        &lt;/div&gt;
	
	        &lt;div class="form-group"&gt;
	            @Html.LabelFor(model =&gt; model.Description, htmlAttributes: new { @class = "control-label col-md-2" })
	            &lt;div class="col-md-10"&gt;
	                @Html.EditorFor(model =&gt; model.Description, new { htmlAttributes = new { @class = "form-control" } })
	                @Html.ValidationMessageFor(model =&gt; model.Description, "", new { @class = "text-danger" })
	            &lt;/div&gt;
	        &lt;/div&gt;
	
	        &lt;div class="form-group"&gt;
	            @Html.LabelFor(model =&gt; model.Status, htmlAttributes: new { @class = "control-label col-md-2" })
	            &lt;div class="col-md-10"&gt;
	                @Html.EnumDropDownListFor(model =&gt; model.Status, htmlAttributes: new { @class = "form-control" })
	                @Html.ValidationMessageFor(model =&gt; model.Status, "", new { @class = "text-danger" })
	            &lt;/div&gt;
	        &lt;/div&gt;
	
	        &lt;div class="form-group"&gt;
	            &lt;div class="col-md-offset-2 col-md-10"&gt;
	                &lt;input type="submit" value="Create" class="btn btn-default" <mark>id="submit-button"</mark> /&gt;
	            &lt;/div&gt;
	        &lt;/div&gt;
	    &lt;/div&gt;
	
	    <mark>&lt;script&gt;
	            // People/Group Picker Code
	            var maxResultsPerPage = 14;
	            var searchUrl = window.location.protocol + "//" + window.location.host + "/Roles/Search";
	            var input = document.getElementById("AssignedToName");
	            var token = "@ViewData["token"]";
	            var tenant = "@ViewData["tenant"]";
	
	            var picker = new AadPicker(searchUrl, maxResultsPerPage, input, token, tenant);
	
	            // Submit the selected user/group to be asssigned.
	            $("#submit-button").click({ picker: picker }, function () {
	                if (!picker.Selected())
	                    return;
	                $("#main-form").get()[0].elements["AssignedToID"].value = picker.Selected().objectId;
	            });
	    &lt;/script&gt;</mark>
	
	}</pre>

	在脚本中，AadPicker 对象为匹配输入的 Azure Active Directory 用户和组搜索 `~/Roles/Search` 操作。然后，在单击提交按钮后，AadPicker 对象将用户 ID 保存到隐藏的 `AssignedToID` 字段。

15. 现在，在 Visual Studio 调试器中运行应用程序，或者发布到 Azure 网站。以应用程序所有者的身份登录并导航到 `~/WorkItems/Create`。对于发布的业务线应用程序，将导航至 `https://mylobapp.chinacloudsites.cn/WorkItems/Create`。您将看到，现在您可以使用相同的 AadPicker 搜索筛选器来选取 Azure Active Directory 用户。

	![](./media/web-sites-dotnet-lob-application-azure-ad/9-create-workitem.png)

16. 填写表单的其余部分并单击**创建**。~/WorkItems/Index 页现在将显示新建的工作项。您还会发现，在下面的屏幕截图中，我删除了 Views\\WorkItems\\Index.cshtml 中的 `AssignedToID` 列。

	![](./media/web-sites-dotnet-lob-application-azure-ad/10-workitem-index.png)

11.	现在，请对**编辑**视图进行类似的更改。在 Views\\WorkItems\\Edit.cshtml 中，更改与上一步的 Views\\WorkItems\\Create.cshtml 中相同的 `Html.BeginForm` 帮助器方法（在上面突出显示的代码中用“编辑”替换“创建”）。

就这么简单！

在 WorkItems 控制器中配置授权和不同操作的业务线功能后，您可以尝试以不同应用程序角色的用户身份登录。

![](./media/web-sites-dotnet-lob-application-azure-ad/11-edit-unauthorized.png)

<a name="bkmk_resources"></a>
## 其他资源

- [通过 SSL 和 Authorize 属性保护应用程序](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database#protect-the-application-with-ssl-and-the-authorize-attribute)
- [使用 Active Directory 在 Azure 网站中进行身份验证](/documentation/articles/web-sites-authentication-authorization)
- [在 Azure 网站中通过 AD FS 身份验证创建 .NET MVC Web 应用](/documentation/articles/web-sites-dotnet-lob-application-adfs)
- [Windows Azure Active Directory 示例和文档](https://github.com/AzureADSamples)
- [Vittorio Bertocci 的博客](http://blogs.msdn.com/b/vbertocci/)
- [将 VS2013 Web 项目从 WIF 迁移到 Katana](http://www.cloudidentity.com/blog/2014/09/15/MIGRATE-A-VS2013-WEB-PROJECT-FROM-WIF-TO-KATANA/)
- [Active Directory 与 Azure AD 之间的相似之处](http://technet.microsoft.com/zh-cn/library/dn518177.aspx)
- [使用单一登录方案进行目录同步](http://technet.microsoft.com/zh-cn/library/dn441213.aspx)
- [Azure AD 支持的令牌和声明类型](/documentation/articles/active-directory-token-and-claims)
 

<!---HONumber=67-->