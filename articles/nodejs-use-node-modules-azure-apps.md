<properties 
	pageTitle="使用 Node.js 模块"
	description="了解如何在使用 Azure 网站或云服务的同时使用 Node.js 模块。" 
	services="" 
	documentationCenter="nodejs" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="multiple" 
	ms.date="05/04/2016"
	wacn.date="06/20/2016"/>





# 将 Node.js 模块与 Azure 应用程序一起使用

本文档提供有关将 Node.js 模块与托管在 Azure 中的应用程序一起使用的指南。其中提供有关确保你的应用程序使用特定版本的模块，以及对 Azure 使用本机模块的指南。

如果您已了解如何使用 Node.js 模块、**package.json** 和 **npm-shrinkwrap.json** 文件，可参考本文中所讨论内容的以下快速摘要：

* Azure App Service 了解 **package.json** 和 **npm-shrinkwrap.json** 文件，可基于这些文件中的条目安装模块。
* Azure 云服务希望所有模块都安装在开发环境中，并将 **node\_modules** 目录包含为部署包的一部分。可以为使用 **package.json** 或 **npm-shrinkwrap.json** 文件在云服务上安装模块提供相应支持，但这需要自定义云服务项目使用的默认脚本。有关如何实现此目的的示例，请参阅[运行 npm 安装以避免部署 Node 模块的 Azure 启动任务](https://github.com/woloski/nodeonazure-blog/blob/master/articles/startup-task-to-run-npm-in-azure.markdown)

> [AZURE.NOTE] 本文不讨论 Azure 虚拟机，因为 VM 中的开发体验将取决于由虚拟机托管的操作系统。

##Node.js 模块

模块是可加载的 JavaScript 包，可为你的应用程序提供特定功能。通常使用 **npm** 命令行工具安装模块，但一些模块（如 http 模块）是作为核心 Node.js 包的一部分提供的。

在安装模块后，这些模块存储在您的应用程序目录结构根目录下的 **node\_modules** 目录中。**node\_modules** 目录中的每个模块都保留自己的 **node\_modules** 目录，其中包含它依赖的所有模块，依赖项链上的每个模块均是如此。这使得所安装的每个模块都对它所依赖的模块具有自己的版本要求，但这样会生成很大的目录结构。

与使用 **package.json** 或 **npm-shrinkwrap.json** 文件相比，在您的应用程序中部署 **node\_modules** 目录会增加部署的大小；但是，它确实可以保证在生产中使用的模块版本与在开发中使用的模块版本是相同的。

###本机模块

虽然多数模块都只是纯文本 JavaScript 文件，但一些模块是特定于平台的二进制映像。这些模块通常是在安装时使用 Python 和 node-gyp 编译的。由于 Azure 云服务依赖作为应用程序一部分部署的 **node\_modules** 文件夹，因此只要作为已安装模块一部分包含的任何本机模块是在 Windows 开发系统中安装和编译的，那么该模块都应在云服务中运行。

Azure 网站不支持所有本机模块，并且在编译那要求具有非常特定的系统组建的模块时可能会失败。一些常用模块（如 MongoDB）需要的本机依赖项是可选的，如果没有这些依赖项可勉强正常工作。对于当今几乎所有的可用本机模块而言，有两种成功的解决方法：

* 在安装了所有系统必备组件的 Windows 计算机运行 **npm install** 。然后，将创建的 **node\_modules** 文件夹作为 Azure 网站应用程序的一部分进行部署。
* 可以将 Azure 网站配置为在部署期间执行自定义 bash 或 shell 脚本，从而使您能够执行自定义命令和精确地配置运行 **npm install** 的方式。有关如何执行此操作的视频，请参阅 [使用 Kudu 自定义网站部署脚本]。

###使用 package.json 文件

可使用 **package.json** 文件来指定您的应用程序所需的顶级依赖项，以便托管平台能够安装这些依赖项，而不是要求您在部署中包含 **node\_packages** 文件夹。部署应用程序后，可使用 **npm install** 命令解析 **package.json** 文件并安装列出的所有依赖项。

在开发期间，您可以在安装模块时使用 **--save**、**--save-dev** 或 **--save-optional** 参数，以便自动将模块条目添加到 **package.json** 文件中。有关详细信息，请参阅 [npm-install](https://npmjs.org/doc/install.html)。

**package.json** 文件的一个潜在问题是它仅指定顶级依赖项的版本。安装的每个模块不一定会指定它所依赖的模块的版本，所以你最终使用的依赖项链可能与开发过程中使用的不同。

> [AZURE.NOTE]
部署到 Azure 网站时，如果 <b>package.json</b> 文件引用本机模块，那么在使用 Git 发布应用程序时您会看到如下错误：

>		npm ERR! module-name@0.6.0 install: 'node-gyp configure build'

>		npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1


###使用 npm-shrinkwrap.json 文件

**npm-shrinkwrap.json** 文件用于尝试消除 **package.json** 文件的模块版本控制限制。虽然 **package.json** 文件仅包含顶级模块的版本，但 **npm-shrinkwrap.json** 文件包含所有模块依赖项链的版本要求。

您的应用程序准备好生产后，便可锁定版本要求，并使用 **npm shrinkwrap** 命令创建 **npm-shrinkwrap.json** 文件。这将使用当前安装在 **node\_modules** 文件夹中的版本，并将这些信息记录到 **npm-shrinkwrap.json** 文件。将应用程序部署到托管环境后，可使用 **npm install** 命令来解析 **npm-shrinkwrap.json** 文件并安装列出的所有依赖项。有关详细信息，请参阅 [npm-install](https://npmjs.org/doc/install.html)。

> [AZURE.NOTE]
部署到 Azure 网站时，如果 <b>npm-shrinkwrap.json</b> 文件引用本机模块，那么在使用 Git 发布应用程序时，您会看到如下错误：

>		npm ERR! module-name@0.6.0 install: 'node-gyp configure build'

>		npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1


##后续步骤

了解如何将 Node.js 模块与 Azure 一起使用后，请学习如何[指定 Node.js 版本]、[生成和部署 Node.js 网站]，以及[如何使用适用于 Mac 和 Linux 的 Azure 命令行界面]。

有关详细信息，请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

[指定 Node.js 版本]: /documentation/articles/nodejs-specify-node-version-azure-apps/
[如何使用适用于 Mac 和 Linux 的 Azure 命令行界面]: /documentation/articles/xplat-cli-install/
[生成和部署 Node.js 网站]: /documentation/articles/web-sites-nodejs-develop-deploy-mac/
[Node.js Web Application with Storage on MongoDB (MongoLab)]: /documentation/articles/store-mongolab-web-sites-nodejs-store-data-mongodb/
[Publishing with Git]: /documentation/articles/web-sites-publish-source-control/
[Build and deploy a Node.js application to an Azure Cloud Service]: /documentation/articles/cloud-services-nodejs-develop-deploy-app/

<!---HONumber=Mooncake_0425_2016-->