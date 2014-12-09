<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="Azure 网站和 ASP.NET 入门" authors="tdykstra" solutions="" manager="wpickett" editor="mollybos" />

# Azure 网站和 ASP.NET 入门

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/net/tutorials/get-started/" title="Visual Studio 2013">Visual Studio 2013</a><a href="/zh-cn/develop/net/tutorials/get-started-vs2012/" title="Visual Studio 2012" class="current">Visual Studio 2012</a></div>

<div class="dev-callout"><strong>说明</strong><p><a href="/zh-cn/develop/net/tutorials/get-started/">本教程较新版本</a>现已推出。如果您使用的是 Visual Studio 2012，则仍可按本版教程进行学习，但其中不会介绍所有最新的 Azure SDK 功能。</p></div>

本教程将向您介绍如何使用 Visual Studio 2012 或 Visual Studio 2012 for Web Express 中的“发布 Web”向导将 ASP.NET Web 应用程序部署到 Azure 网站。如果您愿意，可以使用 Visual Studio 2010 或 Visual Web Developer Express 2010 按照教程中的步骤进行操作。

您可以免费注册一个 Azure 帐户，而且，如果您还没有 Visual Studio 2012，则此 SDK 会自动安装 Visual Studio 2012 for Web Express。这样您就能够完全免费地开始针对 Azure 进行开发了。

本教程假定您之前未使用过 Azure。完成本教程之后，您将能够在云中启动并运行简单的 Web 应用程序。

您将了解到以下内容：

-   如何通过安装 Azure SDK 以支持计算机进行 Azure 开发。
-   如何创建 Visual Studio ASP.NET MVC 4 项目并将其发布到 Azure 网站。

下图演示了完整的应用程序：

![网站示例][网站示例]

<div class="dev-callout"><p><strong>请注意</strong>若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，则可以<a href="/zh-cn/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F" target="_blank">激活 MSDN 订户权益</a>或<a href="/zh-cn/pricing/free-trial/?WT.mc_id=A261C142F" target="_blank">注册免费试用</a>。</p></div>

### 教程章节

1.  [设置开发环境][设置开发环境]
2.  [在 Azure 中创建网站][在 Azure 中创建网站]
3.  [创建 ASP.NET MVC 4 应用程序][创建 ASP.NET MVC 4 应用程序]
4.  [将应用程序部署到 Azure][将应用程序部署到 Azure]
5.  [后续步骤][后续步骤]

[WACOM.INCLUDE [install-sdk-2012-only](../includes/install-sdk-2012-only.md)]

## <a name="setupwindowsazure"></a><span class="short-header">创建站点</span>创建网站

下一步是创建 Azure 网站。

1.  在 [Azure 管理门户][Azure 管理门户] 中，单击“网站”，然后单击“新建”。

![新建网站][新建网站]

1.  单击“快速创建”。

    ![快速创建][快速创建]

2.  在该向导的“创建网站”步骤中，在“URL”框中输入将用作您的应用程序的唯一 URL 的字符串。完整 URL 将包含您在此处输入的内容和您在文本框旁边看到的后缀。图中显示**示例 1**，但如果已经有人将该字符串用于 URL，则您需要另外输入一个值。

3.  在“区域”下拉列表中，选择离你最近的区域。
    此设置指定您的网站将在哪个数据中心运行。

4.  单击“创建网站”箭头。

    ![创建新网站][创建新网站]

    管理门户返回到“网站”页面，“状态”列显示正在创建网站。稍后（通常不到一分钟），“状态”列会显示已成功创建网站。在左侧导航栏中，您的帐户中的站点数将显示在“网站”图标的旁边。

    ![管理门户的网站页面，网站已创建][管理门户的网站页面，网站已创建]

## <a name="createmvc4app"></a><span class="short-header">创建应用程序</span>创建 ASP.NET MVC 4 应用程序

您已经创建了一个 Azure 网站，但其中还没有内容。下一步将创建要发布到 Azure 的 Visual Studio Web 应用程序。

### 创建项目

1.  启动 Visual Studio 2012 或 Visual Studio 2012 for Web Express。

2.  在“文件”菜单中，单击“新建”，然后单击“项目”。

![“文件”菜单中的“新建项目”][“文件”菜单中的“新建项目”]

1.  在“新建项目”对话框中，展开“C#”并在“已安装的模板”下选择“Web”，然后选择“ASP.NET MVC 4 Web 应用程序”。

2.  确保将 **.NET Framework 4.5** 选为目标框架。

3.  将该应用程序命名为 **MyExample**，然后单击“确定”。

![“新建项目”对话框][“新建项目”对话框]

1.  在“新建 ASP.NET MVC 4 项目”对话框中，选择“Internet 应用程序”模板并单击“确定”**OK**。

![新建 ASP.NET MVC 4 项目对话框][新建 ASP.NET MVC 4 项目对话框]

### 在本地运行应用程序

1.  按 **CTRL**+**F5** 运行应用程序。随后在默认浏览器中显示该应用程序主页。

![正在本地运行的网站][正在本地运行的网站]

以上是创建要部署到 Azure 的简单应用程序所需执行的全部步骤。

## <a name="deploytowindowsazure"></a><span class="short-header">部署应用程序</span>将应用程序部署到 Azure

1.  在 Visual Studio 中，在“解决方案资源管理器”中右键单击该项目，从上下文菜单中选择“发布”。

    ![项目上下文菜单中的“发布”][项目上下文菜单中的“发布”]

“发布 Web”向导将打开。

1.  在“发布 Web”向导的“配置文件”选项卡中，单击“导入”。

    ![导入发布设置][导入发布设置]

“导入发布配置文件”对话框随即出现。

1.  如果您之前未在 Visual Studio 中添加 Azure 订阅，请执行下列步骤。在这些步骤中，您将添加订阅，以便“从 Azure 网站导入”下的下拉列表中包含您的网站。

    -   在“导入发布配置文件”对话框中，单击“从 Azure 网站导入”，然后单击“添加 Azure 订阅”。

    ![添加 Azure 订阅][添加 Azure 订阅]

    -   在“导入 Azure 订阅”对话框中，单击“下载订阅文件”。

    ![下载订阅文件][下载订阅文件]

    -   在浏览器窗口中，保存 *.publishsettings* 文件。

    ![下载 .publishsettings 文件][下载 .publishsettings 文件]

    -   在“导入 Azure 订阅”对话框中，单击“浏览”并导航到 *.publishsettings* 文件。

    ![下载订阅][下载订阅]

    -   单击“导入”。

    ![导入][导入]

    > [WACOM.NOTE]
    > .publishsettings 文件中包含您的凭据（未编码），这些凭据用来管理您的 Azure 订阅和服务。确保此文件安全的最佳做法是，将其暂时存储在您的源目录的外部（例如存储在 Libraries\\Documents 文件夹中），然后在完成导入后将其删除。获得了 .publishsettings 文件访问权的恶意用户可以编辑、创建和删除您的 Azure 服务。

2.  在“导入发布配置文件”对话框中，选择“从 Azure 网站导入”，再从下拉列表中选择您的网站，然后单击“确定”。

    ![导入发布配置文件][导入发布配置文件]

3.  在“连接”选项卡中，单击“验证连接”以确保设置正确。

    ![验证连接][验证连接]

4.  在连接通过验证后，“验证连接”按钮旁边会出现一个绿色复选标记。单击“下一步”。

    ![验证成功的连接][验证成功的连接]

5.  在“设置”选项卡中，取消选中“在运行时使用此连接字符串”选项，因为此应用程序不使用数据库。对于此页上的其余项，您可以接受其默认设置。您正在部署“发布”生成配置，因此不需要在目标服务器上删除文件、预编译应用程序或排除“App\_Data”文件夹中的文件。
    单击“下一步”。

    ![“设置”选项卡][“设置”选项卡]

6.  在“预览”选项卡中，单击“开始预览”。

    ![“预览”选项卡中的“开始预览”按钮][“预览”选项卡中的“开始预览”按钮]

    该选项卡会显示将复制到服务器的文件的列表。显示预览并不是发布应用程序所必需的，但它是一个需要了解的很有用的功能。在本例中，您不需要对显示的文件列表执行任何操作。

    ![“开始预览”文件输出][“开始预览”文件输出]

7.  单击“发布”。Visual Studio 开始执行将文件复制到 Azure 服务器的过程。

8.  “输出”窗口将显示已执行的部署操作并报告已成功完成部署。

    ![报告部署成功的输出窗口][报告部署成功的输出窗口]

9.  成功完成部署后，默认浏览器将自动打开指向已部署网站的 URL。
    您创建的应用程序现在运行于云中。

    ![在 Azure 中运行的网站][网站示例]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

在本教程中，您已了解如何将简单的 Web 应用程序部署到 Azure 网站。其中还提供了其他资源，旨在向您说明如何对网站进行管理、缩放以及故障排除，如何添加数据库、身份验证和授权功能以及如何决定是否应在 Azure 云服务中而不是 Azure 网站中运行您的应用程序。

### 如何管理网站

在使用完网站后，可以将其删除，有时您可能要暂时使其脱机或更改网站设置。可以从 Visual Studio 中的“服务器资源管理器”中执行其中一些功能。

![服务器资源管理器中的 Azure 网站][服务器资源管理器中的 Azure 网站]

![Visual Studio 中的网站配置][Visual Studio 中的网站配置]

若要删除网站，您可使用 Azure 管理门户。以下屏幕快照显示了管理门户的“仪表板”选项卡中的“停止”、“重新启动”和“删除”按钮。

![管理门户仪表板选项卡][管理门户仪表板选项卡]

可以在“配置”选项卡上更改站点设置。有关更多信息，请参见[如何管理网站][如何管理网站]。

### 如何缩放网站

随着您的网站公开并开始获得更多流量，响应时间可能会增加。若要改善此情况，您可以轻松地在管理门户的“缩放”选项卡中添加服务器资源。

![管理门户缩放选项卡][管理门户缩放选项卡]

有关更多信息，请参见[如何缩放网站][如何缩放网站]。（通过添加服务器资源来缩放网站不是免费的。）

### 如何对网站进行故障排除

您可能需要查看跟踪或日志输出来寻求帮助以进行故障排除。Visual Studio 提供了内置工具，使您能够在 Azure 日志生成时轻松实时地对其进行查看。

![Visual Studio 中的日志][Visual Studio 中的日志]

有关更多信息，请参见[在 Visual Studio 中排除 Windows Azure 网站的故障][在 Visual Studio 中排除 Windows Azure 网站的故障]。

### 如何添加数据库和授权功能

大多数生产网站使用数据库并且只允许特定授权用户访问某些网站功能。有关说明如何开始使用数据库访问、身份验证和授权的教程，请参见[使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站][使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站]。

### 如何决定应用程序是否应在云服务中运行

在某些方案中，您可能需要在 Azure 云服务中而不是 Azure 网站中运行应用程序。有关更多信息，请参见 [Azure 执行模型][Azure 执行模型]和 [Azure 网站、 云服务和虚拟机比较][Azure 网站、 云服务和虚拟机比较]。有关说明如何创建多层 ASP.NET Web 应用程序并将其部署到云服务的系列教程，请参见[使用存储表、队列和 Blob 的 .NET 多层应用程序][使用存储表、队列和 Blob 的 .NET 多层应用程序]。

  [网站示例]: ./media/web-sites-dotnet-get-started-vs2012/DeployedWebSite.png
  [设置开发环境]: #setupdevenv
  [在 Azure 中创建网站]: #setupwindowsazure
  [创建 ASP.NET MVC 4 应用程序]: #createmvc4app
  [将应用程序部署到 Azure]: #deploytowindowsazure
  [后续步骤]: #nextsteps
  [Azure 管理门户]: http://manage.windowsazure.com
  [新建网站]: ./media/web-sites-dotnet-get-started-vs2012/WebSiteNew.png
  [快速创建]: ./media/web-sites-dotnet-get-started-vs2012/ClickQuickCreate.png
  [创建新网站]: ./media/web-sites-dotnet-get-started-vs2012/CreateWebsite.png
  [管理门户的网站页面，网站已创建]: ./media/web-sites-dotnet-get-started-vs2012/WebSiteStatusRunning.png
  [“文件”菜单中的“新建项目”]: ./media/web-sites-dotnet-get-started-vs2012/NewVSProject.png
  [“新建项目”对话框]: ./media/web-sites-dotnet-get-started-vs2012/NewMVC4WebApp.png
  [新建 ASP.NET MVC 4 项目对话框]: ./media/web-sites-dotnet-get-started-vs2012/InternetAppTemplate.png
  [正在本地运行的网站]: ./media/web-sites-dotnet-get-started-vs2012/AppRunningLocally.png
  [项目上下文菜单中的“发布”]: ./media/web-sites-dotnet-get-started-vs2012/PublishVSSolution.png
  [导入发布设置]: ./media/web-sites-dotnet-get-started-vs2012/ImportPublishSettings.png
  [添加 Azure 订阅]: ./media/web-sites-dotnet-get-started-vs2012/rzAddWAsub.png
  [下载订阅文件]: ./media/web-sites-dotnet-get-started-vs2012/rzDownLoadDownload.png
  [下载 .publishsettings 文件]: ./media/web-sites-dotnet-get-started-vs2012/rzDown2.png
  [下载订阅]: ./media/web-sites-dotnet-get-started-vs2012/rzDownLoadBrowse.png
  [导入]: ./media/web-sites-dotnet-get-started-vs2012/rzImp.png
  [导入发布配置文件]: ./media/web-sites-dotnet-get-started-vs2012/ImportPublishProfile.png
  [验证连接]: ./media/web-sites-dotnet-get-started-vs2012/ValidateConnection.png
  [验证成功的连接]: ./media/web-sites-dotnet-get-started-vs2012/ValidateConnectionSuccess.png
  [“设置”选项卡]: ./media/web-sites-dotnet-get-started-vs2012/PublishWebSettingsTab.png
  [“预览”选项卡中的“开始预览”按钮]: ./media/web-sites-dotnet-get-started-vs2012/PublishWebStartPreview.png
  [“开始预览”文件输出]: ./media/web-sites-dotnet-get-started-vs2012/PublishWebStartPreviewOutput.png
  [报告部署成功的输出窗口]: ./media/web-sites-dotnet-get-started-vs2012/PublishOutput.png
  [服务器资源管理器中的 Azure 网站]: ./media/web-sites-dotnet-get-started-vs2012/ServerExplorerWSSettings.png
  [Visual Studio 中的网站配置]: ./media/web-sites-dotnet-get-started-vs2012/WSConfigurationInVS.png
  [管理门户仪表板选项卡]: ./media/web-sites-dotnet-get-started-vs2012/MPStopStartDelete.png
  [如何管理网站]: /zh-cn/manage/services/web-sites/how-to-manage-websites/
  [管理门户缩放选项卡]: ./media/web-sites-dotnet-get-started-vs2012/MPScale.png
  [如何缩放网站]: /zh-cn/manage/services/web-sites/how-to-scale-websites/
  [Visual Studio 中的日志]: ./media/web-sites-dotnet-get-started-vs2012/LogsInVS.png
  [在 Visual Studio 中排除 Windows Azure 网站的故障]: /zh-cn/develop/net/tutorials/troubleshoot-web-sites-in-visual-studio/
  [使用成员资格、OAuth 和 SQL Database 将安全的 ASP.NET MVC 应用程序部署到 Azure 网站]: /zh-cn/develop/net/tutorials/web-site-with-sql-database/
  [Azure 执行模型]: /zh-cn/develop/net/fundamentals/compute/
  [Azure 网站、 云服务和虚拟机比较]: http://azure.microsoft.com/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/
  [使用存储表、队列和 Blob 的 .NET 多层应用程序]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
