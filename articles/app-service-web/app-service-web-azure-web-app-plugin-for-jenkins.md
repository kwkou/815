<!-- need to be verified -->

<properties
    pageTitle="如何在 Jenkins 连续集成中使用 Azure Web 应用插件 | Azure"
    description="说明如何在 Jenkins 连续集成中使用 Azure Web 应用插件。"
    services="app-service\web"
    documentationcenter=""
    author="rmcmurray"
    manager="erikre"
    editor="" />
<tags 
    ms.assetid="be6c4e62-da76-44f6-bb00-464902734805"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="java"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="12/05/2016"
    ms.author="robmcm" />

# 如何在 Jenkins 连续集成中使用 Azure Web 应用插件
使用适用于 Jenkins 的 Azure Web 应用插件，可以在运行分布式构建时轻松地在 Azure 上创建 Web 应用，并将 WAR 文件部署到 Web 应用。

### 先决条件
执行本文中的步骤之前，需要注册和授权客户端应用程序，然后检索客户端 ID 和客户端密钥，在身份验证期间会将这些信息发送到 Azure Active Directory。有关这些先决条件的详细信息，请参阅以下文章：

* [将应用程序与 Azure Active Directory 集成][integrate-apps-with-AAD]
* [注册客户端应用][register-client-app]

此外，将需要从以下 URL 下载 **azure-webapp-plugin.hpi** 文件：

* [https://github.com/Microsoft/azure-webapp-plugin/tree/master/install](https://github.com/Microsoft/azure-webapp-plugin/tree/master/install)

## 如何安装适用于 Jenkins 的 Azure Web 应用插件
1. [从 GitHub 下载 **azure-webapp-plugin.hpi** 文件][azure-webapp-plugin-install]
2. 登录到 Jenkins 仪表板。
3. 在该仪表板中，单击“管理 Jenkins”。
   
    ![管理 Jenkins][jenkins-dashboard]  

4. 在“管理 Jenkins”页中，单击“管理插件”。
   
    ![管理插件][manage-jenkins]  

5. 单击“高级”选项卡，然后单击“上载插件”部分中的“浏览”。导航到在“先决条件”中将 **azure-webapp-plugin.hpi** 文件下载到的位置，然后在选择该文件后单击“上载”。
   
    ![上载插件][upload-plugin]  

6. 如有必要，重启 Jenkins。

## 配置 Azure Web 应用插件
1. 在 Jenkins 仪表板中，单击其中一个项目。
   
    ![选择项目][select-project]  

2. 在该项目的页面出现后，单击左侧菜单中的“配置”。
   
    ![配置项目][configure-project]  

3. 在“后期生成操作”部分中，单击“添加后期生成操作”下拉菜单，然后选择“Azure Webapp 配置”。
   
    ![高级选项][advanced-options]  

4. 在“Azure Webapp 配置”部分出现后，在“Azure 配置文件配置”中输入**订阅 ID**、**客户端 ID**、**客户端机密**和 **OAuth 2.0 令牌终结点**信息。
   
    ![Azure 配置文件配置][azure-profile-configuration]  

5. 在“Webapp 配置”部分中，输入**资源组名称**、**位置**、**托管计划名称**、**Web 应用名称**、**Sku 名称**、**Sku 容量**和 **War 文件路径**信息。
   
    ![Webapp 配置][webapp-configuration]  

6. 单击“保存”保存项目的这些设置。
   
    ![保存项目][save-project]  

7. 单击左侧菜单中的“立即生成”。
   
    ![生成项目][build-project]  


> [AZURE.NOTE]
仅当 Web 应用容器尚未存在时，创建 Web 应用容器。
> 
> 

## <a name="see-also"></a> 另请参阅
有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]。

有关适用于 Jenkins 的 Azure Web 应用插件的更多信息，请参阅 GitHub 上的 [Azure Web 应用插件]项目。

<!-- URL List -->


[Azure Java 开发人员中心]: /develop/java/
[integrate-apps-with-AAD]: /documentation/articles/active-directory-integrating-applications/
[register-client-app]: https://powerbi.microsoft.com/zh-cn/documentation/powerbi-developer-register-a-client-app/
[Azure Web 应用插件]: https://github.com/Microsoft/azure-webapp-plugin
[azure-webapp-plugin-install]: https://github.com/Microsoft/azure-webapp-plugin/tree/master/install

<!-- IMG List -->


[jenkins-dashboard]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/jenkins-dashboard.png
[manage-jenkins]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/manage-jenkins.png
[upload-plugin]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/upload-plugin.png
[select-project]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/select-project.png
[configure-project]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/configure-project.png
[advanced-options]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/advanced-options.png
[azure-profile-configuration]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/azure-profile-configuration.png
[build-project]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/build-project.png
[save-project]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/save-project.png
[webapp-configuration]: ./media/app-service-web-azure-web-app-plugin-for-jenkins/webapp-configuration.png

<!---HONumber=Mooncake_1128_2016-->