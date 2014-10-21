<properties linkid="develop-java-tutorials-web-site-get-started" urlDisplayName="Get started with Azure" pageTitle="Get started with Microsoft Azure Web Sites using Java" metaKeywords="" description="This tutorial shows you how to deploy a Java web site to Microsoft Azure." metaCanonical="" services="web-sites" documentationCenter="Java" title="Get started with Azure and Java" videoId="" scriptId="" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" />

# Azure 网站和 Java 入门

本教程演示如何使用 Java 在 Microsoft Azure 上（使用 Azure 应用程序库或 Azure 网站配置 UI）创建网站。

如果你不想使用这些技术中的任何一种，例如，如果你想自定义应用程序容器，请参阅[将自定义 Java 网站上载到 Azure][将自定义 Java 网站上载到 Azure]。

> [WACOM.NOTE] 若要完成本教程，你需要一个 Microsoft Azure 帐户。如果你没有帐户，则可以[激活 MSDN 订户权益][激活 MSDN 订户权益]或[注册免费试用][注册免费试用]。

# 使用 Azure 应用程序库创建 Java 网站

此信息讲解如何使用 Azure 应用程序库为你的网站选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

下面演示从应用程序库中使用 Tomcat 生成的网站的外观会是怎样的：

![使用 Apache Tomcat 的网站][使用 Apache Tomcat 的网站]

下面演示从应用程序库中使用 Jetty 生成的网站的外观会是怎样的：

![使用 Jetty 的网站][使用 Jetty 的网站]

1.  登录到 Microsoft Azure 管理门户。
2.  依次单击“新建”、“计算”、“网站”和“从库中”。
3.  从应用程序列表中，选择 Java 应用程序服务器之一，如 **Apache Tomcat** 或 **Jetty**。
4.  单击“下一步”。
5.  指定 URL 名称。
6.  选择区域。例如，**China East**。
7.  单击“完成”。

你的网站将在几分钟之内创建好。若要查看网站，请在 Azure 管理门户的“网站”视图中，等待状态显示为“正在运行”，然后单击该网站的 URL。

现在你已经使用应用程序容器创建了网站，请参阅“后续步骤”部分，了解有关将应用程序上载到该网站的信息。

# 使用 Azure 配置 UI 创建 Java 网站

此信息讲解如何使用 Azure 配置 UI 为你的网站选择 Java 应用程序容器（Apache Tomcat 或 Jetty）。

1.  登录到 Microsoft Azure 管理门户。
2.  依次单击“新建”、“计算”、“网站”，和“快速创建”。
3.  指定 URL 名称。
4.  选择区域。例如，**China East**。
5.  单击“完成”。你的网站将在几分钟之内创建好。若要查看网站，请在 Azure 管理门户的“网站”视图中，等待状态显示为“正在运行”，然后单击该网站的 URL。
6.  还是在 Azure 管理门户的“网站”视图中，单击你网站的名称以打开
    仪表板。
7.  单击**“配置”**。
8.  在“常规”部分，通过单击可用的版本启用 **Java**。
9.  Web 容器的选项会显示出来，例如，Tomcat 和 Jetty。选择要使用的 Web 容器。
10. 单击**“保存”**。

你的网站将在几分钟之内变成基于 Java 的。要确认该网站是基于 Java 的，请在 Azure 管理门户内的“网站”视图中，等待状态显示为“正在运行”，然后单击该网站的 URL。请注意，页面上提供的文本会指出这个新网站是基于 Java 的网站。

现在你已经使用应用程序容器创建了网站，请参阅“后续步骤”部分，了解有关将应用程序上载到该网站的信息。

# 后续步骤

现在，你有了一台在 Azure 上作为 Java 网站运行的 Java 应用程序服务器。若要添加你自己的应用程序或网页，请参阅[将应用程序或网页添加到 Java 网站][将应用程序或网页添加到 Java 网站]。

  [将自定义 Java 网站上载到 Azure]: ../web-sites-java-custom-upload
  [激活 MSDN 订户权益]: /en-us/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F
  [注册免费试用]: /en-us/pricing/free-trial/?WT.mc_id=A261C142F
  [使用 Apache Tomcat 的网站]: ./media/web-sites-java-get-started/tomcat.png
  [使用 Jetty 的网站]: ./media/web-sites-java-get-started/jetty.png
  [将应用程序或网页添加到 Java 网站]: ../web-sites-java-add-app
