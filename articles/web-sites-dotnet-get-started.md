<properties
	pageTitle="在 Azure 网站中创建 ASP.NET网站"
	description="本教程说明如何在 Visual Studio 2013 中创建一个 ASP.NET Web 项目，并将其部署到 Azure 网站中的网站。在不到 15 分钟内，你即可在云中启动并运行一个应用程序。"
	services="app-service\web"
	documentationCenter=".net"
	authors="tdykstra"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="08/10/2015"
	wacn.date="12/14/2015"/>

# 在 Azure 网站中创建 ASP.NET网站

本教程说明如何使用 Visual Studio 2013 或 Visual Studio 2013 for Web Express 创建一个 ASP.NET网站并将其部署到 Azure 网站。本教程假设你以前未使用过 Azure 或 ASP.NET。完成本教程之后，你将能够在云中启动并运行简单的网站。

下图演示了完整的应用程序：

![网站主页](./media/web-sites-dotnet-get-started/deployedandazure.png)

学习内容：

* 如何通过安装 Azure SDK 来让你的计算机可以进行 Azure 开发。
* 如何创建 Visual Studio ASP.NET Web 项目，并将其部署到 Azure 网站。
* 如何对 Web 项目进行更改并重新部署应用程序。
* 如何使用 Azure 管理门户监视和管理网站。

##<a name="video"></a>注册 Windows Azure

完成本教程需要有一个 Azure 帐户：

* 你可以[免费建立一个 Azure 帐户](/zh-cn/pricing/1rmb-trial/?WT.mc_id=A261C142F) - 获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如网站）。

下图演示了完整的应用程序：

![网站主页](./media/web-sites-dotnet-get-started/Create_App.png)

### 教程章节

* [注册 Windows Azure](#video)
* [设置开发环境](#set-up-the-development-environment)
* [在 Visual Studio 中创建 ASP.NET网站](#create-an-aspnet-web-application)
* [将应用程序部署到 Azure](#deploy-the-application-to-azure)
* [更改和重新部署](#make-a-change-and-redeploy)
* [在管理门户中监视和管理站点](#monitor-and-manage-the-site-in-the-management-portal)
* [后续步骤](#next-steps)

<a name="set-up-the-development-environment"></a>
[AZURE.INCLUDE [install-sdk-2015-2013](../includes/install-sdk-2015-2013.md)]

## 创建 ASP.NET网站
<a name="create-an-aspnet-web-application"></a>

第一步是在 Visual Studio 中创建一个 Web 项目，并在 Azure 网站中创建一个网站。完成时，将项目部署到网站，以使其在 Internet 上可用。

下图演示了在创建和部署步骤中要执行的操作。

![创建和部署](./media/web-sites-dotnet-get-started/Create_App.png)

1. 打开 Visual Studio 2015 或 Visual Studio 2013。 

	如果你使用 Visual Studio 2013，屏幕与屏幕截图稍有不同，但过程基本上相同。

2. 在“文件”菜单中，单击“新建\>项目”。

3. 在“新建项目”对话框中，单击“C\#”\>“Web”\>“ASP.NET网站”。如果需要，可以选择“Visual Basic”。

3. 确保选择 **.NET Framework 4.5.2** 作为目标框架。

4. 将应用程序命名为 **MyExample**。

5. 单击**“确定”**。

	![“新建项目”对话框](./media/web-sites-dotnet-get-started/GS13newprojdb.png)

5. 在“新建 ASP.NET 项目”对话框中，选择“MVC”模板。

	[MVC](http://www.asp.net/mvc) 是用于开发网站的 ASP.NET 框架。

5. 在“新建 ASP.NET 项目”对话框中，确保不要勾选“在云中的托管”，然后单击“确定”。

	![“新建 ASP.NET 项目”对话框](./media/web-sites-dotnet-get-started/GS13newaspnetprojdb.png)

	Windows Azure 中国暂时还不支持在 Visual Studio 中直接创建云中的网站，所以用户需要在“[管理门户](https://manage.windowsazure.cn/)”中新建 Azure 网站。

	几秒钟后，Visual Studio 将在指定的文件夹中创建 Web 项目，并在指定的 Azure 区域中创建网站。

	“解决方案资源管理器”窗口将显示新项目中的文件和文件夹。

	![解决方案资源管理器](./media/web-sites-dotnet-get-started/solutionexplorer.png)

## 将应用程序部署到 Azure
<a name="deploy-the-application-to-azure"></a>

1. 在“[管理门户](https://manage.windowsazure.cn/)”中新建 Azure 网站。

2. 点击新建网站的“仪表盘”，在“速览”里下载发布配置文件

7. 在 Visual Studio 中右键点击你的项目，点击“发布”。

	![选择“发布”](./media/web-sites-dotnet-get-started/choosepublish.png)

	几秒钟后，将显示“发布 Web”向导。

4. 在“配置文件”，点击“导入”，并选择刚刚下载的发布配置文件。

8. 在“发布 Web”向导的“连接”选项卡中，单击“验证连接”，以确保 Visual Studio 可以连接到 Azure 来部署 Web 项目。

	![验证连接](./media/web-sites-dotnet-get-started/GS13ValidateConnection.png)

	在连接通过验证后，“验证连接”按钮旁边会出现一个绿色复选标记。

9. 单击**“下一步”**。

	![验证成功的连接](./media/web-sites-dotnet-get-started/GS13ValidateConnection.png)

10. 在“设置”选项卡中，单击“下一步”。

	你可以接受“配置”和“文件发布选项”的默认值。

	你可以通过“配置”下拉列表部署用于远程调试的调试版本。[后续步骤](#next-steps)部分链接到了说明如何在调试模式下远程运行 Visual Studio 的教程。

	![“设置”选项卡](./media/web-sites-dotnet-get-started/GS13SettingsTab.png)

11. 在“预览”选项卡中，单击“发布”。

	如果你想要查看哪些文件将复制到 Azure，可以单击“开始预览”，然后单击“发布”。

	![](./media/web-sites-dotnet-get-started/GS13previewoutput.png)

	Visual Studio 开始执行将文件复制到 Azure 服务器的过程。

	“输出”窗口将显示已执行的部署操作并报告已成功完成部署。

	成功完成部署后，默认浏览器将自动打开指向已部署网站的 URL。你创建的应用程序现在运行于云中。浏览器地址栏中的 URL 显示正在从 Internet 加载该网站。

	![网站在 Azure 中运行](./media/web-sites-dotnet-get-started/GS13deployedsite.png)

13. 关闭浏览器。

## 更改和重新部署
<a name="make-a-change-and-redeploy"></a>

本教程的此部分是可选的。要执行的操作包括更改主页的 **h1** 标题，在开发计算机上本地运行项目以验证更改，然后将更改部署到 Azure。

2. 在“解决方案资源管理器”中打开 *Views/Home/Index.cshtml* 或 *.vbhtml* 文件，将 **h1** 标题从“ASP.NET”更改为“ASP.NET 和 Azure”，然后保存该文件。

	![MVC index.cshtml](./media/web-sites-dotnet-get-started/index.png)

	![MVC h1 更改](./media/web-sites-dotnet-get-started/mvcandazure.png)

1. 按 CTRL+F5，通过在本地计算机上运行网站来查看更新的标题。

	![网站在本地运行](./media/web-sites-dotnet-get-started/localandazure.png)

	`localhost` URL 显示站点正在本地计算机上运行。默认情况下，应用程序在 IIS Express 中运行。IIS Express 是 IIS 的轻型版本，专为在网站部署过程中使用而设计。

1. 关闭浏览器。

1. 在“解决方案资源管理器”中，右键单击该项目并选择“发布”。

	![选择“发布”](./media/web-sites-dotnet-get-started/choosepublish.png)

	此时将显示“发布 Web”向导的“预览”选项卡。如果需要更改任何发布设置，你可以选择其他选项卡，但现在你要做的无非就是重新部署相同的设置。

2. 在“发布 Web”向导中，单击“发布”。

	![单击发布](./media/web-sites-dotnet-get-started/GS13previewoutput.png)

	Visual Studio 将项目部署到 Azure，并在默认浏览器中打开该网站。

	![已更改部署的网站](./media/web-sites-dotnet-get-started/deployedandazure.png)

**提示**：可以启用“Web 单键发布”工具栏以更快地完成部署。单击“视图”\>“工具栏”，然后选择“Web 单键发布”。在工具栏上，你可以选择一个配置文件，单击一个按钮以发布，或单击一个按钮打开“发布 Web”向导。

![Web 单键发布工具栏](./media/web-sites-dotnet-get-started/weboneclickpublish.png)

## 在管理门户中监视和管理网站
<a name="monitor-and-manage-the-site-in-the-management-portal"></a>

[Azure 管理门户](http://manage.windowsazure.cn)是一个 Web 界面，可让你管理和监视 Azure 服务，例如，刚刚创建的网站。在本教程的本部分，你将了解可在该门户中执行的某些操作。

1. 在浏览器中，转到 [http://manage.windowsazure.cn](http://manage.windowsazure.cn)，然后使用你的 Azure 凭据登录。

	门户将显示你的 Azure 服务的列表。

2. 单击你的网站的名称。

	![包含标注新网站的门户主页](./media/web-sites-dotnet-get-started-vs2013/portalhome.png)
  
3. 单击“仪表板”选项卡。

	“仪表板”选项卡显示使用统计信息的概述，以及许多常用站点管理功能的链接。在“速览”下，还可以找到应用程序主页的链接。

	![门户网站仪表板选项卡](./media/web-sites-dotnet-get-started-vs2013/portaldashboard.png)
  
	此时，站点没有过多的通信，因此可能不会在图形中显示任何内容。如果浏览到应用程序，刷新页面几次，然后刷新门户“仪表板”页的话，你将会看到显示了一些统计信息。单击“监视”选项卡可查看更多详细信息。

4. 单击“配置”选项卡。

	“配置”选项卡可让你控制用于站点的 .NET 版本、启用 [WebSockets](/blog/2013/11/14/introduction-to-websockets-on-windows-azure-web-sites/) 和[诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)等功能、设置[连接字符串值](http://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)，等等。[](/documentation/articles/web-sites-configure/)

	![门户网站配置选项卡](./media/web-sites-dotnet-get-started-vs2013/portalconfigure.png)
  
5. 单击“缩放”选项卡。

	对于网站服务的付费层，“缩放”选项卡允许你控制为网站提供服务的计算机的大小和数量，以处理流量的变化。[](/documentation/articles/web-sites-scale/)

	你可以手动缩放或配置条件或计划以进行自动缩放。

	![门户网站缩放选项卡](./media/web-sites-dotnet-get-started-vs2013/portalscale.png)

这只是管理门户的一部分功能。你还可以创建新网站、删除现有站点、停止和重新启动站点，以及管理其他类型的 Azure 服务，如数据库和虚拟机。


## 后续步骤
<a name="next-steps"></a>

在本教程中，你已了解如何创建简单的网站并将其部署到 Azure 网站。你可以通过以下相关主题和资源来了解有关 Azure 网站中网站的详细信息。

* 部署 Web 项目的其他方法

	在本教程中，你已了解通过一项操作创建和部署网站的最快方法。有关使用 Visual Studio 或通过[源代码管理系统](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control)[自动部署](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/continuous-integration-and-continuous-delivery)进行的其他部署方法的概述，请参阅[如何部署 Azure 网站](/documentation/articles/web-sites-deploy)。

	Visual Studio 还可以生成 Windows PowerShell 脚本让你自动完成部署。有关详细信息，请参阅[使一切自动化（使用 Azure 生成真实的云应用程序）](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything)。

* 如何在 Visual Studio 中管理网站

	有关可在“服务器资源管理器”中执行的网站管理功能的信息，请参阅[在 Visual Studio 中对 Azure 网站进行故障排除](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio)。

* 如何对网站进行故障排除

	Visual Studio 提供了相应的功能使你能够在 Azure 日志生成时轻松实时地对其进行查看。你还可以在 Azure 中采用调试模式远程运行。有关详细信息，请参阅[在 Visual Studio 中对 Azure 网站进行故障故障](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio)。

* 如何添加数据库和授权功能

	有关说明如何访问数据库以及将某些站点功能限制给已授权用户的教程，请参阅[将包含成员资格、OAuth 和 SQL 数据库的安全 ASP.NET MVC 应用程序部署到 Azure 网站](/documentation/articles/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)。

* 如何添加自定义域名和 SSL

	有关如何使用 SSL 和你自己的域（例如 www.contoso.com 而不是 contoso.chinacloudsites.cn）的信息，请参阅以下资源：

	* [为 Azure 网站配置自定义域名](/documentation/articles/web-sites-custom-domain-name/)。 
	* [为 Azure 网站启用 HTTPS](/documentation/articles/web-sites-configure-ssl-certificate/)

* 空闲超时后如何避免唤醒等待时间

	默认情况下，网站如果已处于空闲状态相当一段时间，则是未加载的状态。此后，第一次请求必须等待重新加载站点。为了避免等待，你可以启用 AlwaysOn 功能。有关详细信息，请参阅[如何配置网站](/documentation/articles/web-sites-configure/)中的配置选项。

* 如何添加聊天之类的实时功能

	如果要在网站中包含实时功能（如聊天服务、游戏、股票行情等），你可以使用 [ASP.NET SignalR](http://www.asp.net/signalr) 和 [WebSockets](/blog/2013/11/14/introduction-to-websockets-on-windows-azure-web-sites/) 传输方法来获得最佳性能。有关详细信息，请参阅[将 SignalR 与 Azure 网站配合使用](http://www.asp.net/signalr/overview/signalr-20/getting-started-with-signalr-20/using-signalr-with-windows-azure-web-sites)。

* 如何为网站在 Azure 网站、云服务和 VM 之间做出选择

	在 Azure 中，你可以根据本教程中所述在网站中运行网站，也可以在云服务或虚拟机中运行网站。有关详细信息，请参阅 [Azure 执行模型](/documentation/articles/fundamentals-application-models/)和 [Azure 网站、云服务和 VM：何时使用何种产品？](/documentation/articles/choose-web-site-cloud-service-vm/)。
