<properties 
	pageTitle="在 Azure 上部署 ASP.NET MVC 5 移动 Web 应用" 
	description="本教程说明如何使用 ASP.NET MVC 5 Web 应用程序中的移动功能将 Web 应用部署到 Azure。" 
	services="app-service" 
	documentationCenter=".net" 
	authors="cephalin" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service"
	ms.date="01/12/2016"
	wacn.date="02/26/2016"/>


# 在 Azure 上部署 ASP.NET MVC 5 移动 Web 应用

本教程介绍有关如何生成一个方便移动的 ASP.NET MVC 5 Web 应用并将其部署到 Azure Web 应用的基础知识。对于本教程中，你需要 [Visual Studio Express 2013 for Web][Visual Studio Express 2013] 或者 Visual Studio 专业版（如果你已具有）。你可以使用 [Visual Studio 2015]，但屏幕快照将会有所不同并且你必须使用 ASP.NET 4.x 模板。

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 你将生成

在本教程中，你将在[初学者项目][StarterProject]提供的简单会议列表应用程序中添加移动功能。以下屏幕截图显示了已完成的应用程序中的 ASP.NET 会话在 Internet Explorer 11 F12 开发人员工具的浏览器模拟器中的情况。

![][FixedSessionsByTag]

可以使用 Internet Explorer 11 F12 开发人员工具和 [Fiddler 工具][Fiddler]来帮助调试应用程序。

## 将要学到的技能

学习内容：

-	如何使用 Visual Studio 2013 将 Web 应用程序直接发布到 Azure 中的 Web 应用。
-   ASP.NET MVC 5 模板如何使用 CSS Bootstrap 框架来改善在移动设备上的显示。
-   如何创建面向特定移动浏览器（如 iPhone 和 Android）的移动视图
-   如何创建响应式视图（跨设备响应不同浏览器的视图）

## 设置开发环境

请通过安装 Azure SDK for .NET 2.5.1 或更高版本来设置开发环境。

1. 若要安装 Azure SDK for .NET，请单击以下链接。如果你尚未安装 Visual Studio 2013，可单击该链接安装它。本教程需要安装 Visual Studio 2013。[Azure SDK for Visual Studio 2013][AzureSDKVs2013]
1. 在“Web 平台安装程序”窗口中，单击“安装”，然后进行安装。

还需要安装移动浏览器模拟器。以下版本均可：

-   [Internet Explorer 11 F12 开发人员工具][EmulatorIE11]中的浏览器模拟器（所有移动浏览器屏幕截图都使用此工具）。它为 Windows Phone 8、Windows Phone 7 和 Apple iPad 提供用户代理字符串预设。
-	Google Chrome DevTools 中的浏览器模拟器。它包含众多 Android 设备以及 Apple iPhone、Apple iPad 和 Amazon Kindle Fire 的预设。它还会模拟触控事件。
-   [Opera Mobile 模拟器][EmulatorOpera]

本主题可以附带包含具有 C# 源代码的 Visual Studio：

-   [初学者项目下载][StarterProject]
-   [已完成项目下载][CompletedProject]

##<a name="bkmk_DeployStarterProject"></a>将初学者项目部署到 Azure Web 应用

1.	下载会议列表应用程序[初学者项目][StarterProject]。

2. 	然后，在 Windows 资源管理器中，右键单击下载的 ZIP 文件并选择“ *属性* ”。

3. 	在“属性”对话框中，选择“取消阻止”按钮。（取消阻止后，当你尝试使用从 Web 下载的 *.zip* 文件时，将不再显示安全警告。）

4.	右键单击 ZIP 文件，选择“全部提取”来解压缩该文件。

5. 	在 Visual Studio 中，打开 *C#\\Mvc5Mobile.sln* 文件。

6.	登录到[经典管理门户](https://manage.windowsazure.cn/)，然后单击已退出的网站或创建新网站。

7.	在“仪表板”中的“速览”下，单击“下载发布配置文件”

6.  在“解决方案资源管理器”中，右键单击该项目并单击“发布”。

	![][DeployClickPublish]

7.	在“发布 Web”中单击“导入”，然后选择前面下载的发布配置文件。

13.	“发布 Web”对话框中将填充新 Web 应用的设置。单击“发布”。

	![][DeployPublishSite]

	在 Visual Studio 完成将初学者项目发布到 Azure Web 应用后，将打开桌面浏览器并显示实时 Web 应用。

14.	启动移动浏览器模拟器，将会议应用程序 (*<prefix>*.chinacloudsites.cn) URL 复制到模拟器，然后单击“按标签浏览”链接。如果使用 Internet Explorer 11 作为默认浏览器，则只需依次按 `F12` 和 `Ctrl+8`，然后将浏览器配置文件更改为“Windows Phone”。下图显示纵向模式下的 *AllTags* 视图（选择“按标签浏览”后显示）。

	![][AllTags]

>[AZURE.TIP] 虽然可以从 Visual Studio 内部调试 MVC 5 应用程序，但可以再次将 Web 应用发布到 Azure，以直接从移动浏览器或浏览器模拟器验证实时 Web 应用。

显示内容在移动设备上一目了然。你可能还看到了 Bootstrap CSS 框架应用的一些视觉效果。单击“ASP.NET”链接。

![][SessionsByTagASP.NET]

ASP.NET 标记视图已根据屏幕大小缩放，这是 Bootstrap 自动为你调整的。但是，你可以改进此视图，以更好地适应移动浏览器。例如，以便能够轻松读取“日期”列。本教程的随后部分，你将更改 *AllTags* 视图，使其更适合移动应用。

##<a name="bkmk_bootstrap"></a> Bootstrap CSS 框架

Bootstrap 支持是 MVC 5 模板中内置的新功能。你已经看到了它如何立即改进应用程序中的不同视图。例如，当浏览器宽度较小时，顶部导航栏可自动折叠。在桌面浏览器中，尝试调整浏览器窗口的大小，并了解导航栏如何改变其外观。这是 Bootstrap 内置的响应式 Web 设计。

若要在没有 Bootstrap 的情况下查看 Web 应用的外观，请打开 *App\_Start\\BundleConfig.cs* 并注释掉包含的行 *bootstrap.js* 和 *bootstrap.css*。以下代码显示了更改后 `RegisterBundles` 方法的两个语句：

     bundles.Add(new ScriptBundle("~/bundles/bootstrap").Include(
              //"~/Scripts/bootstrap.js",
              "~/Scripts/respond.js"));

    bundles.Add(new StyleBundle("~/Content/css").Include(
              //"~/Content/bootstrap.css",
              "~/Content/site.css"));

按 `Ctrl+F5` 运行应用程序。

可以看到，可折叠的导航栏现在只是一个普通的未排序列表。再次单击“按标记浏览”，然后单击“ASP.NET”。在移动模拟器视图中，你可以看到该应用程序现在不再根据屏幕大小缩放，而你必须侧向滚动才能看到表的右侧。

![][SessionsByTagASP.NETNoBootstrap]

撤消所做的更改并刷新移动浏览器，以适合移动应用的显示画面是否已恢复。

Bootstrap 并不特定于 ASP.NET MVC 5，你可以在任何 Web 应用程序上利用这些功能。但是，它现已内置到 ASP.NET MVC 5 项目模板中，因此，MVC 5 Web 应用程序可按默认利用 Bootstrap。

有关 Bootstrap 的详细信息，请转到 [Bootstrap][BootstrapSite] 站点。

在下一部分中，你将了解如何提供移动浏览器特定的视图。

##<a name="bkmk_overrideviews"></a> 重写视图、布局和分部视图

你可以重写一般性移动浏览器、单个移动浏览器或任何特定浏览器的任何视图（包括布局和分部视图）。若要提供移动特定的视图，你可以复制视图文件并在文件名中添加 *.Mobile*。例如，若要创建移动 *Index* 视图，可将 *Views\\Home\\Index.cshtml* 复制到 *Views\\Home\\Index.Mobile.cshtml*。

在本节中，你将创建一个移动特定布局文件。

若要开始，请将 *Views\\Shared\\_Layout.cshtml* 复制到 *Views\\Shared\\_Layout.Mobile.cshtml*。打开 *\_Layout.Mobile.cshtml*，并将标题从“MVC5 应用程序”更改为“MVC5 应用程序 (Mobile)”。

在导航栏的每个 `Html.ActionLink` 调用中，删除每个链接 *ActionLink* 中的“浏览者”。以下代码显示移动布局文件的已完成 `<ul class="nav navbar-nav">` 标记。

    <ul class="nav navbar-nav">
        <li>@Html.ActionLink("Home", "Index", "Home")</li>
        <li>@Html.ActionLink("Date", "AllDates", "Home")</li>
        <li>@Html.ActionLink("Speaker", "AllSpeakers", "Home")</li>
        <li>@Html.ActionLink("Tag", "AllTags", "Home")</li>
    </ul>

将 *Views\\Home\\AllTags.cshtml* 文件复制到 *Views\\Home\\AllTags.Mobile.cshtml*。打开此新文件，将 `<h2>` 元素从“Tags”更改为“Tags (M)”：

    <h2>Tags (M)</h2>

使用桌面浏览器和移动浏览器模拟器浏览到标签页。移动浏览器模拟器将显示你所做的两项更改（更改了标题 *\_Layout.Mobile.cshtml* 和标题 *AllTags.Mobile.cshtml*）。

![][AllTagsMobile_LayoutMobile]

相比之下，桌面显示并未变化（使用 *\_Layout.cshtml* 和 *AllTags.cshtml* 中的标题）。

![][AllTagsMobile_LayoutMobileDesktop]

##<a name="bkmk_browserviews"></a> 创建浏览器特定的视图

除了移动特定和桌面特定的视图以外，你还可以为单个浏览器创建视图。例如，你可以创建专门针对 iPhone 或 Android 浏览器的视图。在此部分中，你将为 iPhone 浏览器和 iPhone 版本的 *AllTags* 视图创建布局。

打开 *Global.asax* 文件，并将以下代码添加到 `Application_Start` 方法的底部。

    DisplayModeProvider.Instance.Modes.Insert(0, new DefaultDisplayMode("iPhone")
    {
        ContextCondition = (context => context.GetOverriddenUserAgent().IndexOf
            ("iPhone", StringComparison.OrdinalIgnoreCase) >= 0)
    });

此代码定义要与每个传入请求匹配的名为“iPhone”的新显示模式。如果传入请求与定义的条件（即，如果用户代理包含字符串“iPhone”）匹配，则 ASP.NET MVC 将查找名称包含“iPhone”后缀的视图。

>[AZURE.NOTE] 在添加特定于移动浏览器的显示模式（例如，用于 iPhone 和 Android）时，请务必将第一个参数设置为 `0`（在列表顶部插入），以确保浏览器特定模式优先于移动模板 (*.Mobile.cshtml)。如果移动模板位于列表顶部，则会选择该移动模板而不是你预期的显示模式（第一个匹配项优先，而移动模板与所有移动浏览器匹配）。

在代码中右键单击 `DefaultDisplayMode`，选择“解析”，然后选择 `using System.Web.WebPages;`。这会向 `System.Web.WebPages` 命名空间添加引用，该命名空间中定义了 `DisplayModeProvider` 和 `DefaultDisplayMode` 类型。

![][ResolveDefaultDisplayMode]

或者，也可以简单地将以下行手动添加到文件的 `using` 节。

    using System.Web.WebPages;

保存更改。请将 *Views\\Shared\\_Layout.Mobile.cshtml* 文件复制到 *Views\\Shared\\_Layout.iPhone.cshtml*。打开新文件，然后将标题从 `MVC5 Application (Mobile)` 更改为 `MVC5 Application (iPhone)`。

将 *Views\\Home\\AllTags.Mobile.cshtml* 文件复制到 *Views\\Home\\AllTags.iPhone.cshtml*。在此新文件中，将 `<h2>` 元素从“Tags (M)”更改为“Tags (iPhone)”。

运行应用程序。运行移动浏览器模拟器，确保其用户代理设置为“iPhone”，然后浏览到 *AllTags* 视图。如果你在 Internet Explorer 11 F12 开发人员工具中使用模拟器，请将模拟设置为：

-   浏览器配置文件 = **Windows Phone**
-   用户代理字符串 = **Custom**
-   自定义字符串 = **Apple-iPhone5C1/1001.525**

以下屏幕截图显示了使用自定义用户代理字符串（这是一个 iPhone 5C 用户代理字符串）在 Internet Explorer 11 F12 开发人员工具中的模拟器中呈现的 *AllTags* 视图。

![][AllTagsIPhone_LayoutIPhone]

在移动浏览器中，选择“发言人”链接。因为没有移动视图 (*AllSpeakers.Mobile.cshtml*)，所以使用移动布局视图 (*AllSpeakers.cshtml*) 呈现默认发言人内容 (*\_Layout.Mobile.cshtml*)。如下所示， *\_Layout.Mobile.cshtml* 中定义了标题“MVC5 应用程序 (Mobile)”。

![][AllSpeakers_LayoutMobile]

通过在 *Views\\_ViewStart.cshtml* 文件中将 `RequireConsistentDisplayMode` 设置为 `true`，可以全局禁止默认（非移动）视图在移动布局内呈现，如下所示：

    @{
        Layout = "~/Views/Shared/_Layout.cshtml";
        DisplayModeProvider.Instance.RequireConsistentDisplayMode = true;
    }

当 `RequireConsistentDisplayMode` 设置为 `true` 时，移动布局 (*\_Layout.Mobile.cshtml*) 只用于移动视图。（即，视图文件仅为 ***ViewName**.Mobile.cshtml* 形式。）你可能需要将 `RequireConsistentDisplayMode` 设置为 `true`（如果你的移动布局不太适合你的非移动视图）。下面的屏幕截图显示当 `RequireConsistentDisplayMode` 设置为 `true` 时，如何呈现 *Speakers* 页面（顶部导航栏中没有字符串“(Mobile)”）。

![][AllSpeakers_LayoutMobileOverridden]

你可以通过在特定视图文件中将 `RequireConsistentDisplayMode` 设置为 `false` 来禁用视图中一致的显示模式。 *Views\\Home\\AllSpeakers.cshtml* 文件中的以下标记将 `RequireConsistentDisplayMode` 设置为 `false`：

    @model IEnumerable<string>

    @{
        ViewBag.Title = "All speakers";
        DisplayModeProvider.Instance.RequireConsistentDisplayMode = false;
    }

在本部分中，我们已了解如何创建移动布局和视图，以及如何为特定的设备（如 iPhone）创建布局和视图。但是，Bootstrap CSS 框架的主要优势是响应式布局，也就是说，可以跨桌面、电话和平板电脑浏览器应用单个样式表，以创建一致的外观。在下一部分中，你将了解如何利用 Bootstrap 来创建适合移动的视图。

##<a name="bkmk_Improvespeakerslist"></a>改进发言人列表

正如你刚才看到的，“发言人”视图虽然可读，但链接字迹小，不易在移动设备上点击。在本部分中，你将使 *AllSpeakers* 视图适合移动应用，显示较大的便于点按的链接，并包含一个搜索框，用于快速查找发言人。

你可以使用 Bootstrap [链接列表组][]样式来改进“发言人”视图。在 *Views\\Home\\AllSpeakers.cshtml* 中，将 Razor 文件的内容替换为以下代码。

     @model IEnumerable<string>

    @{
        ViewBag.Title = "All Speakers";
    }

    <h2>Speakers</h2>

    <div class="list-group">
        @foreach (var speaker in Model)
        {
            @Html.ActionLink(speaker, "SessionsBySpeaker", new { speaker }, new { @class = "list-group-item" })
        }
    </div>

`<div>` 标记中的 `class="list-group"` 属性将应用 Bootstrap 列表样式，`class="input-group-item"` 属性将向每个链接应用 Bootstrap 列表项样式。

刷新移动浏览器。更新的视图如下所示：

![][AllSpeakersFixed]

Bootstrap [链接列表组][]样式使每个链接的整个框可单击，这大大改进了用户体验。切换到桌面视图，可以看到一致的外观。

![][AllSpeakersFixedDesktop]

尽管移动浏览器视图得到了改进，但很难在较长的发言人列表中导航。Bootstrap 未提供现成的搜索筛选器功能，但你只需使用几行代码就能添加此功能。首先，将一个搜索框添加到视图，然后与筛选函数的 JavaScript 代码相挂接。在 *Views\\Home\\AllSpeakers.cshtml* 中，将 <form\> 标记添加到 <h2\> 标记的后面，如下所示：

    @model IEnumerable<string>

    @{
        ViewBag.Title = "All Speakers";
    }

    <h2>Speakers</h2>

    <form class="input-group">
        <span class="input-group-addon"><span class="glyphicon glyphicon-search"></span></span>
        <input type="text" class="form-control" placeholder="Search speaker">
    </form>
    <br />
    <div class="list-group">
        @foreach (var speaker in Model)
        {
            @Html.ActionLink(speaker, 
                             "SessionsBySpeaker", 
                             new { speaker }, 
                             new { @class = "list-group-item" })
        }
    </div>

请注意，`<form>` 和 `<input>` 标记都应用了 Bootstrap 样式。`<span>` 元素用于将 Bootstrap [glyphicon][] 添加到搜索框。

在 *脚本* 文件夹中，添加一个名为 *filter.js* 的 JavaScript 文件。打开该文件并在其中粘贴以下代码：

    $(function () {

        // reset the search form when the page loads
        $("form").each(function () {
            this.reset();
        });

        // wire up the events to the <input> element for search/filter
        $("input").bind("keyup change", function () {
            var searchtxt = this.value.toLowerCase();
            var items = $(".list-group-item");

            // show all speakers that begin with the typed text and hide others
            for (var i = 0; i < items.length; i++) {
                var val = items[i].text.toLowerCase();
                val = val.substring(0, searchtxt.length);
                if (val == searchtxt) {
                    $(items[i]).show();
                }
                else {
                    $(items[i]).hide();
                }
            }
        });
    });

你还需要在注册的绑定中包含 filter.js。打开 *App\_Start\\BundleConfig.cs* 并更改第一个捆绑。更改第一个 `bundles.Add` 语句（用于 **jquery** 捆绑），以包含 *Scripts\\filter.js* ，如下所示：

     bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                "~/Scripts/jquery-{version}.js",
                "~/Scripts/filter.js"));

**jquery** 捆绑已由默认的 *\_Layout* 视图呈现。稍后，你可以利用相同的 JavaScript 代码向其他列表视图应用筛选器功能。

刷新移动浏览器并转到 *AllSpeakers* 视图。在搜索框中，键入“sc”。现在，应已根据你的搜索字符串筛选发言人列表。

![][AllSpeakersFixedSearchBySC]

##<a name="bkmk_improvetags"></a> 改进标签列表

与默认“发言人”视图一样，“标签”视图虽然可读，但链接字迹小，不易在移动设备上点击。如果你使用前面所述的代码更改，你可以使用与修复“发言人”视图相同的方式来修复“标签”视图，但是需要在 *Views\\Home\\AllTags.cshtml* 中使用以下 `Html.ActionLink` 方法语法：

    @Html.ActionLink(tag, 
                     "SessionsByTag", 
                     new { tag }, 
                     new { @class = "list-group-item" })

刷新的桌面浏览器将如下所示：

![][AllTagsFixedDesktop]

刷新的移动浏览器将如下所示：

![][AllTagsFixed]

>[AZURE.NOTE] 如果你发现移动浏览器仍然使用了原始列表格式，并奇怪正常的 Bootstrap 样式为何会发生这种情况，则需要知道，这是前面创建移动特定视图后产生的效果。但是，现在你要使用 Bootstrap CSS 框架来创建响应式 Web 设计，并继续删除这些移动特定的视图和移动特定的布局视图。完成此操作后，刷新的移动浏览器将显示 Bootstrap 样式。

##<a name="bkmk_improvedates"></a> 改进日期列表

如果你使用前面所述的代码更改，你可以使用与改进“发言人”视图和“标签”视图相同的方式来修复“日期”视图，但是需要在 *Views\\Home\\AllDates.cshtml* 中使用以下 `Html.ActionLink` 方法语法：

    @Html.ActionLink(date.ToString("ddd, MMM dd, h:mm tt"), 
                     "SessionsByDate", 
                     new { date }, 
                     new { @class = "list-group-item" })

你将获得如下所示的已刷新移动浏览器视图：

![][AllDatesFixed]

你可以通过按日期组织日期时间值来进一步改进“日期”视图。这可以使用 Bootstrap [面板][]样式来实现。将 *Views\\Home\\AllDates.cshtml* 文件的内容替换为下列代码：

    @model IEnumerable<DateTime>

    @{
        ViewBag.Title = "All Dates";
    }

    <h2>Dates</h2>

    @foreach (var dategroup in Model.GroupBy(x=>x.Date))
    {
        <div class="panel panel-primary">
            <div class="panel-heading">
                @dategroup.Key.ToString("ddd, MMM dd")
            </div>
            <div class="panel-body list-group">
                @foreach (var date in dategroup)
                {
                    @Html.ActionLink(date.ToString("h:mm tt"), 
                                     "SessionsByDate", 
                                     new { date }, 
                                     new { @class = "list-group-item" })
                }
            </div>
        </div>
    }

此代码为列表中的每个非重复日期创建一个单独 `<div class="panel panel-primary">` 标记，并像前面一样为相应的链接使用[链接列表组][]。当此代码运行时，移动浏览器看起来像这样：

![][AllDatesFixed2]

切换到桌面浏览器。同样，可以看到一致的外观。

![][AllDatesFixed2Desktop]

##<a name="bkmk_improvesessionstable"></a> 改进 SessionsTable 视图

在此部分中，你将使 *SessionsTable* 视图更适合移动应用。此项更改比前面的更改更宽泛。

在移动浏览器中，点击“标记”按钮，然后在搜索框中输入 **asp**。

![][AllTagsFixedSearchByASP]

点击“ASP.NET”链接。

![][SessionsTableTagASP.NET]

正如你所看到的，显示的内容采用表格式，这种格式当前设计为在桌面浏览器中查看。但是，在移动浏览器中，这种格式有点难于阅读。为解决这些问题，请打开 *Views\\Home\\SessionsTable.cshtml* ，然后用以下代码替换该文件的内容：

    @model IEnumerable<Mvc5Mobile.Models.Session>

    <h2>@ViewBag.Title</h2>

    <div class="container">
        <div class="row">
            @foreach (var session in Model)
            {
                <div class="col-md-4">
                    <div class="list-group">
                        @Html.ActionLink(session.Title, 
                                         "SessionByCode", 
                                         new { session.Code }, 
                                         new { @class="list-group-item active" })
                        <div class="list-group-item">
                            <div class="list-group-item-text">
                                @Html.Partial("_SpeakersLinks", session)
                            </div>
                            <div class="list-group-item-info">
                                @session.DateText
                            </div>
                            <div class="list-group-item-info small hidden-xs">
                                @Html.Partial("_TagsLinks", session)
                            </div>
                        </div>
                    </div>
                </div>
            }
        </div>
    </div>

该代码执行 3 项操作：

-   使用 Bootstrap [自定义链接列表组][]将会话信息设为竖向，这样即可在移动浏览器中阅读所有此类信息（使用 list-group-item-text 等类）
-   将[网格系统][]应用到布局，使会话项在桌面浏览器中横向排布，在移动浏览器中纵向排布（使用 col-md-4 类）
-   使用[响应式实用工具][]，以便在移动浏览器中查看时隐藏会话标记（使用 hidden-xs 类）

你还可以点击标题链接转到相应的会话。下图反映了代码更改。

![][FixedSessionsByTag]

自动应用的 Bootstrap 网格系统将在移动浏览器中纵向排列会话。此外请注意，标记未显示。切换到桌面浏览器。

![][SessionsTableFixedTagASP.NETDesktop]

在桌面浏览器中，请注意标记现在已显示。此外，你可以看到，应用的 Bootstrap 网格系统会在两列中排列会话项。如果放大浏览器，就会看到，排列方式更改为三列。

##<a name="bkmk_improvesessionbycode"></a> 改进 SessionByCode 视图

最后，需要修复 *SessionByCode* 视图，使其适合移动应用。

在移动浏览器中，点击“标记”按钮，然后在搜索框中输入 **asp**。

![][AllTagsFixedSearchByASP]

点击“ASP.NET”链接。将显示 ASP.NET 标记的会话。

![][FixedSessionsByTag]

选择“使用 ASP.NET 和 AngularJS 生成单页”链接。

![][SessionByCode3-644]

默认桌面视图虽然不错，但你可以通过使用一些 Bootstrap GUI 组件轻松改进外观。

打开 *Views\\Home\\SessionByCode.cshtml* 并将内容替换为以下标记：

    @model Mvc5Mobile.Models.Session

    @{
        ViewBag.Title = "Session details";
    }
    <h3>@Model.Title (@Model.Code)</h3>
    <p>
        <strong>@Model.DateText</strong> in <strong>@Model.Room</strong>
    </p>

    <div class="panel panel-primary">
        <div class="panel-heading">
            Speakers
        </div>
        @foreach (var speaker in Model.Speakers)
        {
            @Html.ActionLink(speaker, 
                             "SessionsBySpeaker", 
                             new { speaker }, 
                             new { @class="panel-body" })
        }
    </div>

    <p>@Model.Abstract</p>

    <div class="panel panel-primary">
        <div class="panel-heading">
            Tags
        </div>
        @foreach (var tag in Model.Tags)
        {
            @Html.ActionLink(tag, 
                             "SessionsByTag", 
                             new { tag }, 
                             new { @class = "panel-body" })
        }
    </div>

新的标记使用 Bootstrap 面板样式改进移动视图。

刷新移动浏览器。下图反映你刚才所做的代码更改：

![][SessionByCodeFixed3-644]

## 总结和回顾

本教程说明如何使用 ASP.NET MVC 5 开发适合移动应用的 Web 应用程序。其中包括：

-	将 ASP.NET MVC 5 应用程序部署到 Azure Web 应用
-   使用 Bootstrap 在 MVC 5 应用程序中创建响应式 Web 布局
-   在全局以及针对单个视图重写布局、视图和分部视图
-   使用 `RequireConsistentDisplayMode` 属性控制布局和分步重写的实施
-   创建面向特定浏览器（如 iPhone 浏览器）的视图
-   在 Razor 代码中应用 Bootstrap 样式

## 另请参阅

-   [响应式 Web 设计的 9 项基本原则](http://blog.froont.com/9-basic-principles-of-responsive-web-design/)
-   [Bootstrap][BootstrapSite]
-   [Bootstrap 官方博客][]
-   [Tutorial Republic 中的 Bootstrap Twitter 教程][]
-   [Bootstrap 演练中心][]
-   [W3C 建议移动 Web 应用程序的最佳做法][]
-   [用于媒体查询的 W3C 候选建议方案][]

<!-- Internal Links -->
[Deploy the starter project to an Azure web app]: #bkmk_DeployStarterProject
[Bootstrap CSS 框架]: #bkmk_bootstrap
[重写视图、布局和分部视图]: #bkmk_overrideviews
[Create Browser-Specific Views]: #bkmk_browserviews
[改进发言人列表]: #bkmk_Improvespeakerslist
[改进标签列表]: #bkmk_improvetags
[改进日期列表]: #bkmk_improvedates
[改进 SessionsTable 视图]: #bkmk_improvesessionstable
[改进 SessionByCode 视图]: #bkmk_improvesessionbycode

<!-- External Links -->
[Visual Studio Express 2013]: http://www.visualstudio.com/downloads/download-visual-studio-vs#d-express-web
[Visual Studio 2015]: https://www.visualstudio.com/downloads/download-visual-studio-vs
[AzureSDKVs2013]: https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids
[Fiddler]: http://www.fiddler2.com/fiddler2/
[EmulatorIE11]: http://msdn.microsoft.com/zh-cn/library/ie/dn255001.aspx
[EmulatorChrome]: https://developers.google.com/chrome-developer-tools/docs/mobile-emulation
[EmulatorOpera]: http://www.opera.com/developer/tools/mobile/
[StarterProject]: http://go.microsoft.com/fwlink/?LinkID=398780&clcid=0x409
[CompletedProject]: http://go.microsoft.com/fwlink/?LinkID=398781&clcid=0x409
[BootstrapSite]: http://getbootstrap.com/
[WebPIAzureSdk23NetVS13]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/WebPIAzureSdk23NetVS13.png
[链接列表组]: http://getbootstrap.com/components/#list-group-linked
[glyphicon]: http://getbootstrap.com/components/#glyphicons
[面板]: http://getbootstrap.com/components/#panels
[自定义链接列表组]: http://getbootstrap.com/components/#list-group-custom-content
[网格系统]: http://getbootstrap.com/css/#grid
[响应式实用工具]: http://getbootstrap.com/css/#responsive-utilities
[Bootstrap 官方博客]: http://blog.getbootstrap.com/
[Tutorial Republic 中的 Bootstrap Twitter 教程]: http://www.tutorialrepublic.com/twitter-bootstrap-tutorial/
[Bootstrap 演练中心]: http://www.bootply.com/
[W3C 建议移动 Web 应用程序的最佳做法]: http://www.w3.org/TR/mwabp/
[用于媒体查询的 W3C 候选建议方案]: http://www.w3.org/TR/css3-mediaqueries/

<!-- Images -->
[DeployClickPublish]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-1.png
[DeployClickWebSites]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-2.png
[DeploySignIn]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-3.png
[DeployUsername]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-4.png
[DeployPassword]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-5.png
[DeployNewWebsite]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-6.png
[DeploySiteSettings]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-7.png
[DeployPublishSite]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/deploy-to-azure-website-8.png
[MobileHomePage]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/mobile-home-page.png
[FixedSessionsByTag]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionsByTag-ASP.NET-Fixed.png
[AllTags]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTags.png
[SessionsByTagASP.NET]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionsByTag-ASP.NET.png
[SessionsByTagASP.NETNoBootstrap]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionsByTag-ASP.NET-NoBootstrap.png
[AllTagsMobile_LayoutMobile]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTagsMobile-_LayoutMobile.png
[AllTagsMobile_LayoutMobileDesktop]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTagsMobile-_LayoutMobile-Desktop.png
[ResolveDefaultDisplayMode]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/Resolve-DefaultDisplayMode.png
[AllTagsIPhone_LayoutIPhone]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTagsIPhone-_LayoutIPhone.png
[AllSpeakers_LayoutMobile]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllSpeakers-_LayoutMobile.png
[AllSpeakers_LayoutMobileOverridden]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllSpeakers-_LayoutMobile-Overridden.png
[AllSpeakersFixed]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllSpeakers-Fixed.png
[AllSpeakersFixedDesktop]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllSpeakers-Fixed-Desktop.png
[AllSpeakersFixedSearchBySC]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllSpeakers-Fixed-SearchBySC.png
[AllTagsFixedDesktop]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTags-Fixed-Desktop.png
[AllTagsFixed]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTags-Fixed.png
[AllDatesFixed]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllDates-Fixed.png
[AllDatesFixed2]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllDates-Fixed2.png
[AllDatesFixed2Desktop]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllDates-Fixed2-Desktop.png
[AllTagsFixedSearchByASP]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/AllTags-Fixed-SearchByASP.png
[SessionsTableTagASP.NET]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionsTable-Tag-ASP.NET.png
[SessionsTableFixedTagASP.NETDesktop]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionsTable-Fixed-Tag-ASP.NET-Desktop.png
[SessionByCode3-644]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionByCode-3-644.png
[SessionByCodeFixed3-644]: ./media/web-sites-dotnet-deploy-aspnet-mvc-mobile-app/SessionByCode-Fixed-3-644.png
 

<!---HONumber=Mooncake_0215_2016-->