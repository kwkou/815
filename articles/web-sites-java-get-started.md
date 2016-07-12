<properties
	pageTitle="在 Azure 中创建 Java Web 应用 | Azure"
	description="本教程演示了如何将 Java Web 应用部署到 Azure 中。"
	services="app-service\web"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="06/01/2016"
	wacn.date="07/04/2016"/>

# 在 Azure 中创建 Java Web 应用

[AZURE.INCLUDE [选项卡](../includes/app-service-web-get-started-nav-tabs.md)]

本教程演示如何通过 [Azure 经典管理门户](https://manage.windowsazure.cn/)[在 Azure 中创建 Java Web 应用](/documentation/services/web-sites/)。Azure 经典管理门户是可用于管理 Azure 资源的 Web 界面。

> [AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以[注册试用版]。
>

## Java 应用程序选项

在 Azure Web 应用中设置 Java 应用程序可以有多种方法。

1. 创建一个应用，然后使用 Azure 配置 UI。

	Azure 网站提供多个使用默认配置的 Tomcat 和 Jetty 版本。如果你要托管的应用程序可以使用某个内置版本，则这种设置 Web 容器的方法是最简单的，但缺少其他方法中的配置功能。使用这种方法时，你先在 Azure 经典管理门户中创建一个应用，然后转到该应用的“配置”边栏选项卡，以便选择你的 Java 版本以及所需的 Java Web 容器。使用此方法时，应用将从辅助角色用于托管该应用的本地硬盘运行，不会占用租户的磁盘空间。使用此模型时，你没有相关访问权限，因此无法编辑文件系统中此部分的文件，这意味着你不能执行配置 *server.xml* 文件或将库文件放在 */lib* 文件夹中这样的操作。
  
3. 创建一个应用，然后手动复制和编辑配置文件

	你可能想要托管一个自定义 Java 应用程序，该程序不部署在 Azure Web 应用所提供的任何 Web 容器中。例如：
	
	* 你的 Java 应用程序需要 Azure 不直接支持，或者库中不提供的 Tomcat 或 Jetty 版本。
	* 你的 Java 应用程序接受 HTTP 请求，不以 WAR 的形式部署到预先存在的 Web 容器中。
	* 你想要从头开始自行配置 Web 容器。 
	* 你想要使用在 Azure 中不受支持的 Java 版本，且想要自行上载它。

	对于这样的情况，你可以通过 Azure 经典管理门户创建一个应用，然后手动提供相应的运行时文件。在本示例中，这些文件将占用你 App Service 计划的存储空间配额。有关详细信息，请参阅[将自定义 Java Web 应用上载到 Azure](/documentation/articles/web-sites-java-custom-upload/)。

## <a name="portal"></a>创建和配置 Java Web 应用

此信息讲解如何使用 Azure 配置 UI 为你的 Web 应用选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

1. 登录到 Microsoft [Azure 经典管理门户](https://manage.windowsazure.cn/)。
2. 依次单击“新建”、“计算”、“网站”和“快速创建”。
3. 指定 URL 名称。
4. 选择区域。例如“中国东部”。
5. 单击“完成”。你的网站将在几分钟之内创建好。若要查看网站，请在 Azure 经典管理门户的“网站”视图中，等待状态显示“正在运行”，然后单击该网站的 URL。
6. 还是在 Azure 经典管理门户的“网站”视图中，单击你网站的名称以打开仪表板。
7. 单击**“配置”**。
8. 在“常规”部分中，通过单击可用版本启用“Java”。
9. Web 容器的选项会显示出来，例如，Tomcat 和 Jetty。选择要使用的 Web 容器。 
10. 单击“保存”。 

你的 Web 应用将在几分钟之内变成基于 Java 的。若要确认该应用是否是基于 Java 的应用，请单击其 URL。请注意，页面上提供的文本会指出这个新 Web 应用是基于 Java 的 Web 应用。

现在你已经使用应用容器创建了 Web 应用，请参阅[后续步骤](#next-steps)部分，了解有关如何将应用程序上传到该 Web 应用的信息。

##<a name="next-steps"></a>后续步骤

现在，你有了一台在 Azure 的 Web 应用中运行的 Java 应用程序服务器。若要将你自己的代码部署到 Web 应用，请参阅[将应用程序或网页添加到 Java Web 应用](/documentation/articles/web-sites-java-add-app/)。

有关如何在 Azure 中开发 Java 应用程序的详细信息，请参阅 [Java 开发中心](/develop/java/)。

<!-- URL List -->

[Add an application or webpage to your Java web app]: /documentation/articles/web-sites-java-add-app/
[Azure App Service plans overview]: /documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/
[Azure Portal]: https://portal.azure.cn/
[activate your Visual Studio subscriber benefits]: /pricing/1rmb-trial/
[注册试用版]: /pricing/1rmb-trial/
[Try Azure Web App]: https://tryappservice.azure.com/
[web app in Azure]: /documentation/services/web-sites/
[Java Developer Center]: /develop/java/
[Using the Azure Portal to manage your Azure resources]: /documentation/articles/resource-group-portal/
[Upload a custom Java web app to Azure]: /documentation/articles/web-sites-java-custom-upload/

<!-- IMG List -->

[newwebapp]: ./media/web-sites-java-get-started/newwebapp.png
[newwebapp2]: ./media/web-sites-java-get-started/newwebapp2.png
[selectwebapp]: ./media/web-sites-java-get-started/selectwebapp.png
[versions]: ./media/web-sites-java-get-started/versions.png
[newmarketplace]: ./media/web-sites-java-get-started/newmarketplace.png
[webmobilejetty]: ./media/web-sites-java-get-started/webmobilejetty.png
[jettyblade]: ./media/web-sites-java-get-started/jettyblade.png
[jettyportalcreate2]: ./media/web-sites-java-get-started/jettyportalcreate2.png
[jettyurl]: ./media/web-sites-java-get-started/jettyurl.png
[tomcat]: ./media/web-sites-java-get-started/tomcat.png
[jetty]: ./media/web-sites-java-get-started/jetty.png

<!---HONumber=Mooncake_0627_2016-->