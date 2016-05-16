<properties 
	pageTitle="Azure 中的 Web 应用入门" 
	description="了解如何轻松地在 Azure Web 应用中实时运行 Web 应用。在 5 分钟内学会如何进行实际开发并立即查看结果。" 
	services="app-service\web"
	documentationCenter=""
	authors="cephalin" 
	manager="wpickett" 
	editor="" 
/>

<tags
	ms.service="app-service-web"
	ms.date="04/04/2016"
	wacn.date="05/16/2016"/>
	
# Azure 中的 Web 应用入门

本教程帮助你快速开始将 Web 应用部署到 [Azure Web 应用](/documentation/services/web-sites)。只需执行少量的操作，就可以：

- 部署示例 Web 应用（在 ASP.NET、PHP、Node.js、Java 或 Python 之间选择）。
- 在短短几秒内看到应用实时运行。
- 以推送 [Git](http://www.git-scm.com/) 提交内容的相同方式来更新 Web 应用。

另外，将提供 [Azure 管理门户](https://manage.windowsazure.cn)的速览并探讨可用的功能。

## 先决条件

若要完成本教程，你需要：

- Git。可在[此处](http://www.git-scm.com/downloads)下载二进制安装文件。你应该能够从选择的命令行终端运行 `git --version`。 
- 对 Git 有一个基本的了解。
- Azure CLI。[此处](/documentation/articles/xplat-cli-install)提供了安装说明。你应该能够从选择的命令行终端运行 `azure --version`。
- 一个 Azure 帐户。如果你没有帐户，可以[注册1元试用帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 部署 Web 应用

让我们将 Web 应用部署到 Azure。

1. 打开新的 Windows 命令提示符、Linux Shell 或 OS X 终端，执行 `CD` 切换到工作目录，然后克隆示例应用，如下所示：

        git clone <github_sample_url>

    对于 &lt;github\_sample\_url>，请使用下列其中一个 URL（根据所需的框架而定）：

    - ASP.NET：[https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git](https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git)
    - PHP (CodeIgniter)：[https://github.com/Azure-Samples/app-service-web-php-get-started.git](https://github.com/Azure-Samples/app-service-web-php-get-started.git)
    - Node.js (Express)：[https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git](https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git) 
    - Java：[https://github.com/Azure-Samples/app-service-web-java-get-started.git](https://github.com/Azure-Samples/app-service-web-java-get-started.git)
    - Python (Django)：[https://github.com/Azure-Samples/app-service-web-python-get-started.git](https://github.com/Azure-Samples/app-service-web-python-get-started.git)

2. 执行 `CD` 切换到示例应用的项目根目录。例如，

        cd app-service-web-dotnet-get-started

3. 如下所示登录到 Azure：

        azure login -e AzureChinaCloud -u <your account>
    
    根据提示继续登录你的 Azure 订阅。

4. 在 Azure 中以下一个命令创建具有唯一应用名称的 Azure Web 应用资源。

        azure site create --git <app_name>
      
    >[AZURE.NOTE] 如果从未设置 Azure 订阅的部署凭据，系统将提示你进行创建。Azure 只将这些凭据（而不是 Azure 帐户凭据）用于 Git 部署与 FTP 登录。
    
    现在已在 Azure 中创建应用。而且当前的目录也已进行 Git 初始化并作为 Git 远程连接到新的 Azure Web 应用。
    你可以浏览到应用的 URL (http://&lt;app_name>.chinacloudsites.cn) 以查看优雅的默认 HTML 页面，但让我们立即实际获取自己的代码。

4. 现在，将示例代码部署到新的 Azure Web 应用，如同使用 Git 推送任何代码一样：

        git push azure master 
    
    >[AZURE.NOTE] 系统将要求提供你的部署密码。如果你是 Azure Web 应用的新用户，请提供刚刚创建的部署密码，然后即可开始部署。
    
    `git push` 不仅将代码放在 Azure 中，也在部署引擎中触发部署任务。如果项目（存储库）根目录中有任何 package.json (Node.js) 或 requirements.txt (Python)，或 ASP.NET 项目中有 packages.config，则部署脚本将为你还原所需的包。

祝贺你，你的应用已部署到 Azure Web 应用。

## 查看应用实时运行

若要查看 Azure 中实时运行的应用，请从存储库中的任何目录运行以下命令：

    azure site browse

## 更新应用

现在可以使用 Git 随时从项目（存储库）根目录进行推送，以更新实时站点。此操作的方式如同首次将应用部署到 Azure。例如，每次想要推送已在本地测试的新更改时，只需从项目（存储库）根目录运行以下命令：
    
    git add .
    git commit -m "<your_message>"
    git push azure master

## 其他部署方式

有多种方式可以部署 Web 应用，而从本地存储库进行 Git 部署只是其中一种方式。你可以直接从 Visual Studio 部署、从 GitHub 部署、通过 FTP 上载文件，等等。有关部署选项的详细信息，请参阅[将你的应用部署到 Azure Web 应用](/documentation/articles/web-sites-deploy)。

## 在 Azure 管理门户中查看应用

现在，让我们转到 Azure 管理门户，查看所创建的应用：

1. 使用具有 Azure 订阅的帐户登录到 [Azure 管理门户](https://manage.windowsazure.cn)。

2. 在左栏中，单击“Web Apps”。

3. 单击刚刚创建的 Azure Web 应用，以在门户中打开其边栏选项卡。

Azure Web 应用的门户页提供了一组丰富的设置和工具，让你对应用进行配置、监视、保护和故障排除。请花点时间执行一些简单的任务，让自己熟悉此界面：

- 停止应用
- 重新启动应用
- 单击“配置”或“仪表板”可以查看有关应用的其他信息
- 单击“监视”可进行监视和故障排除  

## 后续步骤

将部署的应用提升到更高的级别。使用身份验证保护其安全。按需缩放。设置一些性能警报。所有这些操作只需按几下鼠标即可完成。

或者，进一步探索如何使用特定的语言框架创建适用于 Azure 的 Web 应用：

- [在 Azure 中创建 ASP.NET Web 应用](/documentation/articles/web-sites-dotnet-get-started)
- [在 Azure 中创建 PHP Web 应用](/documentation/articles/web-sites-php-mysql-deploy-use-git)
- [在 Azure 中创建 Node.js Web 应用](/documentation/articles/web-sites-nodejs-develop-deploy-mac)
- [在 Azure 中创建 Java Web 应用](/documentation/articles/web-sites-java-get-started)
- [在 Azure 中创建 Python Web 应用](/documentation/articles/web-sites-python-ptvs-django-mysql)

<!---HONumber=Mooncake_0509_2016-->