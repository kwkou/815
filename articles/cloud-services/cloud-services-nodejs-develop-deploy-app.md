<properties
	pageTitle="Node.js 入门指南 | Azure"
	description="了解如何创建简单的 Node.js Web 应用程序并将其部署到 Azure 云服务。"
	services="cloud-services"
	documentationCenter="nodejs"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>


<tags
	ms.service="cloud-services"
	ms.workload="tbd"
	ms.tgt_pltfrm="na" 
	ms.devlang="nodejs"
	ms.topic="hero-article"
	ms.date="11/01/2016" 
	wacn.date="01/03/2017" 
	ms.author="robmcm"/>

# 生成 Node.js 应用程序并将其部署到 Azure 云服务

本教程演示如何创建在 Azure 云服务中运行的简单 Node.js 应用程序。云服务是 Azure 中可缩放云应用程序的构建基块。可分隔并独立管理云服务，并可扩展应用程序的前端和后端组件。云服务为可靠托管每个角色提供强大的专用虚拟机。

有关云服务以及如何将它们与 Azure 网站和虚拟机进行比较的更多信息，请参阅 [Azure 网站、云服务和虚拟机的比较]。

>[AZURE.TIP] 想要构建简单网站？ 如果方案仅涉及简单网站前端，请考虑[使用轻型 Web 应用]。可随 Web 应用的增长和需求的变化轻松升级到云服务。

按照本教程，生成托管在 Web 角色中的简单 Web 应用程序。使用计算模拟器在本地测试应用程序，然后使用 PowerShell 命令行工具部署该应用程序。

该应用程序为简单的“hello world”应用程序：

![Web 浏览器中显示“Hello World”网页][A web browser displaying the Hello World web page]

## 先决条件

> [AZURE.NOTE] 本教程使用 Azure PowerShell，因此要求使用 Windows 系统。

- 安装和配置 [Azure PowerShell]。
- 下载并安装用于 .NET 2.7 的 Azure SDK。在安装设置中，选择：
    - MicrosoftAzureAuthoringTools
    - MicrosoftAzureComputeEmulator


## 创建 Azure 云服务项目

执行以下任务，创建新的 Azure 云服务项目及基本 Node.js 基架：

1. 以管理员身份运行 **Windows PowerShell**；在“开始”菜单或“开始”屏幕中，搜索 **Windows PowerShell**。

2. [将 PowerShell 连接]到订阅。

3. 输入以下 PowerShell cmdlet 来创建项目：

        New-AzureServiceProject helloworld

	![New-AzureService helloworld 命令的结果][The result of the New-AzureService helloworld command]

	**New-AzureServiceProject** cmdlet 将生成一个基本结构用于将 Node.js 应用程序发布到云服务。该结构包含向 Azure 发布应用程序所需的配置文件。该 cmdlet 还会将工作目录更改为服务的目录。

	该 cmdlet 将创建以下文件：

	-   **ServiceConfiguration.Cloud.cscfg**、**ServiceConfiguration.Local.cscfg** 和 **ServiceDefinition.csdef**：发布应用程序所需的特定于 Azure 的文件。有关详细信息，请参阅[创建 Azure 托管服务概述]。

	-   **deploymentSettings.json**：存储供 Azure PowerShell 部署 cmdlet 使用的本地设置。

4.  输入以下命令添加新的 Web 角色：

        Add-AzureNodeWebRole

	![Add-AzureNodeWebRole 命令的输出][The output of the Add-AzureNodeWebRole command]

	**Add-azurenodewebrole** cmdlet 将创建一个基本 Node.js 应用程序。它还会修改 **.csfg** 和 **.csdef** 文件，以添加新角色的配置条目。

	> [AZURE.NOTE] 如果不指定角色名称，将使用默认名称。可以提供一个名称作为第一个 cmdlet 参数：`Add-AzureNodeWebRole MyRole`

Node.js 应用在 **server.js** 文件中定义，该文件位于 Web 角色（默认为 **WebRole1**）的目录中。代码如下：

	var http = require('http');
	var port = process.env.port || 1337;
	http.createServer(function (req, res) {
	    res.writeHead(200, { 'Content-Type': 'text/plain' });
	    res.end('Hello World\n');
	}).listen(port);

此代码基本上与 nodejs.org 网站上的“Hello World”示例相同，但它使用云环境分配的端口号。

## 将应用程序部署到 Azure

> [AZURE.NOTE]
要完成本教程，需要一个 Azure 帐户。[注册以获取免费帐户](/pricing/1rmb-trial/)。

### 下载 Azure 发布设置

若要将应用程序部署到 Azure，必须先为 Azure 订阅下载发布设置。

1.  运行以下 Azure Powershell cmdlet：

        Get-AzurePublishSettingsFile -Environment AzureChinaCloud

	此操作将使用浏览器导航到发布设置下载页。可能会提示使用 Microsoft 帐户登录。如果出现提示，请使用与 Azure 订阅关联的帐户。

	将已下载的配置文件保存到可轻松访问的文件位置。

2.  运行以下 cmdlet 以导入下载的发布配置文件：

        Import-AzurePublishSettingsFile -Environment AzureChinaCloud [path to file]


	> [AZURE.NOTE] 导入发布设置之后，请考虑删除下载的 .publishSettings 文件，因为其他人可能利用其中信息访问你的帐户。

### 发布应用程序

若要发布，请运行以下命令：

  	$ServiceName = "NodeHelloWorld" + $(Get-Date -Format ('ddhhmm'))   
	Publish-AzureServiceProject -ServiceName $ServiceName  -Location "China East" -Launch

- **-ServiceName** 指定部署的名称。此名称必须唯一，否则发布过程将会失败。**Get-Date** 命令附加应使名称唯一的日期/时间字符串。

- **-Location** 指定托管应用程序的数据中心。若要查看可用数据中心的列表，请使用 **Get-AzureLocation** cmdlet。

- **-Launch** 用于在部署完成后打开浏览器窗口并导航到托管服务。

发布成功之后，可看到如下响应：

![Publish-AzureService 命令的输出][The output of the Publish-AzureService command]

> [AZURE.NOTE]
首次发布后应用程序后，它可能需要几分钟才能部署完成并可供使用。

部署完成后，系统将打开浏览器窗口并导航到云服务。

![显示“hello world”页面的浏览器窗口；URL 指示该页面托管在 Azure 上。][A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]

应用程序现在在 Azure 上运行。

**Publish-AzureServiceProject** cmdlet 执行以下步骤：

1.  创建包供部署。该包将包含应用程序文件夹中的所有文件。

2.  如果存储帐户不存在，将创建一个新的**存储帐户**。Azure 存储帐户在部署期间用于存储应用程序包。部署完成后，可以安全删除该存储帐户。

3.  如果云服务尚不存在，将创建一个新的**云服务**。**云服务**是一个容器，用于在将应用程序部署到 Azure 后托管该应用程序。有关详细信息，请参阅[创建 Azure 托管服务概述]。

4.  将部署包发布到 Azure。


## 停止并删除应用程序

部署应用程序后，可能需要禁用它以免产生额外费用。Azure 按使用的服务器小时数对 Web 角色实例进行计费。应用程序部署后即开始使用服务器时间，即使相关实例未运行并处于停止状态。

1.  在 Windows PowerShell 窗口中，使用以下 cmdlet 以停止上一节中创建的服务部署：

        Stop-AzureService

	停止服务可能需要花费几分钟时间。服务停止后，将收到消息表明已停止服务。

	![Stop-AzureService 命令的状态][The status of the Stop-AzureService command]

2.  若要删除服务，请调用以下 cmdlet：

        Remove-AzureService

	在出现提示时，输入 **Y** 以删除服务。

	删除服务可能需要花费几分钟时间。删除服务后，将收到消息表明已删除服务。

	![Remove-AzureService 命令的状态][The status of the Remove-AzureService command]

	> [AZURE.NOTE] 删除服务不会删除最初发布服务时创建的存储帐户，用户仍需为使用的存储付费。如果没有任何其他对象使用存储，请删除它。

## 后续步骤

有关详细信息，请参阅 [Node.js 开发人员中心]。

<!-- URL List -->

[Azure 网站、云服务和虚拟机的比较]: /documentation/articles/choose-web-site-cloud-service-vm/
[使用轻型 Web 应用]:/documentation/articles/web-sites-nodejs-develop-deploy-mac/
[Azure Powershell]: /documentation/articles/powershell-install-configure/
[将 PowerShell 连接]: /documentation/articles/powershell-install-configure/#Connect
[nodejs.org]: http://nodejs.org/
[创建 Azure 托管服务概述]: /documentation/services/cloud-services/
[Node.js 开发人员中心]: /develop/nodejs/

<!-- IMG List -->


[The result of the New-AzureService helloworld command]: ./media/cloud-services-nodejs-develop-deploy-app/node9.png
[The output of the Add-AzureNodeWebRole command]: ./media/cloud-services-nodejs-develop-deploy-app/node11.png
[A web browser displaying the Hello World web page]: ./media/cloud-services-nodejs-develop-deploy-app/node14.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node19.png
[A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]: ./media/cloud-services-nodejs-develop-deploy-app/node21.png
[The status of the Stop-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node48.png
[The status of the Remove-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node49.png

<!---HONumber=Mooncake_1226_2016-->