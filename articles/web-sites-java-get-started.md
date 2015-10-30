<properties
	pageTitle="在 Azure 网站中创建 Java Web 应用"
	description="本教程演示了如何将 Java Web 应用部署到 Azure 网站。"
	services="app-service\web"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor="jimbe"/>
<tags
	ms.service="app-service-web"
	ms.date="08/31/2015"
	wacn.date="10/22/2015"/>

# Azure 网站和 Java 入门

> [AZURE.SELECTOR]
- [.Net](/documentation/articles/web-sites-dotnet-get-started)
- [Node.js](/documentation/articles/web-sites-nodejs-develop-deploy-mac)
- [Java](/documentation/articles/web-sites-java-get-started)
- [PHP - Git](/documentation/articles/web-sites-php-mysql-deploy-use-git)
- [PHP - FTP](/documentation/articles/web-sites-php-mysql-deploy-use-ftp)
- [Python](/documentation/articles/web-sites-python-ptvs-django-mysql)

本教程演示如何使用 Java 在 Windows Azure 上（使用 Azure 网站配置 UI）创建网站。

如果你不想使用这些技术，例如，如果你想自定义应用程序容器，请参阅[将自定义 Java 网站上载到 Azure](/documentation/articles/web-sites-java-custom-upload)。

> [AZURE.NOTE]若要完成本教程，您需要一个 Windows Azure 帐户。如果你没有帐户，则可以<a href="/zh-cn/pricing/1rmb-trial/?WT.mc_id=A261C142F" target="_blank">注册获取免费试用版</a>。

## 使用 Azure 配置 UI 创建 Java Web 应用

此信息讲解如何使用 Azure 配置 UI 为你的 Web 应用选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

1. 登录到 Windows Azure 管理门户。
2. 依次单击“新建”、“计算”、“网站”和“快速创建”。
3. 指定 URL 名称。
4. 选择区域。例如“中国东部”。
5. 单击“完成”。你的网站将在几分钟之内创建好。若要查看网站，请在 Azure 管理门户的“网站”视图中，等待状态显示“正在运行”，然后单击该网站的 URL。
6. 还是在 Azure 管理门户的“网站”视图中，单击你网站的名称以打开仪表板。
7. 单击**“配置”**。
8. 在“常规”部分中，通过单击可用版本启用“Java”。
9. Web 容器的选项会显示出来，例如，Tomcat 和 Jetty。选择要使用的 Web 容器。 
10. 单击“保存”。 

你的 Web 应用将在几分钟之内变成基于 Java 的。若要确认该应用是否是基于 Java 的应用，请单击其 URL。请注意，页面上提供的文本会指出这个新 Web 应用是基于 Java 的 Web 应用。

现在你已经使用应用程序容器创建了 Web 应用，请参阅“后续步骤”部分，了解有关将应用程序上载到该 Web 应用的信息。

## 后续步骤

现在，你有了一台在 Azure 上作为 Java 网站运行的 Java 应用程序服务器。若要添加你自己的应用程序或网页，请参阅[将应用程序或网页添加到 Java 网站](/documentation/articles/web-sites-java-add-app)。

<!---HONumber=74-->