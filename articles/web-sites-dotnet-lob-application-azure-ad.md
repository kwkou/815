<properties 
	pageTitle="在 Azure 中创建使用 Azure Active Directory 身份验证的 .NET MVC Web 应用" 
	description="学习如何在使用 Azure Active Directory 进行身份验证的 Azure 中创建 ASP.NET MVC 业务线应用程序" 
	services="app-service\web, active-directory" 
	documentationCenter=".net" 
	authors="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="02/29/2016" 
	wacn.date="05/24/2016"/>

# 在 Azure 中创建使用 Azure Active Directory 身份验证的 .NET MVC Web 应用#

在本文中，你将了解如何在使用本地 [Azure Active Directory](/home/features/identity/) 作为标识提供者的 [Azure Web 应用](/documentation/services/web-sites/)中创建 ASP.NET MVC 业务线应用程序。你还将了解如何使用 [Azure Active Directory Graph 客户端库](http://blogs.msdn.com/b/aadgraphteam/archive/2014/06/02/azure-active-directory-graph-client-library-1-0-publish.aspx)查询应用程序中的目录数据。

使用的 Azure Active Directory 租户可以是仅限 Azure 的目录，或者与本地 Active Directory (AD) 进行目录同步，以便为本地或远程的辅助角色创建单一登录体验。

>[AZURE.NOTE]对于 Azure Web 应用，只需单击几下鼠标，就能配置针对 Azure Active Directory 租户的身份验证。有关详细信息，请参阅[使用 Active Directory 在 Azure 中进行身份验证](/documentation/articles/web-sites-authentication-authorization/)。

##<a name="bkmk_build"></a> 要生成的项目 ##

你将在 Azure 中生成用于跟踪工作项并具有以下功能的简单的业务线创建-读取-更新-删除 (CRUD) 应用程序：

- 根据 Azure Active Directory 对用户进行身份验证
- 实现登录和注销功能
- 使用 `[Authorize]` 授权用户执行不同的 CRUD 操作
- 使用 [Azure Active Directory 图形 API](http://msdn.microsoft.com/zh-cn/library/azure/hh974476.aspx) 查询 Azure Active Directory 数据
- 使用 [Microsoft.Owin](http://www.asp.net/aspnet/overview/owin-and-katana/an-overview-of-project-katana)（而不是 Windows Identity Foundation (WIF)），它代表了 ASP.NET 的未来发展方向，与 WIF 相比，它的身份验证和授权设置要简单得多

##<a name="bkmk_need"></a> 所需的项目 ##

[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]

若要完成本教程，你需要以下项目：

- 一个 Azure Active Directory 租户，其中的用户已分配到不同的组
- 在 Azure Active Directory 租户上创建应用程序的权限
- Visual Studio 2013
- [Azure SDK 2.5.1](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids) 或更高版本

##<a name="bkmk_sample"></a> 将示例应用程序用作业务线模板 ##

本教程中的示例应用程序 [WebApp-RoleClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) 由 Azure Active Directory 团队创建，可用作模板来轻松创建新的业务线应用程序。该示例应用程序具有以下内置功能：

- 使用 [OpenID Connect](http://openid.net/connect/) 通过 Azure Active Directory 进行身份验证
- 示例 `TaskTracker` 控制器演示了如何才能授权对应用程序，其中包括 `[Authorize]` 的标准用法中的特定操作的不同角色的控制器。 
- 一个多租户应用程序，其中包含可立即分配给用户和组的预定义角色。 

##<a name="bkmk_run" ></a> 运行示例应用程序 ##

1.	克隆或下载 [WebApp-RoleClaims-DotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) 中的示例解决方案到本地目录。

2.	根据[如何将示例作为单租户应用运行](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims#how-to-run-the-sample-as-a-single-tenant-app)中的说明设置 Azure Active Directory 应用程序和项目。请务必遵照所有有关将多租户应用程序转换为单租户应用程序的说明。

3.	在刚刚创建的 Azure Active Directory 应用程序的 [Azure 经典管理门户](https://manage.windowsazure.cn)视图中，单击“用户”选项卡。然后，将所需的用户分配到所需的角色。

	>[AZURE.NOTE]如果除了分配到用户以外，还要将角色分配到组，则必须将你的 Azure Active Directory 租户升级到 [Active Directory Premium](/home/features/identity/pricing/)。在应用程序的经典管理门户 UI 中，如果你看到的是“用户”选项卡而不是“用户和组”选项卡，你可以转到 Azure Active Directory 租户的“许可证”选项卡来试用 Azure Active Directory Premium。

3.	配置完应用程序后，在 Visual Studio 中键入 `F5` 以运行 ASP.NET 应用程序。

4.	加载应用程序后，单击“登录”，并使用在 Azure 经典管理门户中具有管理员角色的用户身份登录。

5.	如果你已正确配置 Azure Active Directory 应用程序，并在 Web.config 中设置了相应的设置，则应会重定向到登录页。你只需使用在 Azure 经典管理门户中创建 Azure Active Directory 应用程序时所用的帐户登录，因为该帐户是 Azure Active Directory 应用程序的默认所有者。
	
##<a name="bkmk_deploy"></a> 将示例应用程序部署到 Azure Web 应用

现在，你需要将应用程序发布到 Azure 中的 Web 应用。[README.md](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims/blob/master/README.md) 中已经提供了有关部署到 Azure Web 应用的说明，但这些步骤还取消了本地调试环境的配置。下面将介绍如何在保留调试配置的同时进行部署。

1. 右键单击您的项目，然后选择“发布”。

	![](./media/web-sites-dotnet-lob-application-azure-ad/publish-app.png)

2. 单机“导入”，选择已下载的“发布配置文件”。

	如果还没有创建 Web 应用，可以登录 [Azure 经典管理门户](https://manage.windowsazure.cn/)创建一个，然后再“仪表板”的“速览”下，下载“发布配置文件”。

7. 在“目标 URL”中，将 **http** 更改为 **https**。将整个 URL 复制到文本编辑器。稍后将要用到它。然后，单击“下一步”。

	![](./media/web-sites-dotnet-lob-application-azure-ad/5-change-to-https.png)

8. 清除“启用组织身份验证”复选框。

	![](./media/web-sites-dotnet-lob-application-azure-ad/6-enable-code-first-migrations.png)

8. 展开“RoleClaimContext”并选择“执行 Code First 迁移(应用程序启动时运行)”复选框。稍后当你定义其他 Code First 数据模型时，[Code First 迁移](https://msdn.microsoft.com/data/jj591621.aspx)会帮助你在 Azure 中更新应用的数据库架构。

9. 此时请不要单击“发布”转到 Web 发布过程，而是单击“关闭”。单击“是”保存对发布配置文件所做的更改。

2. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中转到你的 Azure Active Directory 租户，然后单击“应用程序”选项卡。

2. 单击页面底部的“添加”。

2. 单击“添加我的组织正在开发的应用程序”。

3. 选择“ Web 应用和/或 Web API”。

4. 为应用程序提供一个名称，然后单击“下一步”。

5. 在“应用程序属性”中，将“登录 URL”设置为你前面保存的 Web 应用URL（例如 `https://<site-name>.chinacloudsites.cn/`），并将“应用 ID URI”设置为 `https://<aad-tenanet-name>/<app-name>`。然后，单击“完成”。

	![](./media/web-sites-dotnet-lob-application-azure-ad/7-app-properties.png)

2.	创建应用程序后，根据前面[定义应用程序角色](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims#step-2-define-your-application-roles)中的说明以相同的方式更新应用程序清单。

3.	在刚刚创建的 Azure Active Directory 应用程序的 [Azure 经典管理门户](https://manage.windowsazure.cn)视图中，单击“用户”选项卡。然后，将所需的用户分配到所需的角色。

6. 单击“配置”选项卡。

7. 在“键”下，通过从下拉列表中选择“1 年”创建新键。

8. 在“Azure Active Directory”条目的“对其他应用程序的权限”下，从“委派权限”下拉列表中选择“登录和读取用户配置文件”与“读取目录数据”。

	> [AZURE.NOTE]此处需要的确切权限取决于你的应用程序所需的功能。某些权限需要“全局管理员”角色才能设置，但本教程只需要“用户”角色。

9.  单击“保存”。

10.  你通过导航离开已保存的配置页之前，请将以下信息复制到文本编辑器中。

	-	客户端 ID
	-	键（如果导航离开页面时，你将不能再次看到密钥）

11. 在 Visual Studio 中，在项目中打开 **Web.Release.config**。将以下 XML 插入 `<configuration>` 标记，并将每个键的值替换为新的 Azure Active Directory 应用程序保存的信息。
	<pre class="prettyprint">
	&lt;appSettings><span>&#13;</span>
		&lt;add key="ida:ClientId" value="<mark>[e.g. 82692da5-a86f-44c9-9d53-2f88d52b478b]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /><span>&#13;</span>
		&lt;add key="ida:AppKey" value="<mark>[e.g. rZJJ9bHSi/cYnYwmQFxLYDn/6EfnrnIfKoNzv9NKgbo=]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /><span>&#13;</span>
		&lt;add key="ida:PostLogoutRedirectUri" value="<mark>[e.g. https://mylobapp.chinacloudsites.cn/]</mark>" xdt:Transform="SetAttributes" xdt:Locator="Match(key)" /><span>&#13;</span>
	&lt;/appSettings></pre>

	请确保 ida:PostLogoutRedirectUri 的值以斜杠“/”结尾。

1. 右键单击您的项目，然后选择“发布”。

2. 单击“发布”以发布到 Azure Web 应用。

完成此操作后，将在 Azure 经典管理门户中配置两个 Azure Active Directory 应用程序，一个用于 Visual Studio 中的调试环境，另一个用于 Azure 中发布的 Web 应用。在调试期间，将使用 Web.config 中的应用程序设置来使**调试**配置适用于 Azure Active Directory，发布配置（默认情况下，会发布**版本**配置）后，将上载转换的 Web.config，其中包含 Web.Release.config 中的应用程序设置更改。

如果你想要附加到已发布的 Web 应用调试器（必须上载的已发布的 Web 应用中的代码的调试符号），你可以创建的调试配置对于 Azure 调试，但使用的 Azure Active Directory 设置从 Web.Release.config 自己自定义 Web.config 转换（例如 Web.AzureDebug.config）克隆。这样，你可以跨不同的环境中维护静态配置。

##<a name="bkmk_crud"></a> 将业务线功能添加到示例应用程序

在本教程的本部分，你将学习如何基于示例应用程序生成所需的业务线功能。你将创建一个简单 CRUD 的工作项跟踪程序，类似于 TaskTracker 控制器，但使用标准的 CRUD 基架和设计模式。你还将使用包含的 Scripts\\AadPickerLibrary.js 从 Azure Active Directory 图形 API 丰富应用程序与数据。

5.	在 Models 文件夹中创建名为 WorkItem.cs 的新 [Code First](http://www.asp.net/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application) 模型，并将代码替换为以下代码：

		using System.ComponentModel.DataAnnotations;
		
		namespace WebApp_RoleClaims_DotNet.Models
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

6.	打开 DAL\\RoleClaimContext.cs 并添加突出显示的代码：
	<pre class="prettyprint">
	public class RoleClaimContext : DbContext<span>&#13;</span>
	{<span>&#13;</span>
	    public RoleClaimContext() : base("RoleClaimContext") { }<span>&#13;</span>

	    public DbSet&lt;Task> Tasks { get; set; }<span>&#13;</span>
	    <mark>public DbSet&lt;WorkItem> WorkItems { get; set; }</mark><span>&#13;</span>
	    public DbSet&lt;TokenCacheEntry> TokenCacheEntries { get; set; }<span>&#13;</span>
	}</pre>

7.	生成项目，以便能够通过 Visual Studio 中的基架逻辑访问你的新模型。

8.	将新的基架项 `WorkItemsController` 添加到 Controllers 文件夹。为此，请右键单击“控制器”，指向“添加”，然后选择“新的基架项”。

9.	选择“使用实体框架的包含视图的 MVC 5 控制器”并单击“添加”。

10.	选择刚创建的模型并单击“添加”。

	![](./media/web-sites-dotnet-lob-application-azure-ad/8-add-scaffolded-controller.png)

9.	打开 Controllers\\WorkItemsController.cs

11. 将突出显示的 [Authorize] 修饰添加到下面的相应操作。
	<pre class="prettyprint">
	...<span>&#13;</span>

    <mark>[Authorize(Roles = "Admin, Observer, Writer, Approver")]</mark><span>&#13;</span>
    public class WorkItemsController : Controller<span>&#13;</span>
    {<span>&#13;</span>
		...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer")]</mark><span>&#13;</span>
        public ActionResult Create()<span>&#13;</span>
        ...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer")]</mark><span>&#13;</span>
        public async Task&lt;ActionResult&gt; Create([Bind(Include = "ItemID,AssignedToID,AssignedToName,Description,Status")] WorkItem workItem)<span>&#13;</span>
        ...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer")]</mark><span>&#13;</span>
        public async Task&lt;ActionResult&gt; Edit(int? id)<span>&#13;</span>
        ...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer")]</mark><span>&#13;</span>
        public async Task&lt;ActionResult&gt; Edit([Bind(Include = "ItemID,AssignedToID,AssignedToName,Description,Status")] WorkItem workItem)<span>&#13;</span>
        ...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer, Approver")]</mark><span>&#13;</span>
        public async Task&lt;ActionResult&gt; Delete(int? id)<span>&#13;</span>
        ...<span>&#13;</span>

        <mark>[Authorize(Roles = "Admin, Writer, Approver")]</mark><span>&#13;</span>
        public async Task&lt;ActionResult&gt; DeleteConfirmed(int id)<span>&#13;</span>
        ...<span>&#13;</span>
	}</pre>

	> [AZURE.NOTE]你可能已注意到某些操作带有 <code>[ValidateAntiForgeryToken]</code> 修饰。由于存在 [Brock Allen](https://twitter.com/BrockLAllen) 在 [MVC 4、AntiForgeryToken 和声明](http://brockallen.com/2012/07/08/mvc-4-antiforgerytoken-and-claims/)中所述的行为，HTTP POST 可能无法完成防伪令牌验证，因为： + Azure Active Directory 不会发送 http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider，而默认情况下防伪令牌需要此项。+ 如果 Azure Active Directory 是与 AD FS 进行同步处理的目录，则默认情况下 AD FS 信任不发送 http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider 声明，不过你可以手动将 AD FS 配置为发送此声明。你将在下一步对此进行处理。

12.  在 App\_Start\\Startup.Auth.cs 中，将以下代码行添加到 `ConfigureAuth` 方法中。右键单击每个命名解析错误并修复错误。

		AntiForgeryConfig.UniqueClaimTypeIdentifier = ClaimTypes.NameIdentifier;
	
	`ClaimTypes.NameIdentifies` 指定 Azure Active Directory 提供的声明 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier`。既然你已注意了授权部分（严格来讲，这并不长），你可以将时间投入到操作的实际功能。

13.	在 Create() 和 Edit() 中添加以下代码，使某些变量可在后面的 JavaScript 中使用。右键单击每个命名解析错误并修复错误。

        ViewData["token"] = AcquireToken(ClaimsPrincipal.Current.FindFirst(Globals.ObjectIdClaimType).Value);
        ViewData["tenant"] = ConfigHelper.Tenant;

13.	`AcquireToken()` 方法尚未定义，现在请在 `WorkItemsController` 类中定义它。右键单击每个命名解析错误并修复错误。

        static string AcquireToken(string userObjectId)
        {
            ClientCredential cred = new ClientCredential(ConfigHelper.ClientId, ConfigHelper.AppKey);
            Claim tenantIdClaim = ClaimsPrincipal.Current.FindFirst(Globals.TenantIdClaimType);
            AuthenticationContext authContext = new AuthenticationContext(String.Format(CultureInfo.InvariantCulture, ConfigHelper.AadInstance, tenantIdClaim.Value), new TokenDbCache(userObjectId));
            AuthenticationResult result = authContext.AcquireTokenSilent(ConfigHelper.GraphResourceId, cred, new UserIdentifier(userObjectId, UserIdentifierType.UniqueId));
            return result.AccessToken;
        }
		
14.	在 Views\\WorkItems\\Create.cshtml（自动搭建基架的项）中，找到 `Html.BeginForm` 帮助器方法并对其进行如下修改：
	<pre class="prettyprint">@using (Html.BeginForm(<mark>"Create", "WorkItems", FormMethod.Post, new { id = "main-form" }</mark>))<span>&#13;</span>
	{<span>&#13;</span>
    @Html.AntiForgeryToken()<span>&#13;</span>

    &lt;div class="form-horizontal"><span>&#13;</span>
        &lt;h4>WorkItem&lt;/h4><span>&#13;</span>
        &lt;hr /><span>&#13;</span>
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })<span>&#13;</span>

        &lt;div class="form-group"><span>&#13;</span>
            &lt;div class="col-md-10"><span>&#13;</span>
                @Html.EditorFor(model => model.AssignedToID, new { htmlAttributes = new { @class = "form-control"<mark>, @type="hidden"</mark> } })<span>&#13;</span>
                @Html.ValidationMessageFor(model => model.AssignedToID, "", new { @class = "text-danger" })<span>&#13;</span>
            &lt;/div><span>&#13;</span>
        &lt;/div><span>&#13;</span>

        &lt;div class="form-group"><span>&#13;</span>
            @Html.LabelFor(model => model.AssignedToName, htmlAttributes: new { @class = "control-label col-md-2" })<span>&#13;</span>
            &lt;div class="col-md-10"><span>&#13;</span>
                @Html.EditorFor(model => model.AssignedToName, new { htmlAttributes = new { @class = "form-control" } })<span>&#13;</span>
                @Html.ValidationMessageFor(model => model.AssignedToName, "", new { @class = "text-danger" })<span>&#13;</span>
            &lt;/div><span>&#13;</span>
        &lt;/div><span>&#13;</span>

        &lt;div class="form-group"><span>&#13;</span>
            @Html.LabelFor(model => model.Description, htmlAttributes: new { @class = "control-label col-md-2" })<span>&#13;</span>
            &lt;div class="col-md-10"><span>&#13;</span>
                @Html.EditorFor(model => model.Description, new { htmlAttributes = new { @class = "form-control" } })<span>&#13;</span>
                @Html.ValidationMessageFor(model => model.Description, "", new { @class = "text-danger" })<span>&#13;</span>
            &lt;/div><span>&#13;</span>
        &lt;/div><span>&#13;</span>

        &lt;div class="form-group"><span>&#13;</span>
            @Html.LabelFor(model => model.Status, htmlAttributes: new { @class = "control-label col-md-2" })<span>&#13;</span>
            &lt;div class="col-md-10"><span>&#13;</span>
                @Html.EnumDropDownListFor(model => model.Status, htmlAttributes: new { @class = "form-control" })<span>&#13;</span>
                @Html.ValidationMessageFor(model => model.Status, "", new { @class = "text-danger" })<span>&#13;</span>
            &lt;/div><span>&#13;</span>
        &lt;/div><span>&#13;</span>

        &lt;div class="form-group"><span>&#13;</span>
            &lt;div class="col-md-offset-2 col-md-10"><span>&#13;</span>
                &lt;input type="submit" value="Create" class="btn btn-default" <mark>id="submit-button"</mark> /><span>&#13;</span>
            &lt;/div><span>&#13;</span>
        &lt;/div><span>&#13;</span>
    &lt;/div><span>&#13;</span>

    <mark>&lt;script><span>&#13;</span>
            // People/Group Picker Code<span>&#13;</span>
            var maxResultsPerPage = 14;<span>&#13;</span>
            var input = document.getElementById("AssignedToName");<span>&#13;</span>
            var token = "@ViewData["token"]";<span>&#13;</span>
            var tenant = "@ViewData["tenant"]";<span>&#13;</span>

            var picker = new AadPicker(maxResultsPerPage, input, token, tenant);<span>&#13;</span>

            // Submit the selected user/group to be asssigned.<span>&#13;</span>
            $("#submit-button").click({ picker: picker }, function () {<span>&#13;</span>
                if (!picker.Selected())<span>&#13;</span>
                    return;<span>&#13;</span>
                $("#main-form").get()[0].elements["AssignedToID"].value = picker.Selected().objectId;<span>&#13;</span>
            });<span>&#13;</span>
    &lt;/script></mark>}<span>&#13;</span>
    
    	</pre>
   
   
    在脚本中，AadPicker 对象将调用 [Azure Active Directory 图形 API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog) 来搜索与输入内容匹配的用户和组。

15. 打开[包管理器控制台](http://docs.nuget.org/Consume/Package-Manager-Console)并运行 **Enable-migrations-EnableAutomaticMigrations**。与你在将应用发布到 Azure 时选择的选项类似，当你在 Visual Studio 中调试应用时，此命令将帮助你在 [LocalDB](https://msdn.microsoft.com/zh-cn/library/hh510202.aspx) 中更新应用的数据库架构。

15. 现在，在 Visual Studio 调试器中运行应用程序，或者再次发布到 Azure Web 应用。以应用程序所有者的身份登录并导航到 `https://<webappname>.chinacloudsites.cn/WorkItems/Create`。现在你会发现，你可以从下拉列表中选择 Azure Active Directory 用户或组，或者键入一些内容来筛选列表。

	![](./media/web-sites-dotnet-lob-application-azure-ad/9-create-workitem.png)

16. 填写表单的其余部分并单击“创建”。~/WorkItems/Index 页现在将显示新建的工作项。你还会发现，在下面的屏幕截图中，我删除了 Views\\WorkItems\\Index.cshtml 中的 `AssignedToID` 列。

	![](./media/web-sites-dotnet-lob-application-azure-ad/10-workitem-index.png)

11.	现在，请对“编辑”视图进行类似的更改。在 Views\\WorkItems\\Edit.cshtml 中，更改与上一步中 Views\\WorkItems\\Create.cshtml 相同的 `Html.BeginForm` 帮助器方法（在上面突出显示的“Create”替换为“Edit”）。

就这么简单！

在 WorkItems 控制器中配置授权和不同操作的业务线功能后，你可以尝试以不同应用程序角色用户身份登录，以查看应用程序如何做出响应。

![](./media/web-sites-dotnet-lob-application-azure-ad/11-edit-unauthorized.png)

##<a name="bkmk_resources"></a> 其他资源

- [通过 SSL 和 Authorize 属性保护应用程序](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/#protect-the-application-with-ssl-and-the-authorize-attribute)
- [使用 Active Directory 在 Azure 中进行身份验证](/documentation/articles/web-sites-authentication-authorization/)
- [在 Azure 中创建使用 AD FS 身份验证的 .NET MVC Web 应用](/documentation/articles/web-sites-dotnet-lob-application-adfs/)
- [Azure Active Directory 示例和文档](https://github.com/AzureADSamples)
- [Vittorio Bertocci 的博客](http://blogs.msdn.com/b/vbertocci/)
- [将 VS2013 Web 项目从 WIF 迁移到 Katana](http://www.cloudidentity.com/blog/2014/09/15/MIGRATE-A-VS2013-WEB-PROJECT-FROM-WIF-TO-KATANA/)
- [Active Directory 与 Azure Active Directory 之间的相似之处](http://technet.microsoft.com/zh-cn/library/dn518177.aspx)
- [使用单一登录方案进行目录同步](http://technet.microsoft.com/zh-cn/library/dn441213.aspx)
- [Azure Active Directory 支持的令牌和声明类型](/documentation/articles/active-directory-token-and-claims/)
 

<!---HONumber=79-->