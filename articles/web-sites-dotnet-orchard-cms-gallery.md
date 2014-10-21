<properties linkid="develop-dotnet-website-from-gallery" urlDisplayName="Web Site from Gallery" pageTitle="Create an Orchard CMS web site from the gallery in Azure" metaKeywords="Azure build website, manage website Azure" description="A tutorial that teaches you how to create a new web site in Azure. Also learn how to launch and manage your site using the Management Portal." metaCanonical="" services="web-sites" documentationCenter=".NET" title="Create an Orchard CMS web site from the gallery in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中从库中创建 Orchard CMS 网站

库中提供了众多由 Microsoft、第三方公司和开源软件计划开发的流行 Web 应用程序。从库中创建 Web 应用程序不要求安装任何软件，只需通过浏览器连接到 [Azure 管理门户][Azure 管理门户]即可。有关库中 Web 应用程序的详细信息，请参阅 [Windows Web 应用程序库][Windows Web 应用程序库]。

在本教程中，你将学习：

-   如何从库中创建新网站

-   如何从管理门户启动和管理网站

你将构建一个使用默认模板的 Orchard CMS 网站。[Orchard][Orchard] 是一个基于 .NET 的免费、开源 CMS 应用程序，它允许你创建自定义的内容驱动型网站。Orchard CMS 包括一个扩展性框架，通过该框架你可以[下载附加模块和主题][下载附加模块和主题]来自定义你的网站。下图显示你将要创建的 Orchard CMS 网站。

![Orchard 博客][Orchard 博客]

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

## 从库中创建 Orchard 网站

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  单击该门户左下角的“新建”图标。

    ![新建][新建]

3.  单击“网站”图标，然后单击“从库中”。

    ![从库创建][从库创建]

4.  在列表中找到并单击“Orchard CMS”图标，然后单击箭头以继续。

    ![列表中的 Orchard][列表中的 Orchard]

5.  在**“配置应用程序”**页面上，为所有字段输入或选择值：

-   输入选择的 URL 名称。
-   选择最靠近你的用户的区域。（这将确保获得最佳性能。）

    ![配置应用][配置应用]

1.  单击框右下角的复选标记可启动新 Orchard CMS 网站的部署。

Azure 将发起构建和部署操作。在构建和部署网站的同时，网站管理门户的底部会显示这些操作的状态。执行完所有操作后，将显示一条消息，指示已创建网站。

## 启动和管理 Orchard 网站

1.  单击“网站”页上的新网站的名称，然后单击门户底部的“浏览”以打开网站的欢迎页面。

    ![启动仪表板][启动仪表板]

    ![“浏览”按钮][“浏览”按钮]

2.  输入 Orchard 所需的配置信息，然后单击“完成设置”完成配置并打开网站主页。

    ![登录到 Orchard][登录到 Orchard]

    你将拥有一个新的 Orchard 网站，它看起来类似如下屏幕快照。

    ![你的 Orchard 网站][Orchard 博客]

3.  按照 [Orchard 文档][Orchard 文档]中的详细介绍了解更多关于 Orchard 的信息并配置你的新网站。

## <span class="short-header">后续步骤</span>后续步骤

-   [使用 Microsoft WebMatrix 开发和部署网站][使用 Microsoft WebMatrix 开发和部署网站] -- 了解如何在 WebMatrix 中编辑 Azure 网站。
-   [使用成员资格、OAuth 和 SQL Database 将安全 ASP.NET MVC 应用程序部署到 Azure 网站][使用成员资格、OAuth 和 SQL Database 将安全 ASP.NET MVC 应用程序部署到 Azure 网站]-- 了解如何从 Visual Studio 创建新网站。

  [Azure 管理门户]: http://manage.windowsazure.cn
  [Windows Web 应用程序库]: http://www.microsoft.com/web/gallery/categories.aspx
  [Orchard]: http://www.orchardproject.net/
  [下载附加模块和主题]: http://gallery.orchardproject.net/
  [Orchard 博客]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-08.png
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [新建]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-01.png
  [从库创建]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-02.png
  [列表中的 Orchard]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-03.png
  [配置应用]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-04.png
  [启动仪表板]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-05.png
  [“浏览”按钮]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-12.png
  [登录到 Orchard]: ./media/web-sites-dotnet-orchard-cms-gallery/orchardgallery-07.png
  [Orchard 文档]: http://docs.orchardproject.net/
  [使用 Microsoft WebMatrix 开发和部署网站]: /en-us/develop/net/tutorials/website-with-webmatrix/
  [使用成员资格、OAuth 和 SQL Database 将安全 ASP.NET MVC 应用程序部署到 Azure 网站]: /en-us/develop/net/tutorials/web-site-with-sql-database/
