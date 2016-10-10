<properties 
	pageTitle="在 5 分钟内将第一个 Java Web 应用部署到 Azure | Azure" 
	description="了解如何部署示例应用，轻松地在应用服务中运行 Web 应用。快速进行实际的开发，立即查看结果。" 
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""
/>  


<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/16/2016"
	wacn.date="" 
	ms.author="cephalin"
/>  

	
# 在 5 分钟内将第一个 Java Web 应用部署到 Azure

本教程旨在帮助用户将一个简单的 Java Web 应用部署到 [Azure App Service](/documentation/articles/app-service-value-prop-what-is/)。应用服务可用于创建 Web 应用、[移动应用后端](/documentation/service/app-service/mobile/)和 [API 应用](/documentation/articles/app-service-api-apps-why-best-platform/)。

用户将能够：

- 在 Azure App Service 中创建 Web 应用。
- 部署示例 Java 应用。
- 查看代码在生产环境中的实时运行。

## 先决条件

- 获取 FTP/FTPS 客户端，如 [FileZilla](https://filezilla-project.org/)。
- 获取 Azure 帐户。如果你没有帐户，可以[注册试用版](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## <a name="create"></a> 创建 Web 应用

1. 使用 Azure 帐户登录到 [Azure 门户（预览版）](https://portal.azure.cn)。

2. 在左侧菜单中，单击“新建”>“Web + 移动”>“Web 应用”。

    ![](./media/app-service-web-get-started-languages/create-web-app-portal.png)  


3. 在应用创建边栏选项卡中，对新应用使用以下设置：

    - **应用名称**：键入唯一名称。
    - **资源组**：选择“新建”，为资源组指定名称。
    - **应用服务计划/位置**：单击进行配置，然后单击“新建”，设置应用服务计划的名称、位置和定价层。可随意使用“免费”定价层。

    完成后，应用创建边栏选项卡如下所示：

    ![](./media/app-service-web-get-started-languages/create-web-app-settings.png)  


3. 单击底部的“创建”。可以单击顶部的“通知”图标，查看进度。

    ![](./media/app-service-web-get-started-languages/create-web-app-started.png)  


4. 完成部署后，会看到此通知消息。单击该消息可打开部署的边栏选项卡。

    ![](./media/app-service-web-get-started-languages/create-web-app-finished.png)  


5. 在“成功的部署”边栏选项卡中，单击“资源”链接，打开新 Web 应用的边栏选项卡。

    ![](./media/app-service-web-get-started-languages/create-web-app-resource.png)  


## 将 Java 应用部署到 Web 应用

接下来，使用 FTPS 将 Java 应用部署到 Azure。

1. 使用以下 PowerShell 命令行设置“本地 Git 存储库”。

		$a = Get-AzureRmResource -ResourceId /subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Web/sites/<web app name>/Config/web -ApiVersion 2015-08-01

		$a.Properties.scmType = "LocalGit"

		Set-AzureRmResource -PropertyObject $a.Properties -ResourceId /subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Web/sites/<web app name>/Config/web -ApiVersion 2015-08-01

7. 返回 Azure 门户预览的 Web 应用边栏选项卡，单击“部署凭据”。

8. 设置部署凭据，单击“保存”。

7. 返回 Web 应用边栏选项卡，单击“概述”。单击“FTP/部署用户名”和“FTPS 主机名”旁的“复制”按钮，复制这些值。

    ![](./media/app-service-web-get-started-languages/get-ftp-url.png)  


    现在可以使用 FTPS 部署 Java 应用了。

8. 在 FTP/FTPS 客户端中，使用上一步中复制的值，登录到 Azure Web 应用的 FTP 服务器。使用之前创建的部署密码。

    以下屏幕截图显示的是使用 FileZilla 进行登录。

    ![](./media/app-service-web-get-started-languages/filezilla-login.png)  


    可能会看到 Azure 无法识别 SSL 证书的安全警告。继续。

9. 单击[此链接](https://github.com/Azure-Samples/app-service-web-java-get-started/raw/master/webapps/ROOT.war)，将 WAR 文件下载到本地计算机。

9. 在 FTP/FTPS 客户端中，导航到远程站点的 **/site/wwwroot/webapps**，将下载到本机计算机中的 WAR 文件拖到该远程目录。

    ![](./media/app-service-web-get-started-languages/transfer-war-file.png)  


    单击“确定”，覆盖 Azure 中的文件。

    >[AZURE.NOTE] 根据 Tomcat 的默认行为，/site/wwwroot/webapps 中的文件名 **ROOT.war** 提供根 Web 应用 (http:// *&lt;appname>* .chinacloudsites.cn)，文件名 ***&lt;anyname>*.war** 提供命名 Web 应用 (http:// *&lt;appname>* .chinacloudsites.cn/ *&lt;anyname>* )。

就这么简单！ 代码现在已在 Azure 中实时运行。在浏览器中，导航到 http:// *&lt;appname>* .chinacloudsites.cn，查看效果。

## 更新应用

需要更新时，只需将新的 WAR 文件上传到 FTP/FTPS 客户端中的同一个远程目录。

## 后续步骤

[根据 Azure 应用商店中的模板创建 Java Web 应用](/documentation/articles/app-service-web-java-get-started/#marketplace)。可以获取自己的完全自定义 Tomcat 容器，也可获取熟悉的管理器用户界面。

直接在 [IntelliJ](/documentation/articles/app-service-web-debug-java-web-app-in-intellij/) 或 [Eclipse](/documentation/articles/app-service-web-debug-java-web-app-in-eclipse/) 中调试 Azure Web 应用。

或者，对第一个 Web 应用执行更多操作。例如：

- 尝试[将代码部署到 Azure 的其他方法](/documentation/articles/web-sites-deploy/)。
- 使 Azure 应用上升到更高的层次。对用户进行身份验证。按需缩放。设置一些性能警报。所有这些操作只需按几下鼠标即可完成。请参阅[在第一个 Web 应用中添加功能](/documentation/articles/app-service-web-get-started-2/)。

<!---HONumber=Mooncake_0926_2016-->