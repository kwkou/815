<properties 
	pageTitle="使用 Socket.io 构建 Node.js 应用程序 | Azure" 
	description="了解如何在 Azure 上托管的 node.js 应用程序中使用 socket.io。" 
	services="cloud-services" 
	documentationCenter="nodejs" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="05/04/2016" 
	wacn.date="05/31/2016"/>

# 在 Azure 云服务中使用 Socket.IO 生成 Node.js 聊天应用程序

Socket.IO 在 node.js 服务器和客户端之间提供实时通信。本教程指导你如何在 Azure 上托管一个基于 socket.IO 的聊天应用程序。有关 Socket.IO 的详细信息，请参阅 <http://socket.io/>。

以下是已完成应用程序的屏幕快照：

![显示托管在 Azure 上的服务的浏览器窗口][completed-app]

## 先决条件

确保以下产品和版本已安装才能成功完成本文中的示例：

* 安装 [Visual Studio 2013](https://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs.aspx)
* 安装 [Node.js](https://nodejs.org/download)
* 安装 [Python 2.7.10 版](https://www.python.org)

## 创建云服务项目

以下步骤创建将托管 Socket.IO 应用程序的云服务项目。

1. 在“开始”菜单或“开始”屏幕中，搜索 **Windows PowerShell**。最后，右键单击“Windows PowerShell”并选择“以管理员身份运行”。

	![Azure PowerShell 图标][powershell-menu]

2. 创建一个名为 **c:\\node** 的目录。
 
		PS C:\> md node

3. 将目录更改为 **c:\\node** 目录
 
		PS C:\> cd node

4. 输入以下命令以创建一个名为 **chatapp** 的新解决方案和一个名为 **WorkerRole1** 的辅助角色：

		PS C:\node> New-AzureServiceProject chatapp
		PS C:\Node> Add-AzureNodeWorkerRole

	你将看到以下响应：

	![new-azureservice 和 add-azurenodeworkerrole cmdlet 的输出](./media/cloud-services-nodejs-chat-app-socketio/socketio-1.png)

## 下载聊天示例

对于此项目，我们将使用 [Socket.IO GitHub 存储库]中的聊天示例。执行以下步骤下载该示例并将其添加到您先前创建的项目中。

1.  使用“克隆”按钮创建存储库的本地副本。你还可以使用“ZIP”按钮下载项目。

    ![显示 https://github.com/LearnBoost/socket.io/tree/master/examples/chat 的浏览器窗口，其中 ZIP 下载图标突出显示][chat-example-view]

3.  浏览本地存储库的目录结构，找到 **examples\\chat** 目录。将该目录的内容复制到早先创建的 **C:\\node\\chatapp\\WorkerRole1** 目录。

    ![资源管理器，显示从存档中解压缩的 examples\\chat 目录的内容][chat-contents]

    上面的屏幕截图中突出显示的项目是从 **examples\\chat** 目录复制的文件

4.  在 **C:\\node\\chatapp\\WorkerRole1** 目录中，删除 **server.js** 文件，然后将 **app.js** 文件重命名为 **server.js**。这将删除前面由 **Add-AzureNodeWorkerRole** cmdlet 创建的默认 **server.js** 文件并用聊天示例中的应用程序文件取代。

### 修改 Server.js 并安装模块

在 Azure 模拟器中测试应用程序之前，必须进行一些小改动。对 server.js 文件执行以下步骤：

1.  在 Visual Studio 或任何文本编辑器中打开 **server.js** 文件。

2.  在 server.js 的开头处查找 **Module dependencies** 节，将包含 **sio = require('..//..//lib//socket.io')** 的行改为 **sio = require('socket.io')**，如下所示：

		var express = require('express')
  		, stylus = require('stylus')
  		, nib = require('nib')
		//, sio = require('..//..//lib//socket.io'); //Original
  		, sio = require('socket.io');                //Updated

3.  为了确保应用程序侦听正确端口，请在记事本或你喜爱的编辑器中打开 server.js，然后通过将 **3000** 替换为 **process.env.port** 来更改以下行，如下所示：

        //app.listen(3000, function () {            //Original
		app.listen(process.env.port, function () {  //Updated
		  var addr = app.address();
		  console.log('   app listening on http://' + addr.address + ':' + addr.port);
		});

保存对 **server.js** 所做更改后，使用以下步骤安装所需的模块，然后在 Azure 模拟器中测试应用程序：

1.  使用 **Azure PowerShell**，将目录切换到 **C:\\node\\chatapp\\WorkerRole1** 目录，然后使用以下命令安装此应用程序所需的模块：

        PS C:\node\chatapp\WorkerRole1> npm install

    这将安装 package.json 文件中所列的模块。命令完成后，您应该看到类似下面的输出：

    ![npm install 命令的输出][The-output-of-the-npm-install-command]

4.  因为此示例最初是 Socket.IO GitHub 存储库的一部分，并通过相对路径直接引用了 Socket.IO 库，而 package.json 文件中未引用 Socket.IO，所以我们必须通过发出以下命令来安装它：

        PS C:\node\chatapp\WorkerRole1> npm install socket.io --save

### 测试和部署

1.  通过发出以下命令启动模拟器：

        PS C:\node\chatapp\WorkerRole1> Start-AzureEmulator -Launch

2.  打开浏览器并导航到 **http://127.0.0.1**。

3.  当浏览器窗口打开时，输入一个昵称，然后按 Enter。这样，您就可以以特定昵称发布消息。要测试多用户功能，请使用同一 URL 打开另外的浏览器窗口但输入不同的昵称。

    ![显示来自用户 1 和用户 2 的聊天消息的两个浏览器窗口](./media/cloud-services-nodejs-chat-app-socketio/socketio-8.png)

3.  测试应用程序之后，发出以下命令停止仿真程序：

        PS C:\node\chatapp\WorkerRole1> Stop-AzureEmulator

4.  若要将应用程序部署到 Azure，请使用 **Publish-AzureServiceProject** cmdlet。例如：

        PS C:\node\chatapp\WorkerRole1> Publish-AzureServiceProject -ServiceName mychatapp -Location "China East" -Launch

	> [AZURE.IMPORTANT] 确保使用唯一名称，否则发布过程会失败。部署完成后，浏览器将打开并导航到已部署的服务。
	> 
	> 如果你收到一条错误，指出导入的发布配置文件中不存在提供的订阅名称，则你必须先为你的订阅下载和导入发布配置文件，然后再部署到 Azure。请参阅[生成 Node.js 应用程序并将其部署到 Azure 云服务](/documentation/articles/cloud-services-nodejs-develop-deploy-app/)中的**将应用程序部署到 Azure**部分

    ![显示托管在 Azure 上的服务的浏览器窗口][completed-app]

	> [AZURE.NOTE] 如果你收到一条错误，指出导入的发布配置文件中不存在提供的订阅名称，则你必须先为你的订阅下载和导入发布配置文件，然后再部署到 Azure。请参阅[生成 Node.js 应用程序并将其部署到 Azure 云服务](/documentation/articles/cloud-services-nodejs-develop-deploy-app/)中的**将应用程序部署到 Azure**部分

你的应用程序现在即可在 Azure 上运行，并可使用 Socket.IO 在不同客户端之间中继聊天消息。

> [AZURE.NOTE] 为简单起见，此示例仅限于连接到同一实例的用户之间的聊天。这意味着如果云服务创建两个辅助角色实例，用户将只能够与连接到同一辅助角色实例的其他用户聊天。要缩放应用程序以使用多个角色实例，你可以使用像 Service Bus 这样的技术在实例之间共享 Socket.IO 存储状态。有关示例，请参阅 [Azure SDK for Node.js GitHub 存储库的](https://github.com/WindowsAzure/azure-sdk-for-node)中的服务总线队列和主题使用示例。

##后续步骤

在本教程中，你已了解如何创建在 Azure 云服务中托管的基本聊天应用程序。若要了解如何在 Azure 网站中托管此应用程序，请参阅[在 Azure 网站中使用 Socket.IO 生成 Node.js 聊天应用程序][chatwebsite]。

有关详细信息，另请参阅 [Node.js 开发人员中心](/develop/nodejs)。

  [chatwebsite]: /documentation/articles/web-sites-nodejs-chat-app-socketio/

  [Azure SLA]: /support/legal/sla/
  [Azure SDK for Node.js GitHub repository]: https://github.com/WindowsAzure/azure-sdk-for-node
  [completed-app]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-10.png
  [Azure SDK for Node.js]: /develop/nodejs/
  [Node.js Web Application]: /documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [Socket.IO GitHub 存储库]: https://github.com/LearnBoost/socket.io/tree/0.9.14
  [Azure Considerations]: #windowsazureconsiderations
  [Hosting the Chat Example in a Worker Role]: #hostingthechatexampleinawebrole
  [Summary and Next Steps]: #summary
  [powershell-menu]: ./media/cloud-services-nodejs-chat-app-socketio/azure-powershell-start.png

  [chat example]: https://github.com/LearnBoost/socket.io/tree/master/examples/chat
  [chat-example-view]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-22.png
  
  
  [chat-contents]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-5.png
  [The-output-of-the-npm-install-command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-7.png
  [The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-9.png
  

<!---HONumber=Mooncake_0523_2016-->
