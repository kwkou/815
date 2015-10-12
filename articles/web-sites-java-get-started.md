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
	ms.date="06/03/2015"
	wacn.date="10/03/2015"/>

# Azure 网站和 Java 入门

本教程演示如何使用 Java 在 Windows Azure 上（使用 Azure 应用程序库或 Azure 网站配置用户界面）创建网站。

如果你不想使用这些技术，例如，如果你想自定义应用程序容器，请参阅[将自定义 Java 网站上载到 Azure](/documentation/articles/web-sites-java-custom-upload)。

> [AZURE.NOTE]若要完成本教程，你需要一个 Windows Azure 帐户。如果你没有帐户，则可以<a href="/zh-cn/pricing/1rmb-trial/" target="_blank">注册获取免费试用版</a>。

# 使用 Azure 应用程序库创建 Java 网站

此信息讲解如何使用 Azure 应用程序库为你的网站选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

下面演示从应用程序库中使用 Tomcat 生成的网站的外观会是怎样的：

![使用 Apache Tomcat 的网站](./media/web-sites-java-get-started/tomcat.png)

下面演示从应用程序库中使用 Jetty 生成的网站的外观会是怎样的：

![使用 Jetty 的网站](./media/web-sites-java-get-started/jetty.png)

1. 登录到 Windows Azure 管理门户。
2. 依次单击“新建”、“计算”、“网站”和“从库中”。
3. 从应用程序列表中，选择 Java 应用程序服务器之一，如 **Apache Tomcat** 或 **Jetty**。
4. 单击**“下一步”**。
5. 指定 URL 名称。
6. 选择区域。例如“中国东部”。
7. 单击“完成”。

你的网站将在几分钟之内创建好。若要查看网站，请在 Azure 管理门户的“网站”视图中，等待状态显示“正在运行”，然后单击该网站的 URL。

现在你已经使用应用程序容器创建了网站，请参阅**后续步骤**部分，了解有关将应用程序上载到该网站的信息。

# 使用 Azure 配置用户界面创建 Java 网站

此信息讲解如何使用 Azure 配置用户界面为你的网站选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

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

你的网站将在几分钟之内变成基于 Java 的网站。要确认该网站基于 Java，请在 Azure 管理门户内的“网站”视图中，等待状态显示为“正在运行”，然后单击该网站的 URL。请注意，页面上提供的文本会指出这个新网站是基于 Java 的网站。

现在你已经使用应用程序容器创建了网站，请参阅**后续步骤**部分，了解有关将应用程序上载到该网站的信息。

# 后续步骤

现在，你有了一台在 Azure 上作为 Java 网站运行的 Java 应用程序服务器。若要添加你自己的应用程序或网页，请参阅[将应用程序或网页添加到 Java 网站](/documentation/articles/web-sites-java-add-app)。

<!---HONumber=71-->