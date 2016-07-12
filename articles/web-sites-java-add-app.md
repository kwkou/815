<properties 
	pageTitle="将 Java 应用程序添加到 Azure Web 应用" 
	description="本教程演示了如何将页面或应用程序添加到已配置为使用 Java 的 Azure Web 应用实例。" 
	services="app-service\web" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web"
	ms.date="05/04/2016" 
	wacn.date="06/29/2016"/>

# 将 Java 应用程序添加到 Azure Web 应用

按照[在 Azure 中创建 Java Web 应用](/documentation/articles/web-sites-java-get-started/)中的说明初始化 Java Web 应用后，即可通过将 WAR 置于 **webapps** 文件夹中来上载应用程序。

**webapps** 文件夹的导航路径因你设置 Web 应用的方式不同而异。

- 如果是通过使用 Azure 配置 UI 来设置 Web 应用，**webapps** 文件夹的路径就会是 **d:\\home\\site\\wwwroot\\webapps** 这样的形式。 

请注意，您可以使用源代码管理来上载您的应用程序或网页，包括在连续集成方案中也这样做。有关源代码管理与你的 Web 应用配合使用的说明，请参阅[从源代码管理发布到 Azure Web 应用](/documentation/articles/web-sites-publish-source-control/)。对于上载应用程序或网页，FTP 也是一种可选择的方式。

Tomcat Web 应用说明：在将 WAR 文件上载到 **webapps** 文件夹后，Tomcat 应用程序服务器将检测到你已经添加了该文件并会将其自动上载。请注意，如果您将文件（除 WAR 文件以外）复制到 ROOT 目录，在使用这些文件之前，将需要重新启动该应用程序服务器。Azure 上运行的 Tomcat Java Web 应用的自动上载功能基于所添加的新 WAR 文件，或添加到 **webapps** 文件夹的新文件或目录。

<!---HONumber=74-->
