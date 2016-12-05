<properties
    pageTitle="将 Java 应用程序添加到 Azure 应用服务 Web 应用"
    description="本教程介绍如何将页面或应用程序添加到已配置为使用 Java 的 Azure 应用服务 Web 应用实例。"
    services="app-service\web"
    documentationcenter="java"
    author="rmcmurray"
    manager="erikre"
    editor="" />  

<tags
    ms.assetid="9b46528b-e2d0-4f26-b8d7-af94bd8c31ef"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="12/05/2016"
    ms.author="robmcm" />  


# 将 Java 应用程序添加到 Azure 应用服务 Web 应用
按照[在 Azure 应用服务中创建 Java Web 应用](/documentation/articles/web-sites-java-get-started/)中的说明初始化 [Azure 应用服务][Azure App Service]中的 Java Web 应用后，可通过将 WAR 放置在 **webapps** 文件夹上载应用程序。

**webapps** 文件夹的导航路径因用户设置 Web 应用实例的方式而异。

* 如果使用 Azure 配置 UI 设置 Web 应用，则 **webapps** 文件夹的路径格式为 **d:\\home\\site\\wwwroot\\webapps**。

请注意，可使用源代码管理上载应用程序或网页，即使在[连续集成方案](/documentation/articles/app-service-continuous-deployment/)中亦是如此。也可使用 FTP 上载应用程序或网页；如需详细了解如何通过 FTP 部署应用程序，请参阅[将应用部署到 Azure 应用服务]。

Tomcat Web 应用说明：将 WAR 文件上载到 **webapps** 文件夹后，Tomcat 应用程序服务器会检测到已添加该文件并自动上载。请注意，如果将文件（除 WAR 文件以外）复制到 ROOT 目录，在使用这些文件之前，将需要重新启动该应用服务器。Azure 上运行的 Tomcat Java Web 应用的自动上载功能基于所添加的新 WAR 文件，或添加到 **webapps** 文件夹的新文件或目录。

## <a name="see-also"></a> 另请参阅
有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

[AZURE.INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]

<!-- URL List -->


[Azure Java 开发人员中心]: /develop/java/
[Azure App Service]: /documentation/articles/app-service-changes-existing-services/
[将应用部署到 Azure 应用服务]: /documentation/articles/web-sites-deploy/

<!---HONumber=Mooncake_1128_2016-->