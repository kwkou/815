<properties
	pageTitle="指定 Node.js 版本"
	description="了解如何指定 Azure 网站和云服务使用的 Node.js 版本"
	services=""
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="multiple"
	ms.date="05/04/2016"
	wacn.date="06/20/2016"/>

# 在 Azure 应用程序中指定 Node.js 版本

托管 Node.js 应用程序时，您可能希望确保您的应用程序使用特定版本的 Node.js。有几种方法可为托管在 Azure 上的应用程序完成此操作。

## 默认版本

由 Azure 提供的 Node.js 版本会不断更新。除非另行指定，否则将使用最新可用版本。

> [AZURE.NOTE] 如果你在 Azure 云服务（Web 角色或辅助角色）中托管应用程序，并且这是你首次部署应用程序，Azure 将尝试使用你在部署环境中所安装的 Node.js 的版本（如果该版本与 Azure 中提供的默认版本之一相匹配）。

## 使用 package.json 进行版本控制

可通过将以下内容添加到你的 **package.json** 文件中来指定要使用的 Node.js 的版本：

	"engines":{"node":version}

其中，*version* 是要使用的特定版本号。你可以为版本指定更复杂的条件，例如：

	"engines":{"node": "0.6.22 || 0.8.x"}

由于 0.6.22 不是托管环境中提供的版本之一，将改为使用 0.8 系列中的最高版本 - 0.8.4。

## 使用应用设置对网站进行版本控制
如果你在网站中托管应用程序，则可以将环境变量 **WEBSITE\_NODE\_DEFAULT\_VERSION** 设置为所需版本。

## 使用 PowerShell 对云服务进行版本控制

如果你在云服务中托管应用程序，并且使用 Azure PowerShell 部署该应用程序，则可使用 **Set-AzureServiceProjectRole** PowerShell cmdlet 替代默认的 Node.js 版本。例如：

	Set-AzureServiceProjectRole WebRole1 Node 0.8.4

请注意，上述语句中的参数区分大小写。通过检查角色的 **package.json** 中的 **engines** 属性可以验证是否已选择正确的 Node.js 版本。

你还可使用 **Get-AzureServiceProjectRoleRuntime** 检索托管为云服务的应用程序可用的 Node.js 版本的列表。始终确认你的项目所依赖的 Node.js 版本是否在此列表中。

## 对 Azure 网站使用自定义版本

虽然 Azure 提供了 Node.js 的几个默认版本，但你可能希望使用并非默认提供的版本。如果你的应用程序托管为 Azure 网站，则可以使用 **iisnode.yml** 文件完成此操作。以下步骤演练了对 Azure 网站使用自定义版本的 Node.Js 的过程：

1. 创建一个新目录，然后在该目录中创建 **server.js** 文件。**server.js** 文件应包含以下内容：

		var http = require('http');
		http.createServer(function(req,res) {
		  res.writeHead(200, {'Content-Type': 'text/html'});
		  res.end('Hello from Azure running node version: ' + process.version + '</br>');
		}).listen(process.env.PORT || 3000);

	这将显示在您浏览网站时使用的 Node.js 版本。

2. 创建一个新网站并记录该网站的名称。例如，以下命令使用 [Azure 命令行工具]创建一个名为 **mywebsite** 的新 Azure 网站，然后为该网站启用 Git 存储库。

		azure site create mywebsite --git

3. 创建一个名为 **bin** 的新目录，将其作为包含 **server.js** 文件的目录的子目录。

4. 下载要用于你的应用程序的特定版本的 **node.exe**（Windows 版本）。例如，以下命令使用 **curl** 下载版本 0.8.1：

		curl -O http://nodejs.org/dist/v0.8.1/node.exe

	将 **node.exe** 文件保存到先前创建的 **bin** 文件夹中。

5. 在 **server.js** 文件所在的目录中创建 **iisnode.yml** 文件，然后将以下内容添加到 **iisnode.yml** 文件中：

		nodeProcessCommandLine: "D:\home\site\wwwroot\bin\node.exe"

	将你的应用程序发布到 Azure 网站后，你的项目中的 **node.exe** 文件将位于此路径。

6. 发布应用程序。例如，由于我在前面创建了一个带 --git 参数的新网站，因此以下命令会将应用程序文件添加到我的本地 Git 存储库，然后将它们推送到网站存储库：

		git add .
		git commit -m "testing node v0.8.1"
		git push azure master

	发布应用程序后，在浏览器中打开该网站。您应看到一则指示“Hello from Azure running node version: v0.8.1”的消息。

## 后续步骤

了解如何指定应用程序使用的 Node.js 版本后，请学习[如何使用模块]、[生成和部署 Node.js 网站]以及[如何使用适用于 Mac 和 Linux 的 Azure 命令行工具]。

有关详细信息，请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

[如何使用适用于 Mac 和 Linux 的 Azure 命令行工具]: /documentation/articles/xplat-cli-install/
[Azure 命令行工具]: /documentation/articles/xplat-cli-install/
[如何使用模块]: /documentation/articles/nodejs-use-node-modules-azure-apps/
[生成和部署 Node.js 网站]: /documentation/articles/web-sites-nodejs-develop-deploy-mac/

<!---HONumber=Mooncake_0215_2016-->