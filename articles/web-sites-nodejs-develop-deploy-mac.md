<properties
	pageTitle="在 Azure 网站中创建 Node.js Web 应用"
	description="了解如何构建 Node.js Web 应用并在 Azure 中部署。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="MikeWasson"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="08/18/2015"
	wacn.date="10/03/2015"/>

# 构建 Node.js 网站并部署到 Azure

本教程将介绍如何创建 [Node][nodejs.org] 应用程序并使用 [Git] 将其部署到 Azure 网站。本教程中的说明适用于任何能够运行 Node 的操作系统。

如果您更愿意观看本教程的视频，下列剪辑将展示类似的步骤：[AZURE.VIDEO create-a-nodejs-site-deploy-from-github]
 
以下是已完成应用程序的屏幕快照：

![显示“Hello World”消息的浏览器。][helloworld-completed]

##创建 Azure 网站并启用 Git 发布

按照这些步骤操作创建一个 Azure 网站，然后对该网站启用 Git 发布。

> [AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial/?WT.mc_id=A7171371E" target="_blank">Azure 免费试用</a>。


1. 登录到 [Azure 管理门户]。

2. 单击该门户左下的“+ 新建”图标。

    ![突出显示了“+新建”链接的Azure 门户][portal-new-website]

3. 单击“网站”，然后单击“快速创建”。输入“URL”的值，并在“区域”下拉菜单中为你的网站选择数据中心。单击对话框底部的复选标记。

    ![“快速创建”对话框][portal-quick-create]

4. 网站状态更改为“正在运行”之后，单击网站名称以访问“仪表板”。

	![打开网站仪表板][go-to-dashboard]

6. 在“快速启动”页的右下方，选择“从源代码管理设置部署”。

	![设置 Git 发布][setup-git-publishing]

6. 当系统询问“你的源代码在哪里？”时，选择“本地 Git 存储库”，然后单击箭头。

	![你的源代码在哪里][where-is-code]

7. 若要启用 Git 发布，必须提供用户名和密码。如果您先前已对某个 Azure 网站启用了发布，则系统不会提示您输入用户名或密码。而是用您先前指定的用户名和密码创建一个 Git 存储库。记下此用户名和密码，因为它们将用于您创建的所有 Azure 网站的 Git 发布。

	![提示输入用户名和密码的对话框。][portal-git-username-password]

8. 一旦 Git 存储库准备就绪，就会显示有关要用于设置本地存储库并将文件推送到 Azure 的 Git 命令的说明。

	![为网站创建存储库后返回的 Git 部署说明。][git-instructions]

##本地构建和测试应用程序

在本部分中，你将从 [nodejs.org] 创建包含“hello world”示例的 **server.js** 文件。通过添加 process.env.PORT 作为在 Azure 网站中运行时要侦听的端口，对原始示例进行修改得到了此示例。

1. 使用文本编辑器在 **helloworld** 目录中创建名为 **server.js** 的新文件。如果 **helloworld** 目录不存在，请创建一个。
2. 在**server.js** 文件中添加以下内容，然后保存：

        var http = require('http')
        var port = process.env.PORT || 1337;
        http.createServer(function(req, res) {
          res.writeHead(200, { 'Content-Type': 'text/plain' });
          res.end('Hello World\n');
        }).listen(port);

3. 打开命令行，使用下面的命令在本地启动网页：

        node server.js

4. 打开 Web 浏览器并导航到 http://localhost:1337。一个显示"Hello World"网页将显示在下面的屏幕快照中，如下所示：

    ![显示“Hello World”消息的浏览器。][helloworld-localhost]

##发布应用程序

1. 从命令行中，将目录更改为 **helloworld** 目录，然后输入下面的命令来初始化本地 Git 存储库。 

		git init

	> [AZURE.NOTE]**Git 命令不可用？** [Git](http://git-scm.com/) 是一个分布式版本控制系统，可用来部署 Azure 网站。有关针对你的平台的安装说明，请参阅 [Git 下载页](http://git-scm.com/download)。

2. 使用以下命令将文件添加到存储库中：

		git add .
		git commit -m "initial commit"

3. 使用以下命令添加 Git remote，以便将更新推送到您之前创建的 Azure 网站：

		git remote add azure [URL for remote repository]

    ![为网站创建存储库后返回的 Git 部署说明。][git-instructions]
 
4. 使用以下命令将更改推送到 Azure：

		git push azure master

	系统将提示你输入之前创建的密码。输出应如下所示：

		Password for 'testsite.scm.chinacloudsites.cn':
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
    
	如果您在管理门户内导航到 Azure 网站的部署选项卡，您将在部署历史记录中看到您的第一个部署：

	![门户上的 Git 部署状态][git-deployments-first]

5. 若要查看你的应用，请在管理门户的“Web 应用”部分中单击“浏览”按钮。

##发布对应用程序所做的更改

1. 在文本编辑器中打开 **server.js** 文件，将“Hello World\\n”更改为“Hello Azure\\n”。保存文件。
2. 从命令行中，将目录更改为 **helloworld** 目录，然后运行以下命令：

		git add .
		git commit -m "changing to hello azure"
		git push azure master

	系统将提示你输入之前创建的密码。如果您在管理门户内导航到 Azure 网站的部署选项卡，您会看到更新的部署历史记录：
	
	![门户上更新的 Git 部署状态][git-deployments-second]

3. 单击“浏览”按钮浏览到你的应用，你将发现已应用上述更新。

	![显示“Hello Azure”的网页][helloworld-completed]

4. 要恢复到以前的部署，可以在“部署”中选择该部署。


##后续步骤

虽然本文中的步骤使用 Azure 门户来创建网站，但你也可以使用[适用于 Mac 和 Linux 的 Azure 命令行工具]执行同样的操作。

Node.js 提供可由您的应用程序使用的丰富的模块生态系统。若要了解 Azure 网站是如何使用模块的，请参阅[将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)。

若要了解有关随 Azure 提供的 Node.js 版本的更多信息以及如何指定要与你的应用程序结合使用的版本，请参阅[在 Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps/)。

如果你将应用程序部署到 Azure 后遇到问题，有关如何诊断问题的信息，请参阅[如何在 Azure 网站中调试 Node.js 应用程序](/documentation/articles/web-sites-nodejs-debug/)。


##其他资源

* [Azure PowerShell]
* [适用于 Mac 和 Linux 的 Azure 命令行工具]

[Azure PowerShell]: /documentation/articles/install-configure-powershell/

[nodejs.org]: http://nodejs.org
[Git]: http://git-scm.com

[Azure 管理门户]: http://manage.windowsazure.cn
[适用于 Mac 和 Linux 的 Azure 命令行工具]: /documentation/articles/xplat-cli/

[helloworld-completed]: ./media/web-sites-nodejs-develop-deploy-mac/helloazure.png
[helloworld-localhost]: ./media/web-sites-nodejs-develop-deploy-mac/helloworldlocal.png
[portal-new- Website]: ./media/web-sites-nodejs-develop-deploy-mac/plus-new.png
[portal-quick-create]: ./media/web-sites-nodejs-develop-deploy-mac/create-quick- Website.png

[portal-git-username-password]: ./media/web-sites-nodejs-develop-deploy-mac/git-deployment-credentials.png
[git-instructions]: ./media/web-sites-nodejs-develop-deploy-mac/git-instructions.png

[git-deployments-first]: ./media/web-sites-nodejs-develop-deploy-mac/git_deployments_first.png
[git-deployments-second]: ./media/web-sites-nodejs-develop-deploy-mac/git_deployments_second.png

[setup-git-publishing]: ./media/web-sites-nodejs-develop-deploy-mac/setup_git_publishing.png
[go-to-dashboard]: ./media/web-sites-nodejs-develop-deploy-mac/go_to_dashboard.png
[where-is-code]: ./media/web-sites-nodejs-develop-deploy-mac/where_is_code.png

<!---HONumber=71-->