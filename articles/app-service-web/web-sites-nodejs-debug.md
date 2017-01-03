<properties
	pageTitle="如何在 Azure 应用服务中调试 Node.js Web 应用"
	description="了解如何在 Azure 应用服务中调试 Node.js Web 应用。"
	tags="azure-portal"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="article"
	ms.date="11/01/2016"
	wacn.date="12/30/2016"
	ms.author="robmcm"/>

# 如何在 Azure 应用服务中调试 Node.js Web 应用

Azure 提供了内置诊断来调试托管在 [Azure 应用服务](/documentation/articles/app-service-changes-existing-services/) Web 应用中的 Node.js 应用程序。在本文中，你将学习如何启用 stdout 和 stderr 的日志记录，如何在浏览器中显示错误信息以及下载和查看日志文件。

[IISNode] 提供对在 Azure 上托管的 Node.js 应用程序的诊断。本文只讨论了有关收集诊断信息的常见设置，并未提供使用 IISNode 的完整参考。有关使用 IISNode 的详细信息，请参阅 GitHub 上的 [IISNode 自述文件]。

<a id="enablelogging"></a>
## 启用日志记录

默认情况下，应用服务 Web 应用只捕获与部署有关的诊断信息，例如，当你使用 Git 部署 Web 应用时。如果在部署期间遇到问题（例如安装在 **package.json** 中引用的模块时失败），或者如果使用自定义部署脚本，此信息会很有用。

要启用 stdout 和 stderr 流的日志记录，必须在 Node.js 应用程序的根创建一个 **IISNode.yml** 文件，并添加以下内容：

	loggingEnabled: true

这将从 Node.js 应用程序启用 stderr 和 stdout 的日志记录。

**IISNode.yml** 文件还可用于控制在发生失败时是否将友好的错误或开发人员错误返回到浏览器。要启用开发人员错误，请将以下行添加到 **IISNode.yml** 文件中：

	devErrorsEnabled: true

启用该选项后，IISNode 将返回发送给 stderr 的最后 64K 的信息，而不是明确的错误提示，例如“发生了内部服务器错误”。

> [AZURE.NOTE] 尽管 devErrorsEnabled 对于开发期间诊断问题很有用，但是在生产环境中启用它可能会导致向最终用户发送开发错误。

如果 **IISNode.yml** 文件在应用程序内已不存在，则在发布更新的应用程序后必须重新启动 Web 应用。如果只在以前已发布的现有 **IISNode.yml** 文件中更改设置，则无需重新启动。

> [AZURE.NOTE] 如果使用 Azure 命令行工具或 Azure PowerShell Cmdlet 创建你的 Web 应用，则将自动创建一个默认的 **IISNode.yml** 文件。

若要重新启动 Web 应用，请在 [Azure 门户预览](https://portal.azure.cn)中选择该 Web 应用，然后单击“重新启动”按钮：

![重新启动按钮][restart-button]

如果在你的开发环境中已安装 Azure 命令行工具，你可以使用以下命令重新启动该 Web 应用：

	azure site restart [sitename]

> [AZURE.NOTE] 尽管 loggingEnabled 和 devErrorsEnabled 是用于捕获诊断信息的最常用的 IISNode.yml 配置选项，但可以使用 IISNode.yml 为托管环境配置多种选项。有关配置选项的完整列表，请参阅 [iisnode\_schema.xml](https://github.com/tjanczuk/iisnode/blob/master/src/config/iisnode_schema.xml) 文件。

<a id="viewlogs"></a>
## 访问日志

可通过三种方法访问诊断日志：使用文件传输协议 (FTP)、下载 ZIP 存档或作为日志的实时更新流（也称作尾标）。下载日志文件的 Zip 存档或者查看实时流需要使用 Azure 命令行工具。可使用以下命令安装它们：

	npm install azure-cli -g

安装后，可以使用“azure”命令访问这些工具。首先，必须配置这些命令行工具使用你的 Azure 订阅。有关如何实现此任务的信息，请参阅[如何使用 Azure 命令行工具](/documentation/articles/xplat-cli-connect/)一文的**如何下载和导入发布设置**部分。

###FTP

要通过 FTP 访问诊断信息，请访问 [Azure 门户预览](https://portal.azure.cn)，选择你的 Web 应用，然后选择“仪表板”。在“快速链接”部分中，“FTP 诊断日志”和“FTPS 诊断日志”链接提供了使用 FTP 协议访问日志的权限。

> [AZURE.NOTE] 如果你以前没有为 FTP 或部署配置用户名和密码，则可以通过选择“设置部署凭据”从“快速启动”管理页来执行此配置。

在仪表板中返回的 FTP URL 针对 **LogFiles** 目录，该目录中将包含以下子目录：

* [部署方法](/documentation/articles/web-sites-deploy/) - 如果你使用 Git 之类的部署方法，将创建同名的目录，并将包含与部署相关的信息。

* nodejs - 从你的所有应用程序实例捕获的 Stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

###Zip 存档

要下载诊断日志的 zip 存档，请使用 Azure 命令行工具中的以下命令：

	azure site log download [sitename]

这将在当前目录中下载 **diagnostics.zip**。此存档包含以下目录结构：

* 部署 - 与你的应用程序部署有关的信息日志

* LogFiles

	* [部署方法](/documentation/articles/web-sites-deploy/) - 如果你使用 Git 之类的部署方法，将创建同名的目录，并将包含与部署相关的信息。

	* nodejs - 从你的所有应用程序实例捕获的 Stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

###实时流（尾标）

要查看诊断日志信息的实时流，请使用 Azure 命令行工具中的以下命令：

	azure site log tail [sitename]

这将随着日志事件在服务器上发生返回更新的日志事件流。该流将返回部署信息以及 stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

<a id="nextsteps"></a>
## 后续步骤

在本文中，你学习了如何启用和访问 Azure 的诊断信息。尽管此信息对于了解你的应用程序发生的问题很有用，但它可能会指出与你正使用的模块有关的问题，或者应用服务 Web Apps 所用的 Node.js 版本与在你的部署环境中使用的版本不同。

有关使用 Azure 上的模块的信息，请参阅[将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)。

有关为你的应用程序指定 Node.js 版本的信息，请参阅[在 Azure 应用程序中指定 Node.js 版本]。

有关详细信息，另请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

[IISNode]: https://github.com/tjanczuk/iisnode
[IISNode 自述文件]: https://github.com/tjanczuk/iisnode#readme
[How to Use The Azure Command-Line Interface]: /documentation/articles/xplat-cli-install/
[Using Node.js Modules with Azure Applications]: /documentation/articles/nodejs-use-node-modules-azure-apps/
[在 Azure 应用程序中指定 Node.js 版本]: /documentation/articles/nodejs-specify-node-version-azure-apps/
[restart-button]: ./media/web-sites-nodejs-debug/restartbutton.png
 

<!---HONumber=Mooncake_Quality_Review_1118_2016-->