<properties
	pageTitle="在 Azure App Service 中创建 Java Web 应用 | Azure"
	description="本教程演示了如何将 Java Web 应用部署到 Azure App Service。"
	services="app-service\web"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="Java"
	ms.topic="get-started-article"
	ms.date="08/11/2016"
	wacn.date="09/26/2016"
	ms.author="robmcm"/>

# 在 Azure App Service 中创建 Java Web 应用

[AZURE.INCLUDE [选项卡](../../includes/app-service-web-get-started-nav-tabs.md)]

本教程演示如何通过 [Azure 门户预览][在 Azure App Service 中创建 Java Web 应用]。Azure 门户预览是可用于管理 Azure 资源的 Web 界面。

> [AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以[注册试用帐户]。
>

## Java 应用程序选项

在 App Service Web 应用中设置 Java 应用程序可以有多种方法。

1. 创建应用，然后配置**应用程序设置**。

	应用服务提供多个使用默认配置的 Tomcat 和 Jetty 版本。如果要托管的应用程序可以使用某个内置版本，则这种设置 Web 容器的方法是最简单的，如果你只想将 war 文件上传至 Web 容器，这种方法则是最适合的。使用这种方法时，先在 Azure 门户预览中创建一个应用，然后转到应用的“应用程序设置”边栏选项卡，选择 Java 版本以及所需的 Java Web 容器。使用此方法时，Java 和 Web 容器均从“程序文件”运行。其他方法会将 Web 容器和 JVM（可能）放在磁盘空间中。使用此模型时，用户不具有编辑文件系统中此部分文件的权限。这意味着无法执行某些操作，如配置 *server.xml* 文件或将库文件放入 */lib* 文件夹。有关详细信息，请参阅本教程后面的[创建和配置 Java Web 应用](#appsettings)部分。

3. 创建一个应用，然后手动复制和编辑配置文件

	你可能需要托管一个自定义 Java 应用程序，该程序不部署在 App Service 所提供的任何 Web 容器中。例如：
	
	* 你的 Java 应用程序需要的 Tomcat 或 Jetty 版本不受 App Service 的直接支持，或者库中不提供该版本。
	* 你的 Java 应用程序接受 HTTP 请求，不以 WAR 的形式部署到预先存在的 Web 容器中。
	* 你想要从头开始自行配置 Web 容器。
	* 你想要使用的 Java 版本在应用服务中不受支持，因此想要自行上载它。

	对于这样的情况，你可以通过 Azure 门户预览创建一个应用，然后手动提供相应的运行时文件。在本示例中，这些文件将占用你的应用服务计划的存储空间配额。有关详细信息，请参阅[将自定义 Java Web 应用上载到 Azure]。

## <a name="portal" id="appsettings"></a> 创建和配置 Java Web 应用

本部分演示如何使用门户的“应用程序设置”边栏选项卡创建 Web 应用并将其配置为适用于 Java。

1. 登录到 [Azure 门户预览]。

2. 单击“新建”>“Web + 移动”>“Web 应用”。

	![新的 Web 应用][newwebapp]

4. 在“Web 应用”框中输入 Web 应用的名称。

	该名称在 chinacloudsites.cn 域中必须是唯一的，因为 Web 应用的 URL 将是 {name}.chinacloudsites.cn。如果你输入的名称不是唯一的，则会在文本框中显示一个红色的感叹号。

5. 选择“资源组”或新建一个。

	有关资源组的详细信息，请参阅[使用 Azure 门户预览管理 Azure 资源]。

6. 选择“App Service 计划/位置”或新建一个。

	有关 App Service 计划的详细信息，请参阅 [Azure App Service 计划概述]。

7. 单击“创建”。

	![创建 Web 应用][newwebapp2]
 
8. 创建Web 应用后，单击“Web 应用”>“{你的 web 应用}”。
 
	![选择 Web 应用][selectwebapp]

9. 在“Web 应用”边栏选项卡中，单击“设置”。

10. 单击“应用程序设置”。

11. 选择所需的 **Java 版本**。

12. 选择所需 **Java 次要版本**。如果选择“最新”，应用将使用应用服务中提供的该 Java 主要版本的最新次要版本。**最新**项对于从“应用程序设置”创建的 Java 应用是唯一的。若从库中创建 Java 应用，则必须管理自己的容器和 JVM 更改。

12. 选择所需的 **Web 容器**。若选择的容器名称以**最新**开头，应用将保留为应用服务中提供的该 Web 容器主要版本的最新版本。

	![Web 容器版本][versions]

13. 单击“保存”。

	你的 Web 应用将在几分钟之内变成基于 Java 的 Web 应用，并配置为使用所选的 Web 容器。

14. 单击“Web 应用”>“{你的新 Web 应用}”。

15. 单击 **URL** 浏览到新站点。

	该网页确认你已创建基于 Java 的 Web 应用。

## 后续步骤

现在，你有了一台在 Azure App Service 的 Web 应用中运行的 Java 应用程序服务器。若要将你自己的代码部署到 Web 应用，请参阅[将应用程序或网页添加到 Java Web 应用]。

有关如何在 Azure 中开发 Java 应用程序的详细信息，请参阅 [Java 开发中心]。

<!-- URL List -->

[将应用程序或网页添加到 Java Web 应用]: /documentation/articles/web-sites-java-add-app/
[Azure App Service 计划概述]: /documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/
[Azure 门户预览]: https://portal.azure.cn/
[激活你的 Visual Studio 订户权益]: /pricing/1rmb-trial/
[注册试用帐户]: /pricing/1rmb-trial/
[Try App Service]: https://tryappservice.azure.com/
[在 Azure App Service 中创建 Java Web 应用]: /documentation/articles/app-service-changes-existing-services/
[Java 开发中心]: /develop/java/
[使用 Azure 门户预览管理 Azure 资源]: /documentation/articles/resource-group-portal/
[将自定义 Java Web 应用上载到 Azure]: /documentation/articles/web-sites-java-custom-upload/

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

<!---HONumber=Mooncake_0919_2016-->