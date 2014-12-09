<properties linkid="web-sites-hybrid-connection" title="混合连接：将 Azure 网站连接到本地资源" pageTitle="混合连接：将 Azure 网站连接到本地资源" description="在 Azure 网站和使用静态 TCP 端口的本地资源之间创建连接" metaKeywords="" services="web-sites" solutions="web" documentationCenter="" authors="timamm" manager="paulettm" editor="mollybos" videoId="" scriptId="" />

# 使用混合连接将 Azure 网站连接到本地资源

您可将 Microsoft Azure 上的网站连接到使用静态 TCP 端口的任何本地资源，例如 SQL Server、MySQL、HTTP Web API、移动服务和大多数自定义 Web 服务。本文将向您介绍如何在 Azure 网站和本地 SQL Server 数据库之间创建混合连接。

> [WACOM.NOTE] 混合连接功能的网站部分仅在 [Azure 门户][Azure 门户]中有提供。若要在 BizTalk 服务中创建连接，请参阅[混合连接][混合连接]。

## 先决条件

-   Azure 订阅。若要获取免费订阅，请参见 [Azure 免费使用][Azure 免费使用]。

-   若要通过混合连接使用本地 SQL Server 或 SQL Server Express 数据库，需要在静态端口上启用 TCP/IP。建议在 SQL Server 上使用默认实例，因为它使用静态端口 1433。有关安装和配置 SQL Server Express 以使用混合连接的信息，请参见[使用混合连接从 Azure 网站连接到本地 SQL Server][使用混合连接从 Azure 网站连接到本地 SQL Server]。

-   其上安装本地混合连接管理器代理的计算机将在本教程稍后部分介绍。

    -   必须能够通过端口 5671 连接至 Azure
    -   必须能够访问您的本地资源的 *hostname*:*portnumber*。

> [WACOM.NOTE]本文中的分步指南假定您从将托管本地混合连接代理的计算机使用浏览器。

## 本文内容

[在 Azure 预览版门户中创建网站][在 Azure 预览版门户中创建网站]

[创建混合连接和 BizTalk 服务][创建混合连接和 BizTalk 服务]

[安装本地混合连接管理器以完成连接][安装本地混合连接管理器以完成连接]

[后续步骤][后续步骤]

## 在 Azure 门户中创建网站

> [WACOM.NOTE] 如果已在 Azure 门户中创建了希望用于此教程的网站，可直接跳至[创建混合连接和 BizTalk 服务][创建混合连接和 BizTalk 服务]。

1.  在 [Azure 门户][Azure 门户]的左下角处，单击“新建”，然后选择“网站”。

    ![新建按钮][新建按钮]

    ![新建网站][新建网站]

2.  在“网站”分页上为您的网站提供名称，然后单击“创建”。

    ![网站名称][网站名称]

3.  几分钟之后，网站创建成功并显示其网站分页。该分页是一个可垂直滚动的仪表板，便于您管理站点。

    ![运行中的网站][运行中的网站]

4.  若要验证该站点是否已激活，可单击“浏览”图标显示默认页面。

    ![单击浏览查看您的网站][单击浏览查看您的网站]

    ![默认的网站页面][默认的网站页面]

接下来，您将为该网站创建混合连接和 BizTalk 服务。

<a name="CreateHC"></a>

## 创建混合连接和 BizTalk 服务

1.  返回门户，向下滚动网站的分页并选择“混合连接”。

    ![混合连接][1]

2.  在“混合连接”分页上，单击“添加”。

    ![添加混合连接][添加混合连接]

3.  “添加混合连接”分页打开。因为这是您的第一个混合连接，所以预先选择了“新建混合连接”选项，“创建混合连接”分页将打开。

    ![创建混合连接][创建混合连接]

    在“创建混合连接”分页上：

    -   在“名称”处为连接提供一个名称。
    -   在“主机名”处输入托管您的资源的本地计算机的名称。
    -   在“端口”处输入本地资源使用的端口号（SQL Server 默认实例使用 1433）。
    -   单击“Biz Talk 服务”

4.  “Biz Talk 服务”分页打开。为 BizTalk 服务输入一个名称，然后单击“确定”。

    ![创建 BizTalk 服务][创建 BizTalk 服务]

    “创建 Biz Talk 服务”分页关闭，您将返回“创建混合连接”分页。

5.  在“创建混合连接”分页上，单击“确定”。

    ![单击确定][单击确定]

6.  完成这一流程后，门户中的“通知”区域将告知您连接已创建成功。

    ![成功通知][成功通知]

7.  在网站分页上，“混合连接”图标现在显示已创建了一个混合连接。

    ![已创建一个混合连接][已创建一个混合连接]

此时，您已完成了云混合连接基础架构中最重要的一部分。接下来，您将创建相应的本地部分。

<a name="InstallHCM"></a>

## 安装本地混合连接管理器以完成连接

1.  在网站分页上，单击“混合连接”图标

    ![混合连接图标][混合连接图标]

2.  在“混合连接”分页上，最近添加的终结点的“状态”栏显示的是“未连接”。单击该连接对其进行配置。

    ![未连接][未连接]

    “混合连接”分页打开。

    ![未连接分页][未连接分页]

3.  在该分页上，单击“侦听器安装”。

    ![单击侦听器安装][单击侦听器安装]

4.  “混合连接属性”分页打开。在“本地混合连接管理器”下，选择“单击此处进行安装”。

    ![单击此处进行安装][单击此处进行安装]

5.  在“应用程序运行安全性警告”对话框中，选择“运行”以继续。

    ![选择运行以继续][选择运行以继续]

6.  在“用户帐户控制”对话框中，选择“是”。

    ![选择是][选择是]

7.  “混合连接管理器”开始下载并安装。

    ![安装][安装]

8.  安装完成后，单击“关闭”。

    ![单击关闭][单击关闭]

    在“混合连接”分页上，“状态”栏现在显示“已连接”。

    ![已连接状态][已连接状态]

现在，混合连接基础架构已完成，接下来创建一个混合应用程序开始使用此架构。

<a name="NextSteps"></a>

## 后续步骤

-   有关创建使用混合连接的 ASP.NET web 应用程序的信息，请参见[使用混合连接从 Azure 网站连接到本地 SQL Server][使用混合连接从 Azure 网站连接到本地 SQL Server]。

-   有关结合使用混合连接与移动服务的信息，请参见[使用混合连接从 Azure 移动服务连接到本地 SQL Server][使用混合连接从 Azure 移动服务连接到本地 SQL Server]。

### 其他资源

[混合连接概述][混合连接概述]

[混合连接网站][混合连接网站]

[BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡][BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡]

<!-- IMAGES -->

  [Azure 门户]: https://manage.windowsazure.cn
  [混合连接]: http://go.microsoft.com/fwlink/p/?LinkID=397274
  [Azure 免费使用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [使用混合连接从 Azure 网站连接到本地 SQL Server]: http://go.microsoft.com/fwlink/?LinkID=397979
  [在 Azure 预览版门户中创建网站]: #CreateSite
  [创建混合连接和 BizTalk 服务]: #CreateHC
  [安装本地混合连接管理器以完成连接]: #InstallHCM
  [后续步骤]: #NextSteps
  [新建按钮]: ./media/web-sites-hybrid-connection-get-started/B01New.png
  [新建网站]: ./media/web-sites-hybrid-connection-get-started/B02NewWebsite.png
  [网站名称]: ./media/web-sites-hybrid-connection-get-started/B03WebsiteCreationBlade.png
  [运行中的网站]: ./media/web-sites-hybrid-connection-get-started/B04WebSiteRunningBlade.png
  [单击浏览查看您的网站]: ./media/web-sites-hybrid-connection-get-started/B05Browse.png
  [默认的网站页面]: ./media/web-sites-hybrid-connection-get-started/B06DefaultWebSitePage.png
  [1]: ./media/web-sites-hybrid-connection-get-started/C01CreateHCHCIcon.png
  [添加混合连接]: ./media/web-sites-hybrid-connection-get-started/C02CreateHCAddHC.png
  [创建混合连接]: ./media/web-sites-hybrid-connection-get-started/C03TwinCreateHCBlades.png
  [创建 BizTalk 服务]: ./media/web-sites-hybrid-connection-get-started/C04CreateHCCreateBTS.png
  [单击确定]: ./media/web-sites-hybrid-connection-get-started/C05CreateBTScomplete.png
  [成功通知]: ./media/web-sites-hybrid-connection-get-started/C06CreateHCSuccessNotification.png
  [已创建一个混合连接]: ./media/web-sites-hybrid-connection-get-started/C07CreateHCOneConnectionCreated.png
  [混合连接图标]: ./media/web-sites-hybrid-connection-get-started/D01HCIcon.png
  [未连接]: ./media/web-sites-hybrid-connection-get-started/D02NotConnected.png
  [未连接分页]: ./media/web-sites-hybrid-connection-get-started/D03NotConnectedBlade.png
  [单击侦听器安装]: ./media/web-sites-hybrid-connection-get-started/D04ClickListenerSetup.png
  [单击此处进行安装]: ./media/web-sites-hybrid-connection-get-started/D05ClickToInstallHCM.png
  [选择运行以继续]: ./media/web-sites-hybrid-connection-get-started/D06ApplicationRunWarning.png
  [选择是]: ./media/web-sites-hybrid-connection-get-started/D07UAC.png
  [安装]: ./media/web-sites-hybrid-connection-get-started/D08HCMInstalling.png
  [单击关闭]: ./media/web-sites-hybrid-connection-get-started/D09HCMInstallComplete.png
  [已连接状态]: ./media/web-sites-hybrid-connection-get-started/D10HCStatusConnected.png
  [使用混合连接从 Azure 移动服务连接到本地 SQL Server]: http://azure.microsoft.com/zh-cn/documentation/articles/mobile-services-dotnet-backend-hybrid-connections-get-started/
  [混合连接概述]: http://azure.microsoft.com/zh-cn/documentation/articles/integration-hybrid-connection-overview/
  [混合连接网站]: http://azure.microsoft.com/zh-cn/services/biztalk-services/
  [BizTalk 服务：“仪表板”、“监视”、“缩放”、“配置”和“混合连接”选项卡]: http://azure.microsoft.com/zh-cn/documentation/articles/biztalk-dashboard-monitor-scale-tabs/
