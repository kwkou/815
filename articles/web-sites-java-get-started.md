<properties
	pageTitle="在 Azure 中创建 Java Web 应用 | Azure"
	description="本教程演示了如何将 Java Web 应用部署到 Azure Web 应用。"
	services="app-service\web"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor="jimbe"/>
<tags
	ms.service="web-sites"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

# 在 Azure 中创建 Java Web 应用

> [AZURE.SELECTOR]
- [.Net](/documentation/articles/web-sites-dotnet-get-started)
- [Node.js](/documentation/articles/web-sites-nodejs-develop-deploy-mac)
- [Java](/documentation/articles/web-sites-java-get-started)
- [PHP - Git](/documentation/articles/web-sites-php-mysql-deploy-use-git)
- [PHP - FTP](/documentation/articles/web-sites-php-mysql-deploy-use-ftp)
- [Python](/documentation/articles/web-sites-python-ptvs-django-mysql)

本教程演示如何通过 [Azure 管理门户](https://manage.windowsazure.cn)在 [Azure Web 应用](/documentation/services/web-sites/)中创建 Java Web 应用。

> [AZURE.NOTE]若要完成本教程，您需要一个 Azure 帐户。如果你没有帐户，可以[注册试用版][]。
> 

## Java 应用程序选项

在 Azure 中设置 Java 应用程序可以有多种方法。

1. 创建一个应用，然后使用 Azure 配置 UI。

	Azure Web 应用提供多个使用默认配置的 Tomcat 和 Jetty 版本。如果你要托管的应用程序可以使用某个内置版本，则这种设置 Web 容器的方法是最简单的，但缺少其他方法中的配置功能。使用这种方法时，你先在门户中创建一个应用，然后转到该应用的“配置”边栏选项卡，以便选择你的 Java 版本以及所需的 Java Web 容器。使用此方法时，应用将从辅助角色用于托管该应用的本地硬盘运行，不会占用租户的磁盘空间。使用此模型时，你没有相关访问权限，因此无法编辑文件系统中此部分的文件，这意味着你不能执行配置 *server.xml* 文件或将库文件放在 */lib* 文件夹中这样的操作。有关详细信息，请参阅本教程后面的[创建和配置 Java Web 应用](#appsettings)部分。
  
3. 创建一个应用，然后手动复制和编辑配置文件

	你可能需要托管一个自定义 Java 应用程序，该程序不部署在 Azure 所提供的任何 Web 容器中。例如，你可能出于以下原因而需要这样做：
	
	* 你的 Java 应用程序需要的 Tomcat 或 Jetty 版本不受 Azure 的直接支持，或者库中不提供该版本。
	* 你的 Java 应用程序接受 HTTP 请求，不以 WAR 的形式部署到预先存在的 Web 容器中。
	* 你想要从头开始自行配置 Web 容器。 
	* 你想要使用的 Java 版本在 Azure 中不受支持，因此想要自行上载它。

	对于这样的情况，你可以通过门户创建一个应用，然后手动提供相应的运行时文件。在本示例中，这些文件将占用你 App Service 计划的存储空间配额。有关详细信息，请参阅[将自定义 Java Web 应用上载到 Azure](/documentation/articles/web-sites-java-custom-upload/)。

##<a name="appsettings"></a> 使用 Azure 配置 UI 创建 Java Web 应用

此信息讲解如何使用 Azure 配置 UI 为你的 Web 应用选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

1. 登录到 Azure [管理门户](https://manage.windowsazure.cn/)。
2. 依次单击“新建”、“计算”、“ Web 应用”和“快速创建”。
3. 指定 URL 名称。
4. 选择区域。例如“中国东部”。
5. 单击“完成”。你的 Web 应用将在几分钟之内创建好。若要查看 Web 应用，请在 Azure 管理门户的“ Web 应用”视图中，等待状态显示“正在运行”，然后单击该 Web 应用的 URL。
6. 还是在 Azure 管理门户的“ Web 应用”视图中，单击你 Web 应用的名称以打开仪表板。
7. 单击**“配置”**。
8. 在“常规”部分中，通过单击可用版本启用“Java”。
9. Web 容器的选项会显示出来，例如，Tomcat 和 Jetty。选择要使用的 Web 容器。 
10. 单击“保存”。 

你的 Web 应用将在几分钟之内变成基于 Java 的。若要确认该应用是否是基于 Java 的应用，请单击其 URL。请注意，页面上提供的文本会指出这个新 Web 应用是基于 Java 的 Web 应用。

现在你已经使用应用程序容器创建了 Web 应用，请参阅“后续步骤”部分，了解有关将应用程序上载到该 Web 应用的信息。

## 后续步骤

现在，你有了一台在 Azure 中运行的 Java 应用程序服务器。若要将你自己的代码部署到 Web 应用，请参阅[将应用程序或网页添加到 Java Web 应用](/documentation/articles/web-sites-java-add-app)。

有关如何在 Azure 中开发 Java 应用程序的详细信息，请参阅 [Java 开发中心](/develop/java/)。

<!-- External Links -->
[activate your MSDN subscriber benefits]: /pricing/1rmb-trial/
[注册试用版]: /pricing/1rmb-trial/

[Try Azure Websites]: https://tryappservice.azure.com/

<!---HONumber=Mooncake_1207_2015-->