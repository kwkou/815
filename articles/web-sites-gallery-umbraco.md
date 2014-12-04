<properties title="从 Microsoft Azure 中的库创建 Umbraco 网站" pageTitle="从 Microsoft Azure 中的库创建 Umbraco 网站" description="必需" metaKeywords="Azure, gallery, Umbraco, web site, website" services="web-sites" solutions="web" documentationCenter="" authors="timamm" manager="paulettm" editor="mollybos" videoId="" scriptId="" />

# 从 Microsoft Azure 中的库创建 Umbraco 网站

Umbraco CMS 是一种功能齐备的开源内容管理系统，可用于创建各种规模的应用程序。Azure 网站应用程序库中提供了众多由 Microsoft、第三方公司以及开源软件计划开发的流行 Web 应用程序。凭借此库，无论是通过应用初学者工具包还是通过集成您自己的设计，只需几分钟就可创建出一个 Umbraco CMS 站点。

本文所展示的新 Azure 预览版门户极大地简化了资源管理。新的 Azure 门户通过将来自 Microsoft 及其合作伙伴的跨平台工具、技术以及服务放置在单一工作区中加速了软件交付流程。您可以将整个应用程序作为单个资源组进行创建、管理以及分析，而不是使用独立的资源，如 Azure 网站、Visual Studio 项目或数据库。

在本教程中，你将学习：

-   如何使用新的 Azure 预览版门户通过库创建新站点
-   如何通过使用 Umbraco CMS 构建博客网站

## 从 Azure 门户中的库创建网站

1.  登录到 [Microsoft Azure 管理门户][Microsoft Azure 管理门户]。

2.  选择“Azure 库”图标。

    ![选择 Web 库][选择 Web 库]

3.  在“库”中，选择“Web”选项卡，然后选择“Umbraco CMS”。

    ![在 Web 库中选择 Umbraco][在 Web 库中选择 Umbraco]

4.  若要创建新的 Umbraco CMS 网站，请单击“创建”。

    ![单击创建][单击创建]

5.  接下来，配置与 Umbraco CMS 关联的所有资源。在本例中，资源是一个网站和一个 SQL Server 数据库。首先，选择“网站”配置网站设置，如“URL”、“Web 托管计划”、“Web 应用程序设置”以及“位置”。

    ![配置资源][配置资源]

6.  现在，配置数据库。选择“数据库”，然后选择“创建新的数据库”。本示例将为 Azure 上的数据库创建 SQL Server。

    ![在 Azure 上创建 SQL Server][在 Azure 上创建 SQL Server]

7.  配置了网站和数据库后，可通过单击第一个“Umbraco CMS”分页底部的“创建”（如上图所示）开始部署应用程序。

    ![单击创建][1]

完成部署后，门户中的开始页面上显示了您针对 Umbraco CMS 的资源组已创建，本例中为 **UmbracoCMSgroup**。在“摘要”部分，单击网站名（本例中为 **umbracocmsgroup**）查看网站的属性。还是在“摘要”部分中，可选择数据库资源以查看关联数据库的属性。

![][0]

## 启动并配置 Umbraco CMS 网站

1.  在网站的详细信息分页中，单击“浏览”浏览您的站点（本例中为 umbracocmsgroup.azurewebsites.net。）

    ![浏览到您的站点][浏览到您的站点]

2.  当您浏览网站时，Umbraco CMS 将启动其安装程序向导。输入所需信息，然后单击“自定义”。

    ![安装 Umbraco 向导][安装 Umbraco 向导]

3.  为 Umbraco 将使用的数据库输入详细的连接和身份验证信息。选择“Microsoft SQL Azure”作为数据库类型。您可以从网站的“站点设置”部分获取数据库的连接字符串。

    ![配置您的数据库][配置您的数据库]

4.  如果您是初次接触 Umbraco CMS，可选择初学者网站工具包。否则，请单击“不，谢谢，我不想安装初学者网站”。

    ![安装初学者网站][安装初学者网站]

5.  Umbraco 安装程序将为您的应用程序完善设置。应用程序配置完成后，您将重定向至 Umbraco CMS 管理仪表板。

    ![Umbraco CMS 仪表板][Umbraco CMS 仪表板]

6.  现在，创建一个要发布的示例文本页面。依次选择“内容”、“溢出”和“TextPage”。

    ![创建文本页面][创建文本页面]

7.  为文本页面输入标题和一些内容，如下所示。完成后，单击“保存并发布”。

    ![输入标题和一些内容][输入标题和一些内容]

8.  页面发布时请稍候。发布完成后，您会在屏幕右下方收到一个小警报。现在，可以在您的网站上浏览新内容了。

    ![已发布的网站页面][已发布的网站页面]

就这么简单！短短几分钟之内，您已使用 Umbraco CMS 成功创建了博客网站。

## 其他资源

[Umbraco 文档][Umbraco 文档]

[Umbraco 视频教程][Umbraco 视频教程]

[Microsoft Azure 预览版门户概述][Microsoft Azure 预览版门户概述]

[Microsoft Azure 预览版门户文档][Microsoft Azure 预览版门户文档]

[Azure 预览版门户（频道 9）][Azure 预览版门户（频道 9）]

[Microsoft Azure 网站文档][Microsoft Azure 网站文档]

<!-- IMAGES -->

  [Microsoft Azure 管理门户]: https://portal.azure.com/
  [选择 Web 库]: ./media/web-sites-gallery-umbraco/01Startboard.PNG
  [在 Web 库中选择 Umbraco]: ./media/web-sites-gallery-umbraco/02WebGallery.PNG
  [单击创建]: ./media/web-sites-gallery-umbraco/03UmbracoCMS.PNG
  [配置资源]: ./media/web-sites-gallery-umbraco/04AppSettings.PNG
  [在 Azure 上创建 SQL Server]: ./media/web-sites-gallery-umbraco/05NewServer.PNG
  [1]: ./media/web-sites-gallery-umbraco/06UmbracoCMSGroup.PNG
  [0]: ./media/web-sites-gallery-umbraco/07UmbracoCMSGroupBlade.PNG
  [浏览到您的站点]: ./media/web-sites-gallery-umbraco/08UmbracoCMSGroupRunning.PNG
  [安装 Umbraco 向导]: ./media/web-sites-gallery-umbraco/09InstallUmbraco7.png
  [配置您的数据库]: ./media/web-sites-gallery-umbraco/10ConfigureYourDatabase.png
  [安装初学者网站]: ./media/web-sites-gallery-umbraco/11InstallAStarterWebsite.png
  [Umbraco CMS 仪表板]: ./media/web-sites-gallery-umbraco/14FriendlyCMS.PNG
  [创建文本页面]: ./media/web-sites-gallery-umbraco/15CreateItemUnderOverflow.PNG
  [输入标题和一些内容]: ./media/web-sites-gallery-umbraco/16EnterAName.PNG
  [已发布的网站页面]: ./media/web-sites-gallery-umbraco/17MyPage.PNG
  [Umbraco 文档]: http://our.umbraco.org/documentation
  [Umbraco 视频教程]: https://umbraco.com/help-and-support/video-tutorials.aspx
  [Microsoft Azure 预览版门户概述]: http://azure.microsoft.com/zh-cn/overview/preview-portal/
  [Microsoft Azure 预览版门户文档]: http://azure.microsoft.com/zh-cn/documentation/preview-portal/
  [Azure 预览版门户（频道 9）]: http://channel9.msdn.com/Blogs/Windows-Azure/Azure-Preview-portal
  [Microsoft Azure 网站文档]: http://azure.microsoft.com/zh-cn/documentation/services/web-sites/
