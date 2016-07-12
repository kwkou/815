<properties
	pageTitle="在 Azure 中创建 Node.js Web 应用 | Azure"
	description="学习如何将 Node.js 应用程序部署到 Azure 中的 Web 应用。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>

# 在 Azure 中创建 Node.js Web 应用

> [AZURE.SELECTOR]
- [.Net](/documentation/articles/web-sites-dotnet-get-started/)
- [Node.js](/documentation/articles/web-sites-nodejs-develop-deploy-mac/)
- [Java](/documentation/articles/web-sites-java-get-started/)
- [PHP - Git](/documentation/articles/web-sites-php-mysql-deploy-use-git/)
- [PHP - FTP](/documentation/articles/web-sites-php-mysql-deploy-use-ftp/)
- [Python](/documentation/articles/web-sites-python-ptvs-django-mysql/)

本教程说明如何创建一个简单的 [Node.js](http://nodejs.org) 应用程序，并使用 [Git](http://git-scm.com) 将其部署到 [Azure Web 应用](/documentation/services/web-sites)中的 [Web 应用](/home/features/web-site)。本教程中的说明适用于任何能够运行 Node.js 的操作系统。

学习内容：

* 如何使用 Azure 经典管理门户在 Azure 中创建 Web 应用。
* 如何通过推送到 Web 应用的 Git 存储库，将 Node.js 应用程序部署到 Web 应用。

已完成的应用程序将简短的“hello world”字符串写入浏览器。

![显示“Hello World”消息的浏览器。][helloworld-completed]

有关教程以及适用于更复杂 Node.js 应用程序的示例代码，或有关如何在 Azure 中使用 Node.js 的其他主题，请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

> [AZURE.NOTE]
若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以[注册试用版](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 创建 Web 应用并启用 Git 发布

请按照以下步骤在 Azure 中创建 Web 应用，然后启用 Git 发布。

[Git](http://git-scm.com/) 是一个分布式版本控制系统，可用来部署 Azure Web 应用。你将在本地 Git 存储库中存储你为 Web 应用编写的代码，并通过推送到远程存储库将代码部署到 Azure。这种部署方法是 Azure Web 应用的一项功能。

1. 登录到 [Azure 经典管理门户]。

2. 单击该经典管理门户左下的“+ 新建”图标。

    ![突出显示了“+新建”链接的Azure 经典管理门户][portal-new-website]

3. 单击“ Web 应用”，然后单击“快速创建”。输入“URL”的值，并在“区域”下拉菜单中为你的 Web 应用选择数据中心。单击对话框底部的复选标记。

    ![“快速创建”对话框][portal-quick-create]

4. Web 应用状态更改为“正在运行”之后，单击 Web 应用名称以访问“仪表板”。

	![打开 Web 应用仪表板][go-to-dashboard]

6. 在“快速启动”页的右下方，选择“从源代码管理设置部署”。

	![设置 Git 发布][setup-git-publishing]

6. 当系统询问“你的源代码在哪里？”时，选择“本地 Git 存储库”，然后单击箭头。

	![你的源代码在哪里][where-is-code]

7. 若要启用 Git 发布，必须提供用户名和密码。如果您先前已对某个 Azure Web 应用启用了发布，则系统不会提示您输入用户名或密码。而是用您先前指定的用户名和密码创建一个 Git 存储库。记下此用户名和密码，因为它们将用于您创建的所有 Azure Web 应用的 Git 发布。

	![提示输入用户名和密码的对话框。][portal-git-username-password]

8. 一旦 Git 存储库准备就绪，就会显示有关要用于设置本地存储库并将文件推送到 Azure 的 Git 命令的说明。

	![为 Web 应用创建存储库后返回的 Git 部署说明。][git-instructions]

## 本地构建和测试应用程序

在本部分中，你将创建一个 **server.js** 文件，其中包含已稍做修改的“Hello World”示例版本（来自 [nodejs.org]）。代码将添加 process.env.PORT，作为在 Azure Web 应用中运行时要侦听的端口。

1. 创建名为 *helloworld* 的目录。

2. 使用文本编辑器在 *helloworld* 目录中创建名为 **server.js** 的新文件。

2. 将以下代码复制到 **server.js** 文件，然后保存该文件：

        var http = require('http')
        var port = process.env.PORT || 1337;
        http.createServer(function(req, res) {
          res.writeHead(200, { 'Content-Type': 'text/plain' });
          res.end('Hello World\n');
        }).listen(port);

3. 打开命令行，并使用以下命令在本地启动 Web 应用。

        node server.js

4. 打开 Web 浏览器并导航到 http://localhost:1337。

	显示“Hello World”的网页随即出现，如以下屏幕截图所示。

    ![显示“Hello World”消息的浏览器。][helloworld-localhost]

## 发布应用程序

1. 如果你尚未安装 Git，现在请安装。

	有关针对你的平台的安装说明，请参阅 [Git 下载页](http://git-scm.com/download)。

1. 从命令行中，将目录更改为 **helloworld** 目录，然后输入以下命令来初始化本地 Git 存储库。

		git init


2. 使用以下命令将文件添加到存储库中：

		git add .
		git commit -m "initial commit"

3. 使用以下命令添加 Git remote，以便将更新推送到你之前创建的 Web 应用：

		git remote add azure [URL for remote repository]

4. 使用以下命令将更改推送到 Azure：

		git push azure master

	系统将提示你输入之前创建的密码。输出类似于以下示例。

		Counting objects: 3, done.
		Delta compression using up to 8 threads.
		Compressing objects: 100% (2/2), done.
		Writing objects: 100% (3/3), 374 bytes, done.
		Total 3 (delta 0), reused 0 (delta 0)
		remote: New deployment received.
		remote: Updating branch 'master'.
		remote: Preparing deployment for commit id '5ebbe250c9'.
		remote: Preparing files for deployment.
		remote: Deploying Web.config to enable Node.js activation.
		remote: Deployment successful.
		To https://user@testsite.scm.chinacloudsites.cn/testsite.git
		 * [new branch]      master -> master

5. 若要查看你的应用，请在 Azure 经典管理门户的“Web 应用”部分中单击“浏览”按钮。

## 发布对应用程序所做的更改

1. 在文本编辑器中打开 **server.js** 文件，将“Hello World\\n”更改为“Hello Azure\\n”。 

2. 保存文件。

2. 从命令行中，将目录更改为 **helloworld** 目录，然后运行以下命令：

		git add .
		git commit -m "changing to hello azure"
		git push azure master

	系统将再次提示你输入密码。

3. 刷新你从中导航到 Web 应用 URL 的浏览器窗口。

	![显示“Hello Azure”的网页][helloworld-completed]
4. 要恢复到以前的部署，可以在“部署”中选择该部署。

## 后续步骤

你已将 Node.js 应用程序部署到 Azure 中的 Web 应用。若要详细了解 Azure Web 应用如何运行 Node.js 应用程序，请参阅 [Azure Web Apps：Node.js](http://blogs.msdn.com/b/silverlining/archive/2012/06/14/windows-azure-websites-node-js.aspx) 和[在 Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps/)。

Node.js 提供可由您的应用程序使用的丰富的模块生态系统。若要了解 Web Apps 是如何使用模块的，请参阅[将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)。

如果你将应用程序部署到 Azure 后遇到问题，请参阅[如何在 Azure Web 应用中调试 Node.js 应用程序](/documentation/articles/web-sites-nodejs-debug/)，以了解有关诊断问题的信息。

本文将使用 Azure 经典管理门户来创建 Web 应用。你也可以使用 [Azure 命令行界面](/documentation/articles/xplat-cli-install/)或 [Azure PowerShell](/documentation/articles/powershell-install-configure/) 执行相同的操作。

有关如何在 Azure 上开发 Node.js 应用程序的详细信息，请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

[Azure 经典管理门户]: http://manage.windowsazure.cn
[Azure Command-Line Tools for Mac and Linux]: /documentation/articles/xplat-cli-install/
[Azure PowerShell]: /documentation/articles/powershell-install-configure/
[portal-new-website]: ./media/web-sites-nodejs-develop-deploy-mac/plus-new.png
[portal-git-username-password]: ./media/web-sites-nodejs-develop-deploy-mac/git-deployment-credentials.png
[git-instructions]: ./media/web-sites-nodejs-develop-deploy-mac/git-instructions.png
[git-deployments-first]: ./media/web-sites-nodejs-develop-deploy-mac/git_deployments_first.png
[git-deployments-second]: ./media/web-sites-nodejs-develop-deploy-mac/git_deployments_second.png
[where-is-code]: ./media/web-sites-nodejs-develop-deploy-mac/where_is_code.png
[helloworld-completed]: ./media/web-sites-nodejs-develop-deploy-mac/helloazure.png
[helloworld-localhost]: ./media/web-sites-nodejs-develop-deploy-mac/helloworldlocal.png
[portal-quick-create]: ./media/web-sites-nodejs-develop-deploy-mac/create-quick-website.png
[portal-quick-create2]: ./media/web-sites-nodejs-develop-deploy-mac/create-quick-website2.png
[setup-git-publishing]: ./media/web-sites-nodejs-develop-deploy-mac/setup_git_publishing.png
[go-to-dashboard]: ./media/web-sites-nodejs-develop-deploy-mac/go_to_dashboard.png
[deployment-part]: ./media/web-sites-nodejs-develop-deploy-mac/deployment-part.png
[deployment-credentials]: ./media/web-sites-nodejs-develop-deploy-mac/deployment-credentials.png
[git-url]: ./media/web-sites-nodejs-develop-deploy-mac/git-url.png

<!---HONumber=Mooncake_0215_2016-->