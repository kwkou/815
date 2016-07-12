<properties 
	pageTitle="在 5 分钟内将第一个 Web 应用部署到 Azure" 
	description="了解如何通过几个步骤来部署一个示例应用，从而轻松地在 Azure 中运行 Web 应用。在 5 分钟内学会如何进行实际开发并立即查看结果。" 
	services="app-service\web"
	documentationCenter=""
	authors="cephalin" 
	manager="wpickett" 
	editor="" 
/>

<tags
	ms.service="app-service-web"
	ms.date="05/12/2016"
	wacn.date="05/30/2016"/>
	
# 在 5 分钟内将第一个 Web 应用部署到 Azure

[AZURE.INCLUDE [选项卡](../includes/app-service-web-get-started-nav-tabs.md)]

本教程帮助你将第一个 Web 应用部署到 [Azure Web 应用](/documentation/services/web-sites)。Azure 允许你创建 Web 应用。

只需执行少量的操作，就可以：

- 部署示例 Web 应用（在 ASP.NET、PHP、Node.js、Java 或 Python 之间选择）。
- 在短短几秒内看到应用实时运行。
- 以[推送 Git 提交](https://git-scm.com/docs/git-push)的相同方式来更新 Web 应用。

另外，将提供 [Azure 经典管理门户](https://manage.windowsazure.cn)的速览并探讨可用的功能。

##<a name="Prerequisites"></a> 先决条件

- [安装 Git](http://www.git-scm.com/downloads)。 
- [安装 Azure CLI](/documentation/articles/xplat-cli-install/)。 
- 获取 Azure 帐户。如果你没有帐户，可以[注册试用版](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 部署 Web 应用

让我们将 Web 应用部署到 Azure。

1. 打开新的 Windows 命令提示符、PowerShell 窗口、Linux shell 或 OS X 终端。运行 `git --version` 和 `azure --version` 验证你的计算机上是否已安装 Git 和 Azure CLI。 

    ![在 Azure 中测试第一个 Web 应用的 CLI 工具安装](./media/app-service-web-get-started/1-test-tools.png)

    如果尚未安装这些工具，请参阅[先决条件](#Prerequisites)中的下载链接。

1. 执行 `CD` 切换到工作目录并克隆示例应用，如下所示：

        git clone <github_sample_url>

    ![在 Azure 中克隆第一个 Web 应用的应用示例代码](./media/app-service-web-get-started/2-clone-sample.png)

    对于 *&lt;github\_sample\_url>*，请使用下列其中一个 URL（根据所需的框架而定）：

    - HTML+CSS+JS：[https://github.com/Azure-Samples/app-service-web-html-get-started.git](https://github.com/Azure-Samples/app-service-web-html-get-started.git)
    - ASP.NET：[https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git](https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git)
    - PHP (CodeIgniter)：[https://github.com/Azure-Samples/app-service-web-php-get-started.git](https://github.com/Azure-Samples/app-service-web-php-get-started.git)
    - Node.js (Express)：[https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git](https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git) 
    - Java：[https://github.com/Azure-Samples/app-service-web-java-get-started.git](https://github.com/Azure-Samples/app-service-web-java-get-started.git)
    - Python (Django)：[https://github.com/Azure-Samples/app-service-web-python-get-started.git](https://github.com/Azure-Samples/app-service-web-python-get-started.git)

2. 执行 `CD` 切换到示例应用的存储库。例如，

        cd app-service-web-html-get-started

3. 如下所示登录到 Azure：

        azure login -e AzureChinaCloud -u <your account>

4. 在 Azure 中以下一个命令创建具有唯一应用名称的 Azure Web 应用资源。出现提示时，请指定所需区域数目。

        azure site create --git <app_name>
    
    ![在 Azure 中创建第一个 Web 应用的 Azure 资源](./media/app-service-web-get-started/4-create-site.png)
    
    >[AZURE.NOTE] 如果从未设置 Azure 订阅的部署凭据，系统将提示你进行创建。Azure 只将这些凭据（而不是 Azure 帐户凭据）用于 Git 部署与 FTP 登录。
    
    现在已在 Azure 中创建应用。而且当前的目录也已进行 Git 初始化并作为 Git 远程连接到新的 Azure Web 应用。
    你可以浏览到应用的 URL (http://&lt;app_name>.chinacloudsites.cn) 以查看优雅的默认 HTML 页面，但让我们立即实际获取自己的代码。

4. 现在，将示例代码部署到新的 Azure Web 应用，如同使用 Git 推送任何代码一样：

        git push azure master 

    ![在 Azure 中将代码推送到第一个 Web 应用](./media/app-service-web-get-started/5-push-code.png)
    
    如果你使用了某种语言框架，看到的输出将与上面所示不同。这是因为，`git push` 不仅会将代码放在 Azure 中，而且会在部署引擎中触发部署任务。如果项目（存储库）根目录中有任何 package.json (Node.js) 或 requirements.txt (Python)，或 ASP.NET 项目中有 packages.config，则部署脚本将为你还原所需的包。

祝贺你，你的应用已部署到 Azure Web 应用。

## 查看应用实时运行

若要查看 Azure 中实时运行的应用，请从存储库中的任何目录运行以下命令：

    azure site browse

## 更新应用

现在可以使用 Git 随时从项目（存储库）根目录进行推送，以更新实时站点。此操作的方式如同首次将应用部署到 Azure。例如，每次想要推送已在本地测试的新更改时，只需从项目（存储库）根目录运行以下命令：
    
    git add .
    git commit -m "<your_message>"
    git push azure master

## 在 Azure 经典管理门户中查看应用

现在，让我们转到 Azure 经典管理门户，以查看所创建的应用：

1. 使用具有 Azure 订阅的帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在左栏中，单击“Web Apps”。

3. 单击刚刚创建的 Azure Web 应用，以在经典管理门户中打开。

Azure Web 应用的经典管理门户页提供了一组丰富的设置和工具，让你对应用进行配置、监视、保护和故障排除。请花点时间执行一些简单的任务，让自己熟悉此界面：

- 停止应用
- 重新启动应用
- 单击“配置”或“仪表板”可以查看有关应用的其他信息
- 单击“监视”可进行监视和故障排除  

## 后续步骤

- 除了使用 Git 和 Azure CLI 以外，还可以使用其他方式将 Web 应用部署到 Azure（请参阅 [Deploy your app to Azure Web App（将你的应用部署到 Azure Web 应用）](/documentation/articles/web-sites-deploy/)）。根据你的语言框架找到所需的开发和部署步骤，只需在本文顶部选择你的框架即可。

<!---HONumber=Mooncake_0523_2016-->