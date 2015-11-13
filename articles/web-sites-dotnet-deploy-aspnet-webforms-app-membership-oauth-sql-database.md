<properties 
	pageTitle="创建包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET Web 窗体应用并部署到 Azure 网站" 
	description="本教程说明如何生成整合了 SQL 数据库的安全 ASP.NET 4.5 Web 窗体 Web 应用程序并将其部署到 Azure。" 
	services="app-service\web" 
	documentationCenter=".net" 
	authors="Erikre" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="08/06/2015" 
	wacn.date="10/03/2015"/>


# 创建包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET Web 窗体应用并部署到 Azure 网站

##概述
本教程说明如何生成整合了 SQL 数据库的安全 ASP.NET 4.5 Web 窗体 Web 应用程序并将其部署到 Azure。

>[AZURE.NOTE]有关本教程中的 MVC 版本，请参阅[创建包含身份验证和 SQLDB 的 ASP.NET MVC 应用并将其部署到 Azure 网站](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。

你可以免费注册一个 Azure 帐户，而且，如果你还没有 Visual Studio 2013，则此 SDK 会自动安装 Visual Studio 2013 for Web Express。你可以免费开始针对 Azure 进行开发。

本教程假定你之前未使用过 Windows Azure。完成本教程之后，你将能够在云中启动并运行使用云数据库的 Web 应用程序。

学习内容：

- 如何创建 ASP.NET 4.5 Web 窗体项目并将其发布到 Azure 网站。
- 如何使用 OAuth 和 ASP.NET 成员资格保护你的应用程序。
- 如何为成员资格和应用程序数据使用单个数据库。
- 如何使用 Web 窗体基架创建网页以便修改数据。
- 如何使用新成员资格 API 添加用户和角色。
- 如何使用 SQL 数据库在 Azure 中存储数据。

你将生成一个简单的联系人列表 Web 应用程序，该应用程序基于 ASP.NET 4.5 Web 窗体构建并使用实体框架进行数据库访问。下图显示了用于对数据库进行读取和写入访问的 Web 窗体页：

![联系人 - 编辑页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms00.png)

>[AZURE.NOTE]若要完成本教程，您需要一个 Azure 帐户。如果您没有帐户，则可以<a href="/pricing/1rmb-trial/" target="_blank">注册获取试用版</a>。

本教程包含以下部分：

- [设置开发环境](#set-up-the-development-environment)
- [设置 Azure 环境](#Set-up-the-Azure-environment)
- [创建 ASP.NET Web 窗体应用程序](#Create-an-ASP.NET-Web-Forms-Application)
- [向应用程序添加数据库](#Add-a-Database-to-the-Application)
- [为项目启用 SSL](#Enable-SSL-for-the-Project)
- [添加 OAuth 2.0 提供程序](#Add-an-OAuth-2.0-Provider)
- [使用成员资格 API 来限制访问](#Use-the-Membership-API-to-Restrict-Access)
- [将包含数据库的应用程序部署到 Azure](#Deploy-the-Application-with-the-Database-to-Azure)
- [查看数据库](#Review-the-Database)
- [后续步骤](#Next-Steps)

##设置开发环境 
若要开始，请通过安装 Visual Studio 2013 和 Azure SDK for .NET 来设置开发环境。

1. 安装 [Visual Studio 2013](https://www.visualstudio.com/zh-cn/downloads)（如果尚未安装）。  
2. 安装 [Azure SDK for Visual Studio 2013](http://go.microsoft.com/fwlink/?linkid=324322&clcid=0x409)。本教程需要先安装 Visual Studio 2013，然后再安装 Azure SDK for Visual Studio 2013。根据计算机上已有 SDK 依赖项数量的不同，安装 SDK 可能耗时较长，从几分钟到半小时或更长时间不等。  

3. 如果系统提示你是要运行还是保存可执行安装文件，请单击“运行”。
4. 在“Web 平台安装程序”窗口中，单击“安装”，然后进行安装。![Web 平台安装程序](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/Intro-SecureWebForms-01.png)  

      如果你已安装 SDK，则会安装 0 个项。要安装的项数将显示在“Web 平台安装程序”窗口的左下角。

5. 如果您还没有 **Visual Studio Update 2**，请下载并安装 **<!--[-->Visual Studio 2013 Update 2<!--](http://www.microsoft.com/download/details.aspx?id=42666)-->** 或更高版本。

	>[AZURE.NOTE]只有安装了 Visual Studio 2013 Update 2 或更高版本才能使用 Goggle OAuth 2.0 以及在本地使用 SSL，而不会显示任何警告。此外，你需要通过 Update 2 来使用 Web 窗体基架。

安装完成后，你便做好了开发准备工作。

##设置 Azure 环境
在本部分中，你将通过在 Azure 中创建 Azure 和 SQL 数据库来设置 Azure 环境。

###在 Azure 中创建 Web 应用和 SQL 数据库 
在本教程中，你的 Web 应用将在共享宿主环境中运行，这意味着它将在与 Azure 网站中与其他 Web 应用共享的虚拟机 (VM) 上运行。共享宿主环境是一种在云中开始工作的低成本方式。稍后，如果你的 Web 流量增加，则应用程序可进行扩展，通过在专用 VM 上运行来满足需要。如果你需要一个更复杂的体系结构，则可迁移到 Azure 云服务。云服务在你可根据自己的需求进行配置的专用 VM 上运行。

Azure SQL 数据库是根据 SQL Server 技术构建的基于云的关系数据库服务。可以与 SQL Server 一起使用的工具和应用程序也可用于 SQL 数据库。

1. 在 [ Azure 管理门户](https://manage.windowsazure.cn)中，单击左侧选项卡中的“Web Apps”，然后单击“新建”。![Web 平台安装程序](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/Intro-SecureWebForms-02.png)
2. 单击“Web App”，然后单击“自定义创建”。![自定义创建](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/Intro-SecureWebForms-03.png) “新建 Web App - 自定义创建”向导将打开。  

3. 在该向导的“创建 Web App”步骤中，在“URL”框中输入将用作你的应用程序的唯一 URL 的字符串。完整的 URL 将包含你在此处输入的内容和你在文本框旁边看到的后缀。图中显示了一个 URL，但可能已有人使用了该 URL，因此**必须另外选择一个 URL**。![联系人 - 创建新网站](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/Intro-SecureWebForms-04.png)
4. 在“Web 托管计划”下拉列表中，选择离你最近的区域。此设置指定你的 VM 将在哪个数据中心运行。
5. 在“数据库”下拉列表中，选择“创建免费的 20 MB SQL 数据库”。
6. 在“数据库连接字符串名称”框中，保留默认值 *DefaultConnection*。
7. 单击框底部的箭头。该向导将前进到“指定数据库设置”步骤。
8. 在“名称”框中，输入 *`ContactDB`*。![数据库设置](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/Intro-SecureWebForms-05.png)  
9. 在“服务器”框中，选择“新建 SQL 数据库”服务器。或者，如果你之前创建了 SQL Server 数据库，则可从下拉列表控件中选择 SQL Server。
10. 将“区域”设置为创建 Web 应用的同一区域。
11. 输入管理员“登录名”和“密码”。如果你选择了“新建 SQL 数据库服务器”，则在此处不要输入现有名称和密码。你应输入新的名称和密码，你现在定义的名称和密码将在你以后访问数据库时使用。如果你选择了之前创建的 SQL Server，系统将提示你输入之前创建的 SQL Server 帐户名称的密码。本教程将不选中“高级”框。
12. 单击对话框右下角的复选标记以指示你已完成操作。

“Azure 管理门户”返回到“Web Apps”页面，并且“状态”列显示正在创建网站。稍后（通常不到一分钟），“状态”列会显示已成功创建网站。在左侧的导航栏中，你的帐户中拥有的网站的数量将会显示在“Web 应用”图标旁边，而数据库的数量将会显示在“SQL 数据库”图标旁边。
##创建 ASP.NET Web 窗体应用程序 
你已经创建了一个 Web 应用，但其中还没有内容。下一步将创建要发布到 Azure 的 Visual Studio Web 应用程序。
###创建项目 
1. 在 Visual Studio 中，从“文件”菜单选择“新建项目”。![文件菜单 - 新建项目](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms01.png)  
2. 在左侧依次选择“模板”、“Visual C#”、“Web”模板组。 
3. 在中心列中选择“ASP.NET Web 应用程序”模板。
4. 将你的项目命名为 *ContactManager* 并单击“确定”。![新建项目对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms02.png)  

      在本系列教程中，项目名称为 **ContactManager**。建议使用与此完全相同的项目名称，以便整个系列教程中提供的代码按预期运行。

5. 在“新建 ASP.NET 项目”对话框中，选择“Web 窗体”模板。取消选中“在云中托管”（如果已选中），然后单击“确定”。![“新建 ASP.NET 项目”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms03.png) 随后将创建 Web 窗体应用程序。
###更新母版页
在 ASP.NET Web 窗体中，母版页允许你在应用程序中创建一致的页面布局。可以使用单个母版页定义应用程序中所有页（或一组页）的外观和标准行为。然后，你可以创建包含想要显示的内容的各个内容页。当用户请求内容页时，ASP.NET 会将内容页与母版页合并，以生成将母版面布局与内容页中的内容相结合的输出。新站点需要更新应用程序名称和链接。该链接将指向显示联系人详细信息的页。若要进行这些更改，请修改母版页上的 HTML。

1. 在“解决方案资源管理器”中，找到并打开“Site.Master”页。
2. 如果该页在“设计”视图中，请切换到“源”视图。
3. 通过修改或添加标记以更新母版页，从而页面的标记显示如下：

<pre class="prettyprint">
&lt;%@ Master Language="C#" AutoEventWireup="true" CodeBehind="Site.master.cs" Inherits="ContactManager.SiteMaster" %>

&lt;!DOCTYPE html>

&lt;html lang="en">
&lt;head runat="server">
    &lt;meta charset="utf-8" />
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" />
    &lt;title>&lt;%: Page.Title %> - <mark>Contact Manager</mark>&lt;/title>

    &lt;asp:PlaceHolder runat="server">
        &lt;%: Scripts.Render("~/bundles/modernizr") %>
    &lt;/asp:PlaceHolder>
    &lt;webopt:bundlereference runat="server" path="~/Content/css" />
    &lt;link href="~/favicon.ico" rel="shortcut icon" type="image/x-icon" />

&lt;/head>
&lt;body>
    &lt;form runat="server">
        &lt;asp:ScriptManager runat="server">
            &lt;Scripts>
                &lt;%--To learn more about bundling scripts in ScriptManager see http://go.microsoft.com/fwlink/?LinkID=301884 --%>
                &lt;%--Framework Scripts--%>
                &lt;asp:ScriptReference Name="MsAjaxBundle" />
                &lt;asp:ScriptReference Name="jquery" />
                &lt;asp:ScriptReference Name="bootstrap" />
                &lt;asp:ScriptReference Name="respond" />
                &lt;asp:ScriptReference Name="WebForms.js" Assembly="System.Web" Path="~/Scripts/WebForms/WebForms.js" />
                &lt;asp:ScriptReference Name="WebUIValidation.js" Assembly="System.Web" Path="~/Scripts/WebForms/WebUIValidation.js" />
                &lt;asp:ScriptReference Name="MenuStandards.js" Assembly="System.Web" Path="~/Scripts/WebForms/MenuStandards.js" />
                &lt;asp:ScriptReference Name="GridView.js" Assembly="System.Web" Path="~/Scripts/WebForms/GridView.js" />
                &lt;asp:ScriptReference Name="DetailsView.js" Assembly="System.Web" Path="~/Scripts/WebForms/DetailsView.js" />
                &lt;asp:ScriptReference Name="TreeView.js" Assembly="System.Web" Path="~/Scripts/WebForms/TreeView.js" />
                &lt;asp:ScriptReference Name="WebParts.js" Assembly="System.Web" Path="~/Scripts/WebForms/WebParts.js" />
                &lt;asp:ScriptReference Name="Focus.js" Assembly="System.Web" Path="~/Scripts/WebForms/Focus.js" />
                &lt;asp:ScriptReference Name="WebFormsBundle" />
                &lt;%--Site Scripts--%>
            &lt;/Scripts>
        &lt;/asp:ScriptManager>

        &lt;div class="navbar navbar-inverse navbar-fixed-top">
            &lt;div class="container">
                &lt;div class="navbar-header">
                    &lt;button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                        &lt;span class="icon-bar">&lt;/span>
                        &lt;span class="icon-bar">&lt;/span>
                        &lt;span class="icon-bar">&lt;/span>
                    &lt;/button>
                    &lt;a class="navbar-brand" runat="server" <mark>id="ContactDemoLink"</mark> href="~/<mark>Contacts/Default.aspx</mark>"><mark>Contact Demo</mark>&lt;/a>
                &lt;/div>
                &lt;div class="navbar-collapse collapse">
                    &lt;ul class="nav navbar-nav">
                        &lt;li>&lt;a runat="server" href="~/">主页&lt;/a>&lt;/li>
                        &lt;li>&lt;a runat="server" href="~/About">关于&lt;/a>&lt;/li>
                        &lt;li>&lt;a runat="server" href="~/Contact">联系人&lt;/a>&lt;/li>
                    &lt;/ul>
                    &lt;asp:LoginView runat="server" ViewStateMode="Disabled">
                        &lt;AnonymousTemplate>
                            &lt;ul class="nav navbar-nav navbar-right">
                                &lt;li>&lt;a runat="server" href="~/Account/Register">注册&lt;/a>&lt;/li>
                                &lt;li>&lt;a runat="server" href="~/Account/Login">登录&lt;/a>&lt;/li>
                            &lt;/ul>
                        &lt;/AnonymousTemplate>
                        &lt;LoggedInTemplate>
                            &lt;ul class="nav navbar-nav navbar-right">
                                &lt;li>&lt;a runat="server" href="~/Account/Manage" title="Manage your account">你好，&lt;%: Context.User.Identity.GetUserName()  %> !&lt;/a>&lt;/li>
                                &lt;li>
                                    &lt;asp:LoginStatus runat="server" LogoutAction="Redirect" LogoutText="Log off" LogoutPageUrl="~/" OnLoggingOut="Unnamed_LoggingOut" />
                                &lt;/li>
                            &lt;/ul>
                        &lt;/LoggedInTemplate>
                    &lt;/asp:LoginView>
                &lt;/div>
            &lt;/div>
        &lt;/div>
        &lt;div class="container body-content">
            &lt;asp:ContentPlaceHolder ID="MainContent" runat="server">
            &lt;/asp:ContentPlaceHolder>
            &lt;hr />
            &lt;footer>
                &lt;p>复制(&amp;C); &lt;%: DateTime.Now.Year %> - <mark>联系人管理器</mark>&lt;/p>
            &lt;/footer>
        &lt;/div>
    &lt;/form>
&lt;/body>
&lt;/html>
</pre>

在本教程的后面，你将要添加 Web 窗体基架。基架将创建上述“联系人演示”链接引用的页。
###在本地运行应用程序 
 
1. 在“解决方案资源管理器”中，右键单击“Default.aspx”页并选择“设为起始页”。 
2. 接下来，按“CTRL+F5”运行应用程序。应用程序的默认页会显示在默认浏览器窗口中。![联系人 - 创建新网站](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms04.png)  

这就是你创建将要部署到 Azure 的应用程序目前所需的全部操作。稍后，你将要添加数据库功能以及所需的页来显示和编辑联系人数据。
###将应用程序部署到 Azure
创建并在本地运行应用程序后，可以将应用程序部署到 Azure。

1. 在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。![选择“发布”](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms05.png) 此时将显示“发布 Web”对话框。  

2. 在“发布 Web”对话框的“配置文件”选项卡中，单击“Azure Web 应用”。
	  
3. 如果你尚未登录，请在“选择现有 Web 应用”对话框中单击“登录”按钮。完成登录后，选择你在本教程第一部分中创建的 Web 应用。单击“确定”继续。![选择“现有网站”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms07.png) Visual Studio 将下载发布设置。
4. 在“发布 Web”对话框中，单击“发布”。![“发布 Web”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms08.png) 你将在 Visual Studio 中的“Web 发布活动”窗口中看到总体发布状态：![Web 发布活动](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms09.png)  

你创建的应用程序现在在云中运行。下次从 Visual Studio 部署该应用程序时，仅会部署已更改（或新的）文件。![浏览器中的应用](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms10.png)

>[AZURE.NOTE]如果在发布已建立的 Web 应用时遇到错误，则可以在添加新的文件之前清除该位置。再次发布应用程序，但这次请在“发布 Web”对话框中选择“设置”选项卡。然后，将配置设置为“调试”，并选择“删除目标位置的其他文件”选项。选择“发布”以再次部署应用程序。![“发布 Web”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms11.png)

##向应用程序添加数据库 
接下来，你将更新 Web 窗体应用程序以添加显示和更新联系人以及在默认数据库中存储数据的功能。当你创建 Web 窗体项目时，默认情况下也会创建数据库。应用程序将使用 Entity Framework 访问数据库并读取和更新数据库中的数据。
###添加数据模型类 
首先，使用代码创建一个简单的数据模型。此数据模型将包含在名为 `Contacts` 的类中。选择 `Contacts` 类名的目的是避免与 Web 窗体模板创建的 Contact.aspx.cs 文件中包含的 `Contact` 类名冲突。

1. 在“解决方案资源管理器”中，右键单击“模型”文件夹，然后依次选择“添加”、“类”。![选择类](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms12.png) 此时将显示“添加新项”对话框。  

2. 将此新类命名为 *Contacts.cs*。![“添加新项”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms13.png)
3. 将默认代码替换为以下代码：  
	<pre class="prettyprint">
using System.ComponentModel.DataAnnotations;
using System.Globalization;

namespace ContactManager.Models
{
    public class Contacts
    {
        [ScaffoldColumn(false)]
        [Key]
        public int ContactId { get; set; }
        public string Name { get; set; }
        public string Address { get; set; }
        public string City { get; set; }
        public string State { get; set; }
        public string Zip { get; set; }
        [DataType(DataType.EmailAddress)]
        public string Email { get; set; }
    }
}
</pre>

“Contacts”类定义你将为每个联系人存储的数据以及数据库需要的主键 (`ContactID`)。“Contacts”类表示要显示的联系人数据。Contacts 对象的每个实例将对应于关系数据库表中的某行，Contacts 类的每个属性将映射到关系数据库表中的某列。在本教程的后面，你将要查看数据库中包含的联系人数据。

###Web 窗体基架 
前面你已创建“Contacts”模型类。现在，你可以使用 Web 窗体基架以生成处理此数据时使用的“列表”、“插入”、“编辑”和“删除”页。Web Forms Scaffolder 使用实体框架、引导和动态数据。默认情况下，在使用 Visual Studio 2013 时，Web Forms Scaffolder 将以扩展形式安装到项目中，作为项目模板的一部分。

可以执行以下步骤来使用 Web Forms Scaffolder。

1. 在 Visual Studio 中，从菜单栏中依次选择“工具”、“扩展和更新”。此时将显示“扩展和更新”对话框。
2. 从对话框的左窗格中，依次选择“联机”、“Visual Studio 库”、“工具”和“基架”。
3. 如果没有在列表中看到“Web 窗体基架”，请在对话框右侧的搜索框中输入“Web 窗体基架”。  
4. 如果尚未安装 Web 窗体基架，请选择“下载”以下载并安装“Web 窗体基架”。根据需要重新启动 Visual Studio。在提示时，请务必保存对项目所做的更改。![“扩展和更新”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/ExtensionsAndUpdatesDB.png)  
5. 生成项目 (**Ctrl+Shift+B**).在使用基架机制之前，必须生成项目。  
6. 在“解决方案资源管理器”中，右键单击“项目”，然后依次选择“添加”、“新建基架项”。此时将显示“添加基架”对话框。
7. 从左窗格中选择“Web 窗体”，并从中央窗格中选择“使用实体框架的 Web 窗体页”。然后单击“添加”。![“添加基架”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms13a.png) 此时将显示“添加 Web 窗体页”对话框。  

8. 在“添加 Web 窗体页”对话框中，将“模型类”设置为 `Contacts (ContactManager.Models)`。将“数据上下文类”设置为 `ApplicationDbContext (ContactManager.Models)`。然后单击“添加”。![“添加 Web 窗体页”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms13b.png)

Web 窗体基架添加的新文件夹包含*Default.aspx*、*Delete.aspx*、*Edit.aspx* 和 *Insert.aspx* 页。此外，Web 窗体基架还会创建 *DynamicData* 文件夹，其中包含 *EntityTemplates* 文件夹和 *FieldTemplates* 文件夹。`ApplicationDbContext` 将用于成员资格数据库和联系人数据。

###将应用程序配置为使用数据模型 
接下来的任务是启用 Code First 迁移功能以便基于你创建的数据模型创建数据库。此外，你将要添加示例数据和数据初始值设定项。

1. 在“工具”菜单中，依次选择“NuGet Package Manager”和“Package Manager Console”。![“添加 Web 窗体页”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms13c.png)  
2. 在“Package Manager Console”窗口中，输入以下命令：  
	<pre class="prettyprint">
enable-migrations
</pre>Enable-migrations 命令将创建一个 *Migrations* 文件夹，并在该文件夹中放入一个可编辑以对数据库进行种子设定并配置数据迁移的 *Configuration.cs* 文件。  
3. 在“Package Manager Console”窗口中，输入以下命令：  
	<pre class="prettyprint">
add-migration Initial
</pre>`add-migration Initial` 命令将在创建数据库的 *Migrations* 文件夹中生成一个名为 <date_stamp>Initial 的文件。第一个参数 (Initial) 是任意参数并将用于创建文件名称。你可以在“解决方案资源管理器”中查看新的类文件。在 `Initial`中，`Up` 方法用于创建 `Contact`，而 `Down` 方法（在你想要返回到以前的状态时使用）用于删除该表。  
4. 打开 *Migrations\\Configuration.cs* 文件。 
5. 添加以下命名空间：  
	<pre class="prettyprint">
using ContactManager.Models;
</pre>
6. 将 `Seed`方法替换为以下代码：  
	<pre class="prettyprint">
protected override void Seed(ContactManager.Models.ApplicationDbContext context)
{
    context.Contacts.AddOrUpdate(p => p.Name,
       new Contacts
       {
           ContactId = 1,
           Name = "Ivan Irons",
           Address = "One Microsoft Way",
           City = "Redmond",
           State = "WA",
           Zip = "10999",
           Email = "ivani@wideworldimporters.com",
       },
       new Contacts
        {
            ContactId = 2,
            Name = "Brent Scholl",
            Address = "5678 1st Ave W",
            City = "Redmond",
            State = "WA",
            Zip = "10999",
            Email = "brents@wideworldimporters.com",
        },
        new Contacts
        {
            ContactId = 3,
            Name = "Terrell Bettis",
            Address = "9012 State St",
            City = "Redmond",
            State = "WA",
            Zip = "10999",
            Email = "terrellb@wideworldimporters.com",
        },
        new Contacts
        {
            ContactId = 4,
            Name = "Jo Cooper",
            Address = "3456 Maple St",
            City = "Redmond",
            State = "WA",
            Zip = "10999",
            Email = "joc@wideworldimporters.com",
        },
        new Contacts
        {
            ContactId = 5,
            Name = "Ines Burnett",
            Address = "7890 2nd Ave E",
            City = "Redmond",
            State = "WA",
            Zip = "10999",
            Email = "inesb@wideworldimporters.com",
        }
        );
}
</pre>
此代码将用联系信息初始化数据库或对其进行种子设定。有关对数据库进行种子设定的更多信息，请参阅[对 Entity Framework (EF) 数据库进行种子设定和调试](http://blogs.msdn.com/b/rickandy/archive/2013/02/12/seeding-and-debugging-entity-framework-ef-dbs.aspx)。  
7. 在“Package Manager Console”中输入以下命令：  
	<pre class="prettyprint">
update-database
</pre>
`update-database`用于运行将创建数据库的初始迁移。默认情况下，将以 SQL Server Express LocalDB 数据库的形式创建数据库。![Package Manager Console](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms13d.png)

###在本地运行应用程序并显示数据 
现在，请运行应用程序，以了解如何查看联系人。

1. 首先，请生成项目（“Ctrl+Shift+B”）。  
2. 按“Ctrl+F5”运行应用程序。浏览器将打开并显示 *Default.aspx* 页。
3. 选择页面顶部的“联系人演示”链接以显示“联系人列表”页。![联系人列表页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms17.png)  

##为项目启用 SSL 
安全套接字层 (SSL) 是一种协议，定义为允许 Web 服务器与 Web 客户端通过使用加密以更安全的方式通信。如果不使用 SSL，在客户端与服务器之间发送数据时，对网络具有实际访问权限的任何人都可以探查数据包。此外，通过一般 HTTP 进行的几种常见身份验证方案也是不安全的。尤其是，基本身份验证和窗体身份验证会发送未加密的凭据。为确保安全，这些身份验证方案必须使用 SSL。

1. 在“解决方案资源管理器”中，单击“ContactManager”项目，然后按**F4**显示“属性”窗口。 
2. 将“已启用 SSL”更改为 `true`。 
3. 复制 **SSL URL** 以便稍后使用。SSL URL 将为 `https://localhost:44300/`，除非你之前已创建 SSL Web 应用（如下所示）。![项目属性](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms18.png)  
4. 在“解决方案资源管理器”中，右键单击“Contact Manager”项目，然后单击“属性”。
5. 在左侧选项卡中，单击“Web”。
6. 将“项目 URL”更改为使用前面保存的“SSL URL”。![项目 Web 属性](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms19.png)  
7. 按“CTRL+S”保存该页。
8. 按“Ctrl+F5”运行应用程序。Visual Studio 将显示一个选项用于避免 SSL 警告。  
9. 单击“是”以信任 IIS Express SSL 证书并继续。![IIS Express SSL 证书详细信息](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms20.png)此时将显示一条安全警告。  

10. 单击“是”将证书安装到本地主机。![“安全警告”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21.png) 此时将显示浏览器窗口。

你可以在本地使用 SSL 轻松测试 Web 应用程序。



##添加 OAuth 2.0 提供程序 
ASP.NET Web 窗体为成员资格和身份验证提供了增强的选项。这些增强功能包括 OAuth。OAuth 是一种开放协议，允许以一种简单而标准的方法从 Web、移动和桌面应用程序进行安全授权。ASP.NET MVC Internet 模板使用 OAuth 公开将 Facebook、Twitter、Google 和 Microsoft 作为身份验证提供程序。虽然本教程仅使用 Google 作为身份验证提供程序，但你可轻松修改代码以使用任何提供程序。实施其他提供程序的步骤与你将在本教程中看到的步骤非常类似。

除了身份验证外，本教程还将使用角色实施授权。只有你添加到 `canEdit` 角色中的用户将能更改数据（创建、编辑或删除联系人）。

可以执行以下步骤来添加 Google 身份验证提供程序。

1. 打开 *App_Start\\Startup.Auth.cs* 文件。 
2. 从 `app.UseGoogleAuthentication()` 方法中删除注释字符，使该方法如下所示：  
	<pre class="prettyprint">
            app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
            {
                ClientId = "",
                ClientSecret = ""
            });
</pre>
3. 导航到 [Google Developers Console](https://console.developers.google.com/)。你还需要使用 Google 开发人员电子邮件帐户 (gmail.com） 登录。如果你没有 Google 帐户，请选择“创建帐户”链接。接下来，你将看到**Google Developers Console**。![Google Developers Console](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21a.png)  

4. 单击“创建项目”按钮，然后输入项目名称和 ID（可以使用默认值）。然后，单击“协议复选框”和“创建”按钮。![Google - 新建项目](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21b.png) 几秒钟后，将会创建新项目，并且浏览器将显示新项目页。
5. 在左侧选项卡中，单击“API 和身份验证”，然后单击“凭据”。
6. 单击“OAuth”下的“创建新客户端 ID”。此时将显示“创建客户端 ID”对话框。![Google - 创建客户端 ID](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21c.png)  
7. 在“创建客户端 ID”对话框中，保留默认“Web 应用程序”作为应用程序类型。  
8. 将“授权的 JavaScript 来源”设置为你之前在本教程中使用的 SSL URL（****https://localhost:44300/**除非你已创建其他 SSL 项目）。此 URL 是应用程序的来源。对于此示例，你只需输入 localhost 测试 URL。但是，你可以针对 localhost 和生产环境输入多个 URL。

9. 将“授权的重定向 URI”设置为：
	<pre class="prettyprint">  
https://localhost:44300/signin-google  
</pre>此值是 ASP.NET OAuth 用户与 Google OAuth 服务器通信时使用的 URI。请记住上面使用的 SSL URL（****https://localhost:44300/**除非你已创建其他 SSL 项目）。
10. 单击“创建客户端 ID”按钮。
11. 在 Visual Studio 中，通过复制 **AppId** 和“应用密码”并粘贴到方法中，来更新 *Startup.Auth.cs* 页的 `UseGoogleAuthentication` 方法。下面显示的 **AppId** 和“应用密码”值是示例，它们不可用。  
	<pre class="prettyprint">  
using System;
using Microsoft.AspNet.Identity;
using Microsoft.AspNet.Identity.EntityFramework;
using Microsoft.AspNet.Identity.Owin;
using Microsoft.Owin;
using Microsoft.Owin.Security.Cookies;
using Microsoft.Owin.Security.DataProtection;
using Microsoft.Owin.Security.Google;
using Owin;
using ContactManager.Models;

namespace ContactManager
{
    public partial class Startup {

        //  http://go.microsoft.com/fwlink/?LinkId=301883
        public void ConfigureAuth(IAppBuilder app)
        {
            // Configure the db context and user manager to use a single instance per request
            app.CreatePerOwinContext(ApplicationDbContext.Create);
            app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);

            // Enable the application to use a cookie to store information for the signed in user
            // and to use a cookie to temporarily store information about a user logging in with a third party login provider
            // Configure the sign in cookie
            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                LoginPath = new PathString("/Account/Login"),
                Provider = new CookieAuthenticationProvider
                {
                    OnValidateIdentity = SecurityStampValidator.OnValidateIdentity&lt;ApplicationUserManager, ApplicationUser>(
                        validateInterval: TimeSpan.FromMinutes(20),
                        regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager))
                }
            });
            // Use a cookie to temporarily store information about a user logging in with a third party login provider
            app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);

            // Uncomment the following lines to enable logging in with third party login providers
            //app.UseMicrosoftAccountAuthentication(
            //    clientId: "",
            //    clientSecret: "");

            //app.UseTwitterAuthentication(
            //   consumerKey: "",
            //   consumerSecret: "");

            //app.UseFacebookAuthentication(
            //   appId: "",
            //   appSecret: "");

            app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
            {
                ClientId = "<mark>000000000000.apps.googleusercontent.com</mark>",
                ClientSecret = "<mark>00000000000</mark>"
            });
        }
    }
}
</pre>
12. 按“Ctrl + F5”构建并运行应用程序。单击“登录”链接。
13. 在“使用其他服务进行登录”下，单击“Google”。![Log in](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21d.png)  
14. 如果需要输入你的凭据，你将被重定向到 google 站点，你可以在这里输入你的凭据。![Google - 登录](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21e.png)  
15. 输入你的凭据后，系统将提示你刚创建的 Web 应用程序，以便授予其权限：![项目默认服务帐户](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21f.png)  
16. 单击“接受”。现在，你将被重定向回到“ContactManager”应用程序的“注册”页，你可以在该页中注册 Google 帐户。![使用你的 Google 帐户注册](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms21g.png)  
17. 你可以选择更改 Gmail 帐户使用的本地电子邮件注册名称，但是，你通常会保留默认电子邮件别名（即，用于身份验证的名称）。单击“登录”。

##使用成员资格 API 来限制访问 
ASP.NET 标识是在生成 ASP.NET Web 应用程序时用于身份验证的成员资格系统。它可以轻松地将特定于用户的配置文件数据与应用程序数据相集成。此外，ASP.NET 标识允许你为应用程序中的用户配置文件选择持久模型。你可以将数据存储在 SQL Server 数据库或其他数据存储中，包括 Azure 存储表等 *NoSQL* 数据存储。

通过使用默认的 ASP.NET Web 窗体模板，你可以获得在运行应用程序时立即可用的内置成员资格功能。你将使用 ASP.NET 标识添加管理员角色并将用户分配给该角色。然后，你将学习如何限制对管理文件夹和该文件夹中用于修改联系人数据的页面的访问。

###添加管理员 
通过 ASP.NET 标识，你可以使用代码添加管理员角色并将用户分配给该角色。

1. 在“解决方案资源管理器”，打开 *Migrations* 文件夹中的 *Configuration.cs* 文件。
2. 在 `ContactManger.Migrations` 命名空间中添加以下 `using` 语句：  
	<pre class="prettyprint">
using Microsoft.AspNet.Identity;
using Microsoft.AspNet.Identity.EntityFramework;
</pre>
3. 将以下 `AddUserAndRole` 方法添加到 `Seed` 方法后面的 `Configuration` 类：  
	<pre class="prettyprint">
    public void AddUserAndRole(ContactManager.Models.ApplicationDbContext context)
    {
        IdentityResult IdRoleResult;
        IdentityResult IdUserResult;

        var roleStore = new RoleStore&lt;IdentityRole>(context);
        var roleMgr = new RoleManager&lt;IdentityRole>(roleStore);

        if (!roleMgr.RoleExists("canEdit"))
        {
            IdRoleResult = roleMgr.Create(new IdentityRole { Name = "canEdit" });
        }

        //var userStore = new UserStore&lt;ApplicationUser>(context);
        //var userMgr = new UserManager&lt;ApplicationUser>(userStore);
        var userMgr = new UserManager&lt;ApplicationUser>(new UserStore&lt;ApplicationUser>(context));

        var appUser = new ApplicationUser
        {
            UserName = "canEditUser@wideworldimporters.com",
            Email = "canEditUser@wideworldimporters.com"
        };
        IdUserResult = userMgr.Create(appUser, "Pa$$word1");

        if (!userMgr.IsInRole(userMgr.FindByEmail("canEditUser@wideworldimporters.com").Id, "canEdit"))
        {
          //  IdUserResult = userMgr.AddToRole(appUser.Id, "canEdit");
            IdUserResult = userMgr.AddToRole(userMgr.FindByEmail("canEditUser@wideworldimporters.com").Id, "canEdit");
        }
    }
</pre>
4. 从 `Seed` 方法的开头添加对 `AddUserAndRole` 方法的调用。请注意，这里只显示了 `Seed` 方法的开头。  
	<pre class="prettyprint">
    protected override void Seed(ContactManager.Models.ApplicationDbContext context)
    {
        <mark>AddUserAndRole(context);</mark>
</pre>
5. 保存所有更改后，在“Package Manager Console”中运行以下命令：  
	<pre class="prettyprint">
Update-Database
</pre>此代码会创建名为 `canEdit` 的新角色，创建电子邮件为 canEditUser@wideworldimporters.com 的新本地用户。然后，该代码会将添加 canEditUser@wideworldimporters.com 到 `canEdit` 角色。有关详细信息，请参阅 [ASP.NET 标识](http://www.asp.net/identity)资源页。

###限制对管理文件夹的访问 
**ContactManager** 示例应用程序允许匿名用户和已登录用户查看联系人。但是，在完成本部分后，只有分配给“canEdit”角色的已登录用户才能修改联系人。

你将要创建一个名为 *Admin* 的文件夹，只有分配给“canEdit”角色的用户才可以访问该文件夹。

1. 在“解决方案资源管理器”中，将一个子文件夹添加到 *Contacts* 文件夹，并命名新的子文件夹为 *Admin*。
2. 将以下文件从 *Contacts* 文件夹移动到 *Contacts/Admin* 文件夹：  
	- *Delete.aspx * 和 * Delete.aspx.cs*
	- *Edit.aspx * 和 * Edit.aspx.cs*
	- *Insert.aspx * 和 * Insert.aspx.cs*
3. 通过在链接到 *Insert.aspx*、*Edit.aspx* 和 *Delete.aspx* 的页面引用之前添加“Admin/”更新 *Contacts/Default.aspx* 中的链接引用：  
	<pre class="prettyprint">
&lt;%@ Page Title="ContactsList" Language="C#" MasterPageFile="~/Site.Master" CodeBehind="Default.aspx.cs" Inherits="ContactManager.Contacts.Default" ViewStateMode="Disabled" %>
&lt;%@ Register TagPrefix="FriendlyUrls" Namespace="Microsoft.AspNet.FriendlyUrls" %>

&lt;asp:Content runat="server" ContentPlaceHolderID="MainContent">
    &lt;h2>联系人列表&lt;/h2>
    &lt;p>
        &lt;asp:HyperLink runat="server" NavigateUrl="<mark>Admin/</mark>Insert.aspx" Text="Create new" />
    &lt;/p>
    &lt;div>
        &lt;asp:ListView runat="server"
            DataKeyNames="ContactId" ItemType="ContactManager.Models.Contacts"
            AutoGenerateColumns="false"
            AllowPaging="true" AllowSorting="true"
            SelectMethod="GetData">
            &lt;EmptyDataTemplate>
                未找到联系人条目
            &lt;/EmptyDataTemplate>
            &lt;LayoutTemplate>
                &lt;table class="table">
                    &lt;thead>
                        &lt;tr>
                            &lt;th>名称&lt;/th>
                            &lt;th>地址&lt;/th>
                            &lt;th>城市&lt;/th>
                            &lt;th>省/市/自治区&lt;/th>
                            &lt;th>邮政编码&lt;/th>
                            &lt;th>电子邮件&lt;/th>
                            &lt;th>&amp;nbsp;&lt;/th>
                        &lt;/tr>
                    &lt;/thead>
                    &lt;tbody>
                        &lt;tr runat="server" id="itemPlaceholder" />
                    &lt;/tbody>
                &lt;/table>
            &lt;/LayoutTemplate>
            &lt;ItemTemplate>
                &lt;tr>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="Name" ID="Name" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="Address" ID="Address" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="City" ID="City" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="State" ID="State" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="Zip" ID="Zip" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;asp:DynamicControl runat="server" DataField="Email" ID="Email" Mode="ReadOnly" />
                    &lt;/td>
                    &lt;td>
                        &lt;a href="<mark>Admin/</mark>Edit.aspx?ContactId=&lt;%#: Item.ContactId%>">Edit&lt;/a> | 
                        &lt;a href="<mark>Admin/</mark>Delete.aspx?ContactId=&lt;%#: Item.ContactId%>">Delete&lt;/a>
                    &lt;/td>
                &lt;/tr>
            &lt;/ItemTemplate>
        &lt;/asp:ListView>
    &lt;/div>
&lt;/asp:Content>
</pre>
4. 对于以下三个文件，将 `Response.Redirect("Default.aspx")` 代码的六个引用更新为 `Response.Redirect("~/Contacts/Default.aspx")`：  
	- *Delete.aspx.cs*
	- *Edit.aspx.cs*
	- *Insert.aspx.cs*  

	现在，当你显示和更新联系人数据时，这些链接都可正常工作。
5. 若要限制对 *Admin* 文件夹的访问，请在“解决方案资源管理器”中右键单击 *Admin* 文件夹，然后选择“添加新项”。
6. 从 Visual C# Web 模板列表中，从中间列表中选择“Web 配置文件”，接受 *Web.config* 的默认名称，然后选择“添加”。
7. 将 *Web.config* 文件中的现有 XML 内容替换为以下内容：
	<pre class="prettyprint">
&lt;?xml version="1.0"?>
&lt;configuration>
  &lt;system.web>
    &lt;authorization>
      &lt;allow roles="canEdit"/>
      &lt;deny users="*"/>
    &lt;/authorization>
  &lt;/system.web>
&lt;/configuration>
</pre>
8. 保存 *Web.config* 文件。*Web.config* 文件指定只有分配给“canEdit”角色的用户才能访问 *Admin* 文件夹中包含的页。 

如果不属于“canEdit”角色的用户尝试修改数据，则系统会将他们重定向到“登录”页。

##将包含数据库的应用程序部署到 Azure 
Web 应用程序现已完成，你可以将其发布到 Azure。

###发布应用程序 
1. 在 Visual Studio 中生成项目（“Ctrl+Shift+B”）
2. 在“解决方案资源管理器”中，右键单击该项目并选择“发布”。![发布菜单选项](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms22.png) 此时将显示“发布 Web”对话框。![“发布 Web”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms22a.png)  
3. 从“配置文件”选项卡中，选择“Azure Web 应用”作为发布目标（如果尚未选择）。![“发布 Web”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms23.png)  
4. 如果你尚未登录，请单击“登录”。
5. 从“现有 Web Apps”下拉框中选择你在本教程前面创建的现有 Web 应用，然后单击“确定”按钮。![选择“现有网站”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms25.png) 如果系统询问你是否保存对配置文件所做的更改，请选择“是”。
6. 单击“设置”选项卡。![选择“现有网站”对话框](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms26.png)  
7. 将“配置”下拉框设置为“调试”。
8. 单击“ApplicationDbContext”旁边的“向下箭头”图标，并将其设置为“ContactDB”。
9. 选中“执行 Code First 迁移”复选框。在此示例中，仅当首次发布应用程序时才应选中此复选框。这样，便只会调用 *Configuration.cs* 文件中的 *Seed* 方法一次。  

10. 然后，单击“发布”。你的应用程序将发布到 Azure。

>[AZURE.NOTE]如果在创建发布配置文件后关闭再重新打开 Visual Studio，你可能在下拉列表中看不到连接字符串。在这种情况下，请不要编辑前面创建的发布配置文件，而是像先前一样创建一个新的配置文件，然后在“设置”选项卡遵循以下步骤。

###在 Azure 中查看应用程序 
1. 在浏览器中，单击“联系人演示”链接。此时将显示“联系人列表”。![浏览器中列出的联系人](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms27.png)  

2. 在“联系人列表”页上选择“新建”。![浏览器中列出的联系人](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms29.png) 你将被重定向到“登录”页，因为你尚未使用可修改联系人的帐户登录。
3. 在输入以下电子邮件和密码后，请单击“登录”按钮。“电子邮件”：`canEditUser@wideworldimporters.com`“密码”：`Pa$$word1`![登录页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms28.png)  

4. 为每个字段输入新数据，然后按“插入”按钮。![添加新联系人页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms30.png) *EditContactList.aspx* 页中将显示新记录。![添加新联系人页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms31.png)

5. 选择“注销”链接。

###停止应用程序 
为了防止其他人注册和使用你的示例应用程序，你可以停止该 Web 应用。

1. 在 Visual Studio 中，从“视图菜单”选择“服务器资源管理器”。 
2. 在“服务器资源管理器”中，导航到“Web App”。
3. 右键单击每个网站实例并选择“停止 Web App”。![停止网站菜单项](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms26a.png)  

	或者，也可以从 Windows Azure 管理门户中选择该 Web 应用，然后单击页面底部的“停止”图标。![添加新联系人页](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms26b.png)

##查看数据库 
必须知道如何直接查看和修改数据库。如果知道如何直接处理数据库，就可以确认数据库中的数据，同时了解数据在每个表中的存储方式。

###检查 SQL Azure 数据库 
1. 在 Visual Studio 中打开“服务器资源管理器”并导航到“ContactDB”。
2. 右键单击“ContactDB”并选择“在 SQL Server 对象资源管理器中打开”。![“在 SQL Server 对象资源管理器中打开”菜单项](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms32.png)  
3. 如果显示了“添加防火墙规则”对话框，请选择“添加防火墙规则”。如果无法从 Visual Studio 展开“SQL 数据库”并且无法查看“ContactDB”，你可以按照说明打开一个或一系列防火墙端口。为此，请遵照 [MVC 教程](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)结尾处的“设置 Azure 防火墙规则”下的说明操作。或者，你也可以通过在本地生成、运行应用程序并在其中添加数据（在 Visual Studio 中按“CTRL+F5”）查看本地数据库的数据。  

4. 如果显示了“连接到服务器”对话框，请输入在本教程开头创建的“密码”，然后按“连接”按钮。如果你不记得密码，可以在本地项目文件中找到它。在“解决方案资源管理器”中展开 *Properties* 文件夹，然后展开 *PublishProfiles* 文件夹。打开 *contactmanager.pubxml* 文件（你的文件可能具有不同的名称）。搜索发布密码的文件。

5. 展开“contactDB”数据库，然后展开“表”。
6. 右键单击“dbo.AspNetUsers”表并选择“查看数据”。![“查看数据”菜单项](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms34.png) 你可以查看与 canEditUser@contoso.com 用户关联的数据。![ContactManager 窗口](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms35.png)  

###通过编辑数据库将用户添加到管理员角色 
在本教程的前面部分，你使用代码将用户添加到了 canEdit 角色。一种替代方法是直接操作成员资格表中的数据。以下步骤说明如何使用此替代方法将用户添加到角色。

1. 在“SQL Server 对象资源管理器”中，右键单击“dbo.AspNetUserRoles”，然后选择“查看数据”。![AspNetUserRoles 数据](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms36.png)  
2. 复制 *RoleId* 并将其粘贴到空（新）行中。![AspNetUserRoles 数据](./media/web-sites-dotnet-deploy-aspnet-webforms-app-membership-oauth-sql-database/SecureWebForms37.png)  
3. 在“dbo.AspNetUsers”表中，找到你要放入角色的用户并复制该用户的 *Id*。
4. 将复制的 *Id* 粘贴到“AspNetUserRoles”表中新行的“UserId”字段内。  

>[AZURE.NOTE]我们正在开发可显著简化用户和角色管理的工具。

##后续步骤
有关 ASP.NET Web 窗体的详细信息，请参阅 ASP.NET Web 应用上的 [了解 ASP.NET Web 窗体](http://www.asp.net/web-forms)和 [Windows Azure 教程和指南](/documentation/services/web-sites/#net)。

本教程基于 Rick Anderson (Twitter [@RickAndMSFT](https://twitter.com/RickAndMSFT)) 在 Tom Dykstra 和 Barry Dorrans (Twitter [@blowdart](https://twitter.com/blowdart)) 的帮助下编写的 MVC 教程[创建包含身份验证和 SQL 数据库的 ASP.NET MVC 应用并部署到 Azure 网站](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。

请提供有关你喜欢的内容或者你希望看到改善的内容的反馈，不仅关于教程本身，也关于它所演示的产品。你的反馈将帮助我们确定优先改进哪些方面。你还可以在[教我编写代码](http://aspnet.uservoice.com/forums/228522-show-me-how-with-code)上请求帮助以及对新主题投票。

 

<!---HONumber=71-->