<properties linkid="develop-dotnet-aspnet-mvc-4-mobile-website" urlDisplayName="ASP.NET MVC 4 mobile website" pageTitle=".NET ASP.NET MVC 4 移动网站 &ndash; Azure 教程" metaKeywords="Azure tutorial, Azure web app tutorial, Azure mobile app, Azure ASP.NET MVC 4,,ASP.NET MVC" description="本教程说明如何使用 ASP.NET MVC 4 Web 应用程序中的移动功能将 Web 应用程序部署到 Azure 网站。" metaCanonical="" services="web-sites" documentationCenter=".NET" title="在 Azure 网站上部署 ASP.NET MVC 移动 Web 应用程序" authors="tdykstra" solutions="" manager="" editor="" />

# 在 Azure 网站上部署 ASP.NET MVC 移动 Web 应用程序

***由 [Rick Anderson][Rick Anderson] 撰写，更新时间：2013 年 6 月 26 日。***

本教程介绍将 Web 应用程序部署到 Azure 网站的基础知识。在本教程中，我们将使用 ASP.NET MVC 4 Web 应用程序中的移动功能。若要执行本教程中的步骤，可以使用 Microsoft Visual Studio 2012。也可以使用 [Visual Studio Express 2012][Visual Studio Express 2012]，这是 Microsoft Visual Studio 的免费版本。

## 您将了解到以下内容：

-   ASP.NET MVC 4 模板如何使用 HTML5 视区属性和自适应呈现来改善在移动设备上的显示。
-   如何创建移动特定视图。
-   如何创建视图切换器，以便用户能够在应用程序的移动视图与桌面视图间切换。
-   如何将 Web 应用程序部署到 Azure。

在本教程中，您将在初学者项目提供的简单会议列表应用程序中添加移动功能。下面的屏幕快照显示完成后的应用程序的主页面，与在 Windows 7 Phone Emulator 中看到的一样。

![MVC4 会议应用程序主页面。][MVC4 会议应用程序主页面。]

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 设置开发环境

通过安装适用于 .NET Framework 的 Azure SDK 来设置开发环境。

1.  若要安装 Azure SDK for .NET，请单击以下链接。如果您尚未安装 Visual Studio 2012，可单击该链接安装它。本教程需要 Visual Studio 2012。
    [Azure SDK for Visual Studio 2012][Azure SDK for Visual Studio 2012]
2.  当提示您运行或保存可执行安装文件时，单击“运行”。
3.  在“Web 平台安装程序”窗口中，单击“安装”，然后进行安装。

![Web 平台安装程序 – Azure SDK for .NET][Web 平台安装程序 – Azure SDK for .NET]

还需要安装移动浏览器模拟器。以下版本均可：

-   [Windows 7 Phone Emulator][Windows 7 Phone Emulator]。（本教程的大部分屏幕快照中都使用这种模拟器。）
-   更改用户代理字符串以模拟 iPhone。请查阅 How-To Geek 网站上的[此博客文章][此博客文章]。
-   [Opera Mobile Emulator][Opera Mobile Emulator]。
-   [Apple Safari][Apple Safari]，将用户代理设置为 iPhone。有关如何将 Safari 中的用户代理设置为“iPhone”的说明，请参阅 David Alison 的博文[如何让 Safari 模拟 IE][如何让 Safari 模拟 IE]。
-   [FireFox][FireFox]，带 [FireFox 用户代理切换器][FireFox 用户代理切换器]。

本教程使用 C# 代码。不过，初学者项目和已完成项目将使用 Visual Basic 代码。可通过包含 Visual Basic 和 C# 源代码的 Visual Studio 项目加深对本主题的了解：

-   [初学者项目下载][初学者项目下载]
-   [已完成项目下载][已完成项目下载]

## 本教程中的步骤

-   [创建 Azure 网站][创建 Azure 网站]
-   [设置初学者项目][设置初学者项目]
-   [重写视图、布局和分部视图][重写视图、布局和分部视图]
-   [使用 jQuery Mobile 定义移动浏览器界面][使用 jQuery Mobile 定义移动浏览器界面]
-   [改进发言人列表][改进发言人列表]
-   [创建移动发言人视图][创建移动发言人视图]
-   [改进标签列表][改进标签列表]
-   [改进日期列表][改进日期列表]
-   [改进 SessionsTable 视图][改进 SessionsTable 视图]
-   [改进 SessionByCode 视图][改进 SessionByCode 视图]
-   [将应用程序部署到 Azure 网站][将应用程序部署到 Azure 网站]

### <a name="bkmk_CreateWebSite"></a>在 Azure 中创建网站

您的 Azure 网站将在共享宿主环境中运行，这意味着它将在与其他 Azure 客户端共享的虚拟机 (VM) 上运行。共享宿主环境是一种在云中开始工作的低成本方式。稍后，如果您的 Web 流量增加，则应用程序可进行扩展，通过在专用 VM 上运行来满足需要。如果您需要一个更复杂的体系结构，则可迁移到 Azure 云服务。云服务在您可根据自己的需求进行配置的专用 VM 上运行。

1.  登录到 [Azure 管理门户][Azure 管理门户]。在管理门户中，单击“新建”。

    ![][0]

2.  单击“网站”，然后单击“快速创建”。

    ![][1]

3.  在“新建网站”中，在“URL”框中输入一个字符串作为您的应用程序的唯一 URL。

    ![][2]

    完整的 URL 由您在此处输入的字符串以及您在文本框下面看到的后缀组成。示意图中显示“MyMobileMVC4WebSite”，但如果有人已经使用了该 URL，则您必须另外选择一个。选择您所在的“区域”。

4.  单击对话框底部的复选标记以指示您已完成操作。

管理门户返回到“网站”页面，“状态”列显示正在创建网站。稍后（通常不到一分钟），“状态”列将显示网站创建成功。在左侧的导航栏中，“网站”图标中将显示您的帐户下拥有的网站数量，“SQL Database”图标中显示数据库的数量。

![][3]

### <a name="bkmk_setupstarterproject"></a>设置初学者项目。

1.  下载[会议列表应用程序初学者项目][初学者项目下载]。

2.  然后，在 Windows 资源管理器中，右键单击 MvcMobileStarterBeta.zip 文件并选择*属性*。

3.  在 MvcMobileRTMStarter.zip“属性”对话框中，选择“取消阻止”按钮。（取消阻止后，当您尝试使用从 Web 下载的 .zip 文件时，将不再显示安全警告。）

    ![“属性”对话框。][“属性”对话框。]

4.  右键单击 MvcMobile.zip 文件，选择“全部提取”来解压缩该文件。

5.  在 Visual Studio 中，打开 MvcMobile.sln 文件。

### 要运行初学者项目，请执行以下步骤

1.  按 Ctrl + F5 运行该应用程序，它将在桌面浏览器中显示。
2.  启动移动浏览器模拟器，将会议应用程序的 URL 复制到模拟器，然后单击“按标签浏览”链接。

    -   如果您使用的是 Windows Phone Emulator，则在 URL 栏中单击并按“暂停”键来访问键盘。下图显示 AllTags 视图（选择“按标签浏览”后显示）。

    ![按标签浏览页面。][按标签浏览页面。]

显示内容在移动设备上一目了然。选择 ASP.NET 链接。

![浏览标记为 ASP.NET 的会话。][浏览标记为 ASP.NET 的会话。]

ASP.NET 标签视图显示非常混乱。例如，“日期”列很难阅读。在本教程的稍后部分，您将创建 AllTags 视图的一个版本，它专门针对移动浏览器且显示内容易于阅读。

## <a name="bkmk_overrideviews"></a>重写视图、布局和分部视图

在本节中，您将创建一个移动特定布局文件。

ASP.NET MVC 4 中的一个重要新功能是一种允许您针对常规移动浏览器、单个移动浏览器或任何特定浏览器重写任何视图（包括布局和分部视图）的简单机制。要提供移动特定视图，您可以复制视图文件并在文件名中添加 .Mobile。例如，若要创建移动索引视图，可将 *Views\\Home\\Index.cshtml* 复制到 *Views\\Home\\Index.Mobile.cshtml*。

若要开始，请将 \*Views\\Shared\\\_Layout.cshtml\* 复制到 \*Views\\Shared\\\_Layout.Mobile.cshtml*。打开* \_Layout.Mobile.cshtml\*，将标题从 **MVC4 Conference** 更改为 **Conference (Mobile)**。

在每次 **Html.ActionLink** 调用中，删除每个链接 ActionLink 中的“Browse by”。以下代码显示已完成的移动布局文件主体部分。

     <body>
        <div class="page">
            <div id="header">
                <div id="logindisplay"></div>
                <div id="title">
                    <h1> Conference (Mobile)</h1>
                </div>
                <div id="menucontainer">
                    <ul id="menu">
                        <li>@Html.ActionLink("Home", "Index", "Home")</li>
                        <li>@Html.ActionLink("Date", "AllDates", "Home")</li>
                        <li>@Html.ActionLink("Speaker", "AllSpeakers", "Home")</li>
                        <li>@Html.ActionLink("Tag", "AllTags", "Home")</li>
                    </ul>
                </div>
            </div>
            <div id="main">
                @RenderBody()
            </div>
            <div id="footer">
            </div>
        </div>
    </body>

将 *Views\\Home\\AllTags.cshtml* 文件复制到 *Views\\Home\\AllTags.Mobile.cshtml*。打开此新文件，将 \<h2\> 元素从“Tags”更改为“Tags (M)”：

     <h2>Tags (M)</h2>

使用桌面浏览器和移动浏览器模拟器浏览到标签页。移动浏览器模拟器显示您所做的两处改动。

![显示对标签页面的更改][显示对标签页面的更改]

与此相反，桌面显示没有变化。

![显示桌面标签视图][显示桌面标签视图]

## <a name="bkmk_usejquerymobile"></a>使用 jQuery Mobile 定义移动浏览器界面

在本节中，您将安装 jQuery.Mobile.MVC NuGet 包，它将安装 jQuery Mobile 和视图切换器小组件。

[jQuery Mobile][jQuery Mobile] 库提供一个可在所有主要移动浏览器上使用的用户界面框架。jQuery Mobile 可对支持 CSS 和 JavaScript 的移动浏览器应用渐进增强。渐进增强允许所有浏览器显示网页的基本内容，同时允许更强大的浏览器和设备拥有更丰富的显示。jQuery Mobile 中包括的 JavaScript 和 CSS 文件为众多元素设定了样式来适应移动浏览器，无需对标记做任何更改。

1.  删除先前创建的 \*Shared\\\_Layout.Mobile.cshtml\* 文件。

2.  将 *Views\\Home\\AllTags.Mobile.cshtml* 重命名为 *Views\\Home\\AllTags.Mobile.cshtml.hide*（之后还会再次用到该文件。）因为该文件不再具有 .cshtml 扩展名，所以 ASP.NET MVC 运行时不会使用它来呈现 *AllTags* 视图。

3.  执行以下操作安装 jQuery.Mobile.MVC NuGet 程序包：

    a. 从“工具”菜单中选择“程序包管理器”控制台，然后选择“库程序包管理器”。

        ![Library package manager][jquery1]

    b. 在“程序包管理器控制台”中，输入 *Install-Package jQuery.Mobile.MVC -version 1.0.0*

        ![Package manager console][jquery2]

jQuery.Mobile.MVC NuGet 程序包将安装以下内容：

-   *App\_Start\\BundleMobileConfig.cs* 文件，引用添加的 jQuery JavaScript 和 CSS 文件时需要此文件。必须按照下面的说明并引用该文件中定义的移动捆绑。
-   jQuery Mobile CSS 文件。
-   一个 ViewSwitcher 控制器小组件 (*Controllers\\ViewSwitcherController.cs)*。
-   jQuery Mobile JavaScript 文件。
-   jQuery Mobile 样式的布局文件 (*Views\\Shared\_Layout.Mobile.cshtml*)。
-   视图切换器分部视图 (*MvcMobile\\Views\\Shared\_ViewSwitcher.cshtml*)，它在每个页面的顶部提供一个可在桌面视图和移动视图之间切换的链接。
-   Content\\images 文件夹中的几个 .png 和 .gif 图像文件。

打开 *Global.asax* 文件，添加以下代码作为 Application\_Start 方法的最后一行。

    BundleMobileConfig.RegisterBundles(BundleTable.Bundles);

以下代码显示完整的 Global.asax 文件。

    using System; 
    using System.Web.Http; 
    using System.Web.Mvc; 
    using System.Web.Optimization; 
    using System.Web.Routing; 
    using System.Web.WebPages; 
     
    namespace MvcMobile 
    { 
     
        public class MvcApplication : System.Web.HttpApplication 
        { 
            protected void Application_Start() 
            { 
                DisplayModeProvider.Instance.Modes.Insert(0, new DefaultDisplayMode("iPhone") 
                { 
                    ContextCondition = (context => context.GetOverriddenUserAgent().IndexOf 
                        ("iPhone", StringComparison.OrdinalIgnoreCase) >= 0) 
                }); 
                AreaRegistration.RegisterAllAreas(); 
     
                WebApiConfig.Register(GlobalConfiguration.Configuration); 
                FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters); 
                RouteConfig.RegisterRoutes(RouteTable.Routes); 
                BundleConfig.RegisterBundles(BundleTable.Bundles); 
                BundleMobileConfig.RegisterBundles(BundleTable.Bundles); 
            } 
        } 
    }

打开 \*MvcMobile\\Views\\Shared\\\_Layout.Mobile.cshtml\* 文件，直接在 *Html.Partial* 调用后添加以下标记：

    <div data-role="header" align="center">
        @Html.ActionLink("Home", "Index", "Home")
        @Html.ActionLink("Date", "AllDates")
        @Html.ActionLink("Speaker", "AllSpeakers")
        @Html.ActionLink("Tag", "AllTags")
    </div>

完整的主体部分如下所示：

    <body>
        <div data-role="page" data-theme="a">
            @Html.Partial("_ViewSwitcher")
            <div data-role="header" align="center">
                @Html.ActionLink("Home", "Index", "Home")
                @Html.ActionLink("Date", "AllDates")
                @Html.ActionLink("Speaker", "AllSpeakers")
                @Html.ActionLink("Tag", "AllTags")
            </div>
            <div data-role="header">
                <h1>@ViewBag.Title</h1>
            </div>
            <div data-role="content">
                @RenderSection("featured", false)
                @RenderBody()
            </div>
        </div>
    </body>

构建应用程序，在移动浏览器模拟器中浏览到 AllTags 视图。您会看到如下内容：

![通过 nuget 安装 jquery 后。][通过 nuget 安装 jquery 后。]

<div class="dev-callout"> 
<b>说明</b> 
<p>对于 IE 或 Chrome，您可以通过将用户代理字符串设置为 iPhone，然后使用 F-12 开发人员工具来调试移动特定代码。如果您的移动浏览器未将&ldquo;主页&rdquo;、&ldquo;发言人&rdquo;、&ldquo;标签&rdquo;和&ldquo;日期&rdquo;链接显示为按钮，表明对 jQuery Mobile 脚本和 CSS 文件的引用可能不正确。</p> 
</div>

除了样式发生改变外，您还可以看到“显示移动视图”和一个允许您从移动视图切换到桌面视图的链接。选择“桌面视图链接”将显示桌面视图。

<!--![Display desktop view][jquery4]-->

桌面视图不提供直接导航回移动视图的途径。现在来修复此问题。打开 \*Views\\Shared\\\_Layout.cshtml\* 文件。在 \<body\> 元素紧下方，添加以下代码来呈现视图切换器小组件：

    @Html.Partial("_ViewSwitcher")

下面是完成后的代码：

    <body>
        @Html.Partial("_ViewSwitcher")

        <div id="title">
            <h1> MVC4 Conference </h1>
        </div>

        @*Items removed for clarity.*@
    </body>

在移动浏览器中刷新“AllTags”视图。您现在可以在桌面和移动视图间导航了。

![导航到移动视图。][导航到移动视图。]

<div class="dev-callout"> 
<b>说明</b> 
<p>您可以将下面的代码添加到 Views\Shared\_ViewSwitcher.cshtml 的末尾，以便在使用用户代理字符串设置为移动设备的浏览器时帮助调试视图。</p> 

<pre>
else 
    { 
@:Not Mobile/Get 
    } 
</pre>
<p>另外，将以下标题添加到 Views\Shared\_Layout.cshtml 文件中。</p> 
<pre>
&lt;h1&gt;Non Mobile Layout MVC4 Conference&lt;/h1&gt;
</pre>
</div>

在桌面浏览器中浏览到 AllTags 页面。因为只将视图切换器小组件添加到了移动布局页面中，所以在桌面浏览器中未显示该小组件。在教程的稍后部分中，您将学习如何将视图切换器小组件添加到桌面视图中。

![查看桌面体验。][查看桌面体验。]

## <a name="bkmk_Improvespeakerslist"></a>改进发言人列表

在移动浏览器中，选择“发言人”链接。因为没有移动视图 (*AllSpeakers.Mobile.cshtml*)，所以使用移动布局视图 (\*\_Layout.Mobile.cshtml\*) 呈现默认发言人内容 (*AllSpeakers.cshtml*)。

![查看移动发言人列表。][查看移动发言人列表。]

通过在 *Views\_ViewStart.cshtml* 文件中将 RequireConsistentDisplayMode 设置为 true，可以全局禁止默认（非移动）视图在移动布局内呈现，像下面这样：

![][4]

    @{
        Layout = "~/Views/Shared/_Layout.cshtml";
        DisplayModes.RequireConsistentDisplayMode = true;
    }

当 *RequireConsistentDisplayMode* 设置为 true 时，移动布局 (\*\_Layout.Mobile.cshtml*) 只用于移动视图。（也就是说，视图文件仅为 ViewName.Mobile.cshtml 形式。）您可能需要将* RequireConsistentDisplayMode\* 设置为 true（如果您的移动布局不太适合您的非移动视图）。下面的屏幕快照显示当 *RequireConsistentDisplayMode* 设置为 true 时，如何呈现“发言人”页面。

![][5]

您可以通过在视图文件中将 *RequireConsistentDisplayMode* 设置为 false 来禁用视图中一致的显示模式。*Views\\Home\\AllSpeakers.cshtml* 文件中的以下标记将 *RequireConsistentDisplayMode* 设置为 false：

    @model IEnumerable<string>
    @{
        ViewBag.Title = "All speakers";
        DisplayModes.RequireConsistentDisplayMode = false;
    }

## <a name="bkmk_mobilespeakersview"></a>创建移动发言人视图

正如您刚才看到的，“发言人”视图虽然可读，但链接字迹小，不易在移动设备上点击。在本节中，您将创建一个移动特定“发言人”视图，它看起来像一个现代移动应用程序，链接字迹大、易于点击，而且包含可快速查找发言人的搜索框。

1.  将 *AllSpeakers.cshtml* 复制到 *AllSpeakers.Mobile.cshtml。*打开 *AllSpeakers.Mobile.cshtml* 文件，移除 \<h2\> 标题元素。
2.  在 **\<ul\>** 标签中，添加 data-role 属性，将其值设置为 *listview*。同其他 \*data-\*\* 属性一样，*data-role="listview"* 可使大的列表项更易于点击。完成的标记如下所示：

        @model IEnumerable<string>
        @{
            ViewBag.Title = "All speakers";
        }
        <ul data-role="listview">
            @foreach(var speaker in Model) {
                <li>@Html.ActionLink(speaker, "SessionsBySpeaker", new { speaker })</li>
            }
        </ul>

3.  刷新移动浏览器。更新的视图如下所示：

    ![][6]

4.  在 **\<ul\>** 标签中，添加 data-filter 属性，将其值设置为 true。下面的代码显示 ul 标记。

        <ul data-role="listview" data-filter="true">

下图在页面顶部显示由 data-filter 属性产生的搜索筛选器框。

![][5]

当您在搜索框中键入每个字母时，jQuery Mobile 将筛选显示的列表，如下图所示。

![][7]

## <a name="bkmk_improvetags"></a>改进标签列表

与默认“发言人”视图一样，“标签”视图虽然可读，但链接字迹小，不易在移动设备上点击。在本节中，您将像修正“发言人”视图那样修正“标签”视图。

1.  将 *Views\\Home\\AllTags.Mobile.cshtml.hide* 文件重命名为 *Views\\Home\\AllTags.Mobile.cshtml*。打开重命名的文件，移除 **\<h2\>** 元素。

2.  将 data-role 和 data-filter 属性添加到 **\<ul\>** 标签中，如下所示：

        <ul data-role="listview" data-filter="true">

    下图显示筛选字母 J 的标签页。

![][8]

## <a name="bkmk_improvedates"></a>改进日期列表

您可以像改进“发言人”和“标签”视图那样改进“日期”视图，以使其更容易在移动设备上使用。

1.  将 *Views\\Home\\AllDates.Mobile.cshtml* 文件复制到 *Views\\Home\\AllDates.Mobile.cshtml*。
2.  打开新文件，移除 **\<h2\>** 元素。
3.  将 *data-role="listview"* 添加到 \<ul\> 标签中，如下所示：

        <ul data-role="listview">

下图显示添加 data-role 属性后“日期”页面的外观。

![][9]

将 *Views\\Home\\AllDates.Mobile.cshtml* 文件的内容替换为下列代码：

    @model IEnumerable<DateTime>
    @{
        ViewBag.Title = "All dates";
        DateTime lastDay = default(DateTime);
    }
    <ul data-role="listview">
        @foreach(var date in Model) {
            if (date.Date != lastDay) {
                lastDay = date.Date;
                <li data-role="list-divider">@date.Date.ToString("ddd, MMM dd")</li>
            }
            <li>@Html.ActionLink(date.ToString("h:mm tt"), "SessionsByDate", new { date })</li>
        }
    </ul>

此代码按日期将所有会话分组。它为每个新日期创建一条列表分隔线，在分隔线下列出一天的所有会话。当此代码运行时，它看起来像这样：

![][10]

## <a name="bkmk_improvesessionstable"></a>改进 SessionsTable 视图

在本节中，您将创建一个移动特定的会话视图。我们所做的更改将比在我们创建的其他视图中更广泛。

在移动浏览器中，点击“发言人”按钮，然后在搜索框中输入 Sc。

![][11]

点击 **Scott Hanselman** 链接。

![][12]

正如您所看到的，显示的内容难以在移动浏览器上阅读。日期列难于阅读，标签列也超出了视图。为解决这些问题，可将 Views\*Home\\SessionsTable.cshtml\* 复制到 *Views\\Home\\SessionsTable.Mobile.cshtml*，然后用以下代码替换文件的内容：

    @using MvcMobile.Models
    @model IEnumerable<Session>

    <ul data-role="listview">
        @foreach(var session in Model) {
            <li>
                <a href="@Url.Action("SessionByCode", new { session.Code })">
                    <h3>@session.Title</h3>
                    <p><strong>@string.Join(", ", session.Speakers)</strong></p>
                    <p>@session.DateText</p>
                </a>
            </li>
        }
    </ul>

该代码移除了 Room 和 Tag 列，将标题、发言人和日期设置为竖式，这样，即可在移动浏览器中阅读所有此类信息。下图反映了代码更改。

![][13]

## <a name="bkmk_improvesessionbycode"></a>改进 SessionByCode 视图

最后，您将创建一个移动特定的 **SessionByCode** 视图。在移动浏览器中，点击“发言人”按钮，然后在搜索框中输入 Sc。

![][14]

点击 **Scott Hanselman** 链接。将显示 Scott Hanselman 的会话。

![][12]

选择 **An Overview of the MS Web Stack of Love** 链接。

![][15]

默认桌面视图虽然不错，但仍可以改进。

将 *Views\\Home\\SessionByCode.cshtml* 复制到 *Views\\Home\\SessionByCode.Mobile.cshtml*，用以下标记替换 *Views\\Home\\SessionByCode.Mobile.cshtml* 文件的内容：

    @model MvcMobile.Models.Session

    @{
        ViewBag.Title = "Session details";
    }
    <h2>@Model.Title</h2>
    <p>
        <strong>@Model.DateText</strong> in <strong>@Model.Room</strong>
    </p>

    <ul data-role="listview" data-inset="true">
        <li data-role="list-divider">Speakers</li>
        @foreach (var speaker in Model.Speakers) {
            <li>@Html.ActionLink(speaker, "SessionsBySpeaker", new { speaker })</li>
        }
    </ul>

    <p>@Model.Description</p>
    <h4>Code: @Model.Code</h4>

    <ul data-role="listview" data-inset="true">
        <li data-role="list-divider">Tags</li>
        @foreach (var tag in Model.Tags) {
            <li>@Html.ActionLink(tag, "SessionsByTag", new { tag })</li>
        }
    </ul>

新的标记使用 **data-role** 属性改进视图的布局。

刷新移动浏览器。下图反映您刚才所做的代码更改：

![][16]

## <a name="bkmk_deployapplciation"></a>将应用程序部署到 Azure 网站

1.  在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。

    ![项目上下文菜单中的“发布”][项目上下文菜单中的“发布”]

    “发布 Web”向导将打开。

2.  在“发布 Web”向导的“配置文件”选项卡中，单击“导入”。

    ![导入发布设置][导入发布设置]

    “导入发布配置文件”对话框随即出现。

3.  如果您之前未在 Visual Studio 中添加 Azure 订阅，请执行下列步骤。在这些步骤中，您将添加订阅，以便“从 Azure 网站导入”下的下拉列表中包含您的网站。

    1.  在“导入发布配置文件”对话框中，单击“添加 Azure 订阅”。

    ![添加 Windows Azure 订阅][添加 Windows Azure 订阅]

    1.  在“导入 Azure 订阅”对话框中，单击“下载订阅文件”。

    ![下载订阅][下载订阅]

    1.  在浏览器窗口中，保存 *.publishsettings* 文件。

    ![下载发布文件][下载发布文件]

    [WACOM.INCLUDE [publishsettingsfilewarningchunk](../includes/publishsettingsfilewarningchunk.md)]
    </br>

    1.  在“导入 Azure 订阅”对话框中，单击“浏览”并导航到 *.publishsettings* 文件。

    ![下载订阅][下载订阅]

    1.  单击“导入”。

    ![导入][导入]

    1.  在“导入发布配置文件”对话框中，选择“从 Azure 网站导入”，再从下拉列表中选择您的网站，然后单击“确定”。

    ![导入发布配置文件][导入发布配置文件]

    1.  在“连接”选项卡中，单击“验证连接”确保设置正确。

    ![验证连接][验证连接]

    1.  连接通过验证后，“验证连接”按钮旁将出现一个绿色复选标记。
        ![连接成功图标和“连接”选项卡中的“下一步”按钮][连接成功图标和“连接”选项卡中的“下一步”按钮]

    2.  您可以接受此页上的所有默认设置。您将部署“发布”生成配置，因此不需要在目前服务器上删除文件。“数据库”下的 **UsersContext (DefaultConnection)** 条目来自使用 DefaultConnection 字符串的 *UsersContext:DbContext* 类。
        单击“下一步”。

    ![“连接”选项卡中的连接成功图标和“下一步”按钮][“连接”选项卡中的连接成功图标和“下一步”按钮]

    1.  在“预览”选项卡中，单击“开始预览”。
        该选项卡将显示将要复制到服务器的文件列表。显示预览并不是发布应用程序所必需的，但它是一个需要了解的很有用的功能。在本例中，您不需要对显示的文件列表执行任何操作。下次发布时，仅已更改的文件会出现在预览列表中。

    ![“预览”选项卡中的“开始预览”按钮][“预览”选项卡中的“开始预览”按钮]

    1.  单击“发布”。

    Visual Studio 开始执行将文件复制到 Azure 服务器的过程。“输出”窗口将显示已执行的部署操作并报告已成功完成部署。

    1.  默认浏览器会自动打开，并指向所部署网站的 URL。您创建的应用程序现在在云中运行。

    ![][17]

您可以使用手机模拟器，通过在移动浏览器中浏览到网站 URL 来测试您的实时网站。

<!-- Internal Links --> <!-- Images --> <!-- External Links -->

  [Rick Anderson]: https://twitter.com/RickAndMSFT
  [Visual Studio Express 2012]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx
  [MVC4 会议应用程序主页面。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/FinishedAPPMainScreen.png
  [Azure SDK for Visual Studio 2012]: http://go.microsoft.com/fwlink/?LinkId=254364
  [Web 平台安装程序 – Azure SDK for .NET]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/WebPIAzureSdk20NetVS12.png
  [Windows 7 Phone Emulator]: http://msdn.microsoft.com/zh-cn/library/ff402530(VS.92).aspx
  [此博客文章]: http://www.howtogeek.com/113439/how-to-change-your-browsers-user-agent-without-installing-any-extensions/
  [Opera Mobile Emulator]: http://www.opera.com/developer/tools/mobile/
  [Apple Safari]: http://www.apple.com/cn/safari/
  [如何让 Safari 模拟 IE]: http://www.davidalison.com/2008/05/how-to-let-safari-pretend-its-ie.html
  [FireFox]: http://www.bing.com/search?q=firefox+download
  [FireFox 用户代理切换器]: https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher/
  [初学者项目下载]: http://go.microsoft.com/fwlink/?LinkId=228307
  [已完成项目下载]: http://go.microsoft.com/fwlink/?LinkId=228306
  [创建 Azure 网站]: #bkmk_CreateWebSite
  [设置初学者项目]: #bkmk_setupstarterproject
  [重写视图、布局和分部视图]: #bkmk_overrideviews
  [使用 jQuery Mobile 定义移动浏览器界面]: #bkmk_usejquerymobile
  [改进发言人列表]: #bkmk_Improvespeakerslis
  [创建移动发言人视图]: #bkmk_mobilespeakersview
  [改进标签列表]: #bkmk_improvetags
  [改进日期列表]: #bkmk_improvedates
  [改进 SessionsTable 视图]: #bkmk_improvesessionstable
  [改进 SessionByCode 视图]: #bkmk_improvesessionbycode
  [将应用程序部署到 Azure 网站]: #bkmk_deployapplciation
  [Azure 管理门户]: https://manage.windowsazure.cn
  [0]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/depoly_mobile_new_website_1.png
  [1]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/depoly_mobile_new_website_2.png
  [2]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/depoly_mobile_new_website_3.png
  [3]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/depoly_mobile_new_website_4.png
  [“属性”对话框。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/propertiespopup.png
  [按标签浏览页面。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/BrowseByTagWithCallout.png
  [浏览标记为 ASP.NET 的会话。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/ASPNetPage.png
  [显示对标签页面的更改]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p2m_layouttags_mobile_thumb.png
  [显示桌面标签视图]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/Windows-Live-Writer_ASP_NET-MVC-4-Mobile-Features_D2FF_p2_layoutTagsDesktop_thumb.png
  [jQuery Mobile]: http://jquerymobile.com/demos/1.0b3/#/demos/1.0b3/docs/about/intro.html
  [通过 nuget 安装 jquery 后。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_afternuget_thumb.png
  [导航到移动视图。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_desktopviewwithmobilelink_thumb.png
  [查看桌面体验。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/Windows-Live-Writer_ASP_NET-MVC-4-Mobile-Features_D2FF_p3_desktopBrowser_thumb.png
  [查看移动发言人列表。]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_speakersdesktop_thumb.png
  [4]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_speakersconsistent_thumb.png
  [5]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_data_filter_thumb.png
  [6]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_updatedspeakerview1_thumb.png
  [7]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_data_filter_sc_thumb.png
  [8]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_tags_j_thumb.png
  [9]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_dates1_thumb.png
  [10]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_dates2_thumb.png
  [11]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_data_filter_sc_thumb_1.png
  [12]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_scottha_thumb.png
  [13]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_sessionsbyscottha_thumb.png
  [14]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_data_filter_sc_thumb_2.png
  [15]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_ps_love_thumb.png
  [16]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/windows-live-writer_asp_net-mvc-4-mobile-features_d2ff_p3_love2_thumb.png
  [项目上下文菜单中的“发布”]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/PublishVSSolution.png
  [导入发布设置]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/ImportPublishSettings.png
  [添加 Windows Azure 订阅]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/rzAddWAsub.png
  [下载订阅]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/rzDownLoad.png
  [下载发布文件]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/rzDown2.png
  [导入]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/rzImp.png
  [导入发布配置文件]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/ImportPublishProfile.png
  [验证连接]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/ValidateConnection.png
  [连接成功图标和“连接”选项卡中的“下一步”按钮]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/dntutmobile-deploy1-publish-005.png
  [“连接”选项卡中的连接成功图标和“下一步”按钮]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/rxPWS.png
  [“预览”选项卡中的“开始预览”按钮]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/dntutmobile-deploy1-publish-007.png
  [17]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/depoly_mobile_new_website_15.png
