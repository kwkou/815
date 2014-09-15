库中提供了众多由 Microsoft、第三方公司和开源软件计划开发的流行 Web 应用程序。从库中创建的 Web 应用程序不要求安装任何软件，只需通过浏览器连接到 Azure 管理门户即可。

在本教程中，你将学习：

-   如何通过库创建新网站。

-   如何通过 Azure 门户部署网站。

你将构建一个使用默认模板的 WordPress 博客。下图演示了完整的应用程序：

![Wordpress 博客][]

**说明**

若要完成本教程，你需要一个 Azure 帐户。只需几分钟即可创建一个免费的试用帐户。有关详细信息，请参阅[创建 Azure 帐户][]。

## 在门户中创建网站

1.  登录到 [Azure 管理门户][]。

2.  单击仪表板左下角的**新建**图标。

    ![新建][]

3.  单击**网站**图标，然后单击**从库中**。

    ![从库创建][]

4.  在列表中找到并单击 WordPress 图标，然后单击**下一步**。

    ![列表中的 WordPress][]

5.  在**配置应用程序**页面上，为所有字段输入或选择值：

-   输入你选择的 URL 名称
-   在**数据库**字段中使**新建 MySQL 数据库**保留选定状态
-   选择离你最近的区域

    ![配置应用][]

1.  然后单击**下一步**。

2.  在**创建新数据库**页面上，你可以为新的 MySQL 数据库指定名称或者使用默认名称。选择离你最近的区域作为托管位置。选中屏幕底部的框以针对你的托管 MySQL 数据库同意 ClearDB 的使用条款。然后，单击指示完成网站创建的复选框。

    ![创建数据库][]

单击**完成**后，Azure 将启动构建和部署操作。在构建和部署网站的过程中，“网站”页的底部会显示这些操作的状态。执行完所有操作后，当网站成功部署时，将显示一条最终状态消息。

## 启动和管理你的 WordPress 网站

1.  在**网站**页上单击你的新网站以打开该网站的仪表板。

    ![启动仪表板][]

2.  在**仪表板**管理页面上，向下滚动并在**网站 URL**下单击左侧的链接以打开网站的欢迎页面。

    ![网站 URL][]

3.  输入 WordPress 所需的正确配置信息，然后单击**安装 WordPress**以完成配置并打开网站的登录页面。

    ![登录到 WordPress][]

4.  通过输入你在**欢迎**页面上指定的用户名和密码登录到新的 WordPress 网站。

5.  你将拥有一个新的 WordPress 网站，它看起来类似下面的网站。

    ![你的 WordPress 网站][Wordpress 博客]

  [Wordpress 博客]: ./media/website-from-gallery/wordpressgallery-09.png
  [创建 Azure 帐户]: http://www.windowsazure.com/en-us/develop/php/tutorials/create-a-windows-azure-account/
  [Azure 管理门户]: http://manage.windowsazure.cn
  [新建]: ./media/website-from-gallery/wordpressgallery-01.png
  [从库创建]: ./media/website-from-gallery/wordpressgallery-02.png
  [列表中的 WordPress]: ./media/website-from-gallery/wordpressgallery-03.png
  [配置应用]: ./media/website-from-gallery/wordpressgallery-04.png
  [创建数据库]: ./media/website-from-gallery/wordpressgallery-05.png
  [启动仪表板]: ./media/website-from-gallery/wordpressgallery-06.png
  [网站 URL]: ./media/website-from-gallery/wordpressgallery-07.png
  [登录到 WordPress]: ./media/website-from-gallery/wordpressgallery-08.png

[5]: ./media/website-from-gallery/wordpressgallery-01.png
[6]: ./media/website-from-gallery/wordpressgallery-02.png
[7]: ./media/website-from-gallery/wordpressgallery-03.png
[8]: ./media/website-from-gallery/wordpressgallery-04.png
[9]: ./media/website-from-gallery/wordpressgallery-05.png
[10]: ./media/website-from-gallery/wordpressgallery-06.png
[11]: ./media/website-from-gallery/wordpressgallery-07.png
[12]: ./media/website-from-gallery/wordpressgallery-08.png
[13]: ./media/website-from-gallery/wordpressgallery-09.png


