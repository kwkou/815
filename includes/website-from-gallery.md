库提供了由 Microsoft、第三方公司和开源软件计划开发的各种流行 Web 应用程序。除了用于连接 Azure 管理门户的浏览器，从库中创建的 Web 应用程序不要求安装其他任何软件。 

在本教程中，您将学习：

- 如何通过库创建新网站。

- 如何通过 Azure 门户部署网站。
 
您将构建一个使用默认模板的 WordPress 博客。下图展示了完整的应用程序：


![Wordpress blog][13]

<div class="dev-callout"><strong>注意</strong>
<p>要完成本教程，您需要一个 Azure 帐户。只需几分钟即可创建一个免费试用帐户。有关详细信息，请参阅<a href="http://www.windowsazure.cn/zh-cn/develop/php/tutorials/create-a-windows-azure-account/" target="_blank">创建 Azure 帐户</a>。</p>
</div>
<br />

## 在门户中创建网站

1. 登录到 [Azure 管理门户 ](http://manage.windowsazure.cn)。

2. 单击仪表板左下角的**新建**图标。
	
	![Create New][5]

3. 单击**网站**图标，然后单击**从库中**。
	
	![Create From Gallery][6]

4. 在列表中找到并单击 WordPress 图标，然后单击**下一步**。
	
	![WordPress from list][7]

5. 在**配置应用**页上，为所有字段输入或选择值：
	
  - 输入选择的 URL 名称	
  - 在**数据库**字段中使**新建 MySQL 数据库**保留选定状态
  - 选择离您最近的区域

	![configure your app][8]

6. 然后单击**下一步**。

7. 在**创建新数据库**页上，您可以为新的 MySQL 数据库指定名称或者使用默认名称。选择离您最近的区域作为托管位置。选中屏幕底部的框以同意 ClearDB 针对托管 MySQL 数据库的使用条款。然后，单击复选框以完成网站创建。 
	
	![create database][9]

单击**完成**后，Azure 将启动构建和部署操作。在构建和部署网站的过程中，"网站"页的底部会显示这些操作的状态。执行所有操作后，网站成功部署时，将显示一条最终状态消息。

## 启动和管理您的 WordPress 网站

1. 在**网站**页上单击您的新站点以打开该站点的仪表板。

	![launch dashboard][10]

2. 在**仪表板**管理页上，向下滚动并在**站点 URL** 下单击左侧的链接以打开站点的"欢迎"页。

	![site URL][11] 

3. 输入 WordPress 所需的正确配置信息，然后单击**安装 WordPress** 以完成配置并打开网站的登录页。

	![login to WordPress][12]

4. 通过在**欢迎**页上输入指定的用户名和密码来登录到新的 WordPress 网站。

5. 您将拥有一个新的 WordPress 网站，它看起来类似以下网站。  

	![your WordPress site][13]






[5]: ./media/website-from-gallery/wordpressgallery-01.png
[6]: ./media/website-from-gallery/wordpressgallery-02.png
[7]: ./media/website-from-gallery/wordpressgallery-03.png
[8]: ./media/website-from-gallery/wordpressgallery-04.png
[9]: ./media/website-from-gallery/wordpressgallery-05.png
[10]: ./media/website-from-gallery/wordpressgallery-06.png
[11]: ./media/website-from-gallery/wordpressgallery-07.png
[12]: ./media/website-from-gallery/wordpressgallery-08.png
[13]: ./media/website-from-gallery/wordpressgallery-09.png





