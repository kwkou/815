<properties
	pageTitle="如何在 Azure 中调试 Node.js Web 应用"
	description="了解如何在 Azure 中调试 Node.js Web 应用。"
	tags="azure-portal"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>

# 如何在 Azure 中调试 Node.js Web 应用

Azure 提供内置的诊断以帮助调试托管在 [Azure Web 应用](/documentation/services/web-sites/) Web Apps 中的 Node.js 应用程序。在本文中，您将学习如何启用 stdout 和 stderr 的日志记录、在浏览器中显示错误信息以及如何下载和查看日志文件。

[IISNode] 提供对在 Azure 上托管的 Node.js 应用程序的诊断。尽管本文讨论的是用于收集诊断信息的最常见设置，但并没有完整介绍如何使用 IISNode。有关使用 IISNode 的详细信息，请参阅 GitHub 上的 [IISNode 自述文件]。

<a id="enablelogging"></a>
## 启用日志记录

默认情况下，Azure Web 应用只捕获与部署有关的诊断信息，例如，当你使用 Git 部署 Web 应用时。如果你在部署期间具有问题，例如安装在 **package.json** 中引用的模块时失败，或者如果你在使用自定义部署脚本，则该信息将很有用。

若要启用 stdout 和 stderr 流的日志记录，你必须在你的 Node.js 应用程序的根创建一个 **IISNode.yml** 文件并且添加以下内容：

	loggingEnabled: true

这将从 Node.js 应用程序启用 stderr 和 stdout 的日志记录。

该 **IISNode.yml** 文件还可用于控制在发生失败时是否将友好的错误或开发人员错误返回到浏览器。若要启用开发人员错误，请将以下行添加到该 **IISNode.yml** 文件：

	devErrorsEnabled: true

在启用该选项后，IISNode 将返回发送给 stderr 的最后 64K 的信息，而不是友好错误，例如“发生了内部服务器错误”。

> [AZURE.NOTE] 尽管在开发期间诊断问题时 devErrorsEnabled 将很有用，但在生产环境中启用它可能会导致发送给最终用户的开发错误。

如果该 **IISNode.yml** 文件在你的应用程序内已不存在，则在发布更新的应用程序后必须重新启动你的 Web 应用。如果你只是在以前已发布的现有 **IISNode.yml** 文件中更改设置，则无需重新启动。

> [AZURE.NOTE] 如果你的 Web 应用是使用 Azure 命令行工具或 Azure PowerShell Cmdlet 创建的，将自动创建一个默认的 **IISNode.yml** 文件。

若要重新启动 Web 应用，请在 [Azure 经典管理门户](https://manage.windowsazure.cn)中选择该 Web 应用，然后单击“重新启动”按钮：

![重新启动按钮][restart-button]

如果 Azure 命令行工具安装在你的开发环境中，你可以使用以下命令重新启动该 Web 应用：

	azure site restart [sitename]

> [AZURE.NOTE] 尽管 loggingEnabled 和 devErrorsEnabled 是用于配置诊断信息的最常用 IISNode.yml 配置选项，但可以使用 IISNode.yml 为您的托管环境配置多种选项。有关配置选项的完整列表，请参阅 [iisnode\_schema.xml](https://github.com/tjanczuk/iisnode/blob/master/src/config/iisnode_schema.xml) 文件。

<a id="viewlogs"></a>
## 访问日志

可通过三种方法访问诊断日志：使用文件传输协议 (FTP)、下载 ZIP 存档或作为日志的实时更新流（也称作尾标）。下载日志文件的 Zip 存档或者查看实时流要求 Azure 命令行工具。可使用以下命令安装它们：

	npm install azure-cli -g

安装后，可以使用“azure”命令访问这些工具。必须首先将这些命令行工具配置为使用你的 Azure 订阅。有关如何实现此任务的信息，请参阅[如何使用Azure 命令行工具](/documentation/articles/xplat-cli-connect/)一文的 **如何下载和导入发布设置** 部分。

###FTP

若要通过 FTP 访问诊断信息，请访问 [Azure 经典管理门户](https://manage.windowsazure.cn)，选择你的 Web 应用，然后选择“仪表板”。在“快速链接”部分中，“FTP 诊断日志”和“FTPS 诊断日志”链接提供使用 FTP 协议对日志的访问。

> [AZURE.NOTE] 如果你以前没有为 FTP 或部署配置用户名和密码，则可以通过选择“设置部署凭据”从“快速启动”管理页来执行此配置。

在仪表板中返回的 FTP URL 针对 **LogFiles** 目录，该目录中将包含以下子目录：

* [部署方法](/documentation/articles/web-sites-deploy/) - 如果你使用 Git 之类的部署方法，将创建同名的目录并且将包含与部署相关的信息。

* nodejs - 从你的所有应用程序实例捕获的 Stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

###Zip 存档

若要下载诊断日志的 zip 存档，请使用 Azure 命令行工具中的以下命令：

	azure site log download [sitename]

这将在当前目录中下载 **diagnostics.zip**。此存档包含以下目录结构：

* 部署 - 与你的应用程序部署有关的信息日志

* LogFiles

	* [部署方法](/documentation/articles/web-sites-deploy/) - 如果你使用 Git 之类的部署方法，将创建同名的目录并且将包含与部署相关的信息。

	* nodejs - 从你的所有应用程序实例捕获的 Stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

###实时流（尾标）

若要查看诊断日志信息的实时流，请使用 Azure 命令行工具中的以下命令：

	azure site log tail [sitename]

这将随着日志事件在服务器上发生返回更新的日志事件流。该流将返回部署信息以及 stdout 和 stderr 信息（在 loggingEnabled 为 true 时）。

<a id="nextsteps"></a>
## 后续步骤

在本文中，你学习了如何启用和访问 Azure 的诊断信息。尽管此信息对于理解你的应用程序发生的问题很有用，但它可能会指出与你正使用的模块有关的问题或者 Azure Web Apps 所用的 Node.js 版本不同于在你的部署环境中使用的版本。

有关使用 Azure 上的模块的信息，请参阅[将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)。

有关为你的应用程序指定 Node.js 版本的信息，请参阅[在 Azure 应用程序中指定 Node.js 版本]。

有关详细信息，另请参阅 [Node.js 开发人员中心](/develop/nodejs/)。

[IISNode]: https://github.com/tjanczuk/iisnode
[IISNode 自述文件]: https://github.com/tjanczuk/iisnode#readme
[How to Use The Azure Command-Line Interface]: /documentation/articles/xplat-cli-install/
[Using Node.js Modules with Azure Applications]: /documentation/articles/nodejs-use-node-modules-azure-apps/
[在 Azure 应用程序中指定 Node.js 版本]: /documentation/articles/nodejs-specify-node-version-azure-apps/
[restart-button]: ./media/web-sites-nodejs-debug/restartbutton.png
 

<!---HONumber=Mooncake_0307_2016-->