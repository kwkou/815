<properties linkid="develop-java-tutorials-web-site-add-app" urlDisplayName="Add an application to your Java web site" pageTitle="Add an application to your Java web site" metaKeywords="" description="This tutorial shows you how to add a page or application to your Java web site on Microsoft Azure." metaCanonical="" services="web-sites" documentationCenter="Java" title="Add an application to your Java web site" videoId="" scriptId="" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" />

# 将应用程序添加到 Azure 上的 Java 网站

按照 [Microsoft Azure 网站和 Java 入门][Microsoft Azure 网站和 Java 入门]中的说明初始化 Java 网站后，即可通过将 WAR 置于 **webapps** 文件夹中来上载应用程序。

**webapps** 文件夹的导航路径因你设置网站的方式不同而异。

-   如果是通过使用 Azure 应用程序库来设置网站，**webapps** 文件夹的路径就会是 **d:\\home\\site\\wwwroot\\bin\\application\_server\\webapps** 这样的形式，其中 **application\_server** 是对你的网站起作用的应用程序服务器的名称。
-   如果是通过使用 Azure 配置 UI 来设置网站，**webapps** 文件夹的路径就会是 **d:\\home\\site\\wwwroot\\webapps** 这样的形式。

请注意，你可以使用源代码管理来上载你的应用程序或网页，包括在连续集成方案中也这样做。有关源代码管理与你的网站配合使用的说明，请参阅[从源代码管理发布到 Azure 网站][从源代码管理发布到 Azure 网站]。对于上载应用程序或网页，FTP 也是一种可选择的方式。

请注意，对于 Tomcat 网站：在将 WAR 文件上载到 **webapps** 文件夹后，Tomcat 应用程序服务器将检测到你已经添加了该文件并会将其自动上载。请注意，如果你将文件（除 WAR 文件以外）复制到 ROOT 目录，在使用这些文件之前，将需要重新启动该应用程序服务器。Azure 上运行的 Tomcat Java 网站的自动上载功能基于所添加的新 WAR 文件，或添加到 **webapps** 文件夹的新文件或目录。

  [Microsoft Azure 网站和 Java 入门]: ../web-sites-java-get-started
  [从源代码管理发布到 Azure 网站]: ../web-sites-publish-source-control
