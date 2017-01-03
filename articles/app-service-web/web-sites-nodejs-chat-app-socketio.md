<properties
	pageTitle="在 Azure 应用服务中使用 Socket.IO 创建 Node.js 聊天应用程序"
	description="此教程演示如何在托管于 Azure 上的 node.js Web 应用中使用 socket.io。"
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

# 在 Azure 应用服务中使用 Socket.IO 创建 Node.js 聊天应用程序

Socket.IO 使用 WebSocket 在 node.js 服务器和客户端之间提供实时通信。还支持回退到使用低版本浏览器的其他传输（如长轮询）。本教程将演示如何以 Azure Web 应用的形式托管基于 Socket.IO 的聊天应用程序，并说明如何通过 [Azure Redis 缓存]缩放应用程序。有关 Socket.IO 的详细信息，请参阅 <http://socket.io/>。

> [AZURE.NOTE] 此任务中的过程适用于[应用服务 Web 应用]；对于云服务，请参阅 [Build a Node.js Chat Application with Socket.IO on an Azure Cloud Service]（在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序）。

## 下载聊天示例

对于此项目，将使用 [Socket.IO GitHub 存储库]中的聊天示例。执行以下步骤下载该示例，并将其添加到先前创建的项目中。

1.  下载 Socket.IO 项目的 [ZIP 或 GZ 存档版本]（本文档使用版本 1.3.5）

1.  解压缩存档，并将 **examples\\chat** 目录复制到新位置。例如，**\\node\\chat**。

## 修改 App.js 并安装模块

1.  将 **index.js** 文件重命名为 **app.js**。这可支持 Azure 检测它是否为 Node.js 应用程序。

1.  在文本编辑器中打开 **app.js** 文件。更改包含 `var io = require('../..')(server);` 的行，如下所示：

		var express = require('express');
		var app = express();
		var server = require('http').createServer(app);
		// var io = require('../..')(server);
        // New:
		var io = require('socket.io')(server);
		var port = process.env.PORT || 3000;

1. 打开 **package.json** 文件，并在 `dependencies` 下面添加对 socket.io 的引用，如下所示：

        "dependencies": {
		  "express": "3.4.8",
		  "socket.io": "1.3.5"
		}

1. 从命令行中，切换到 **\\node\\chat** 目录，然后使用 npm 安装此应用程序所需的模块：

        npm install

    将模块安装到名为 **node\_modules** 的子文件夹。

## 创建 Azure Web 应用

按照以下步骤创建 Azure Web 应用，启用 Git 发布，然后为 Web 应用启用 WebSocket 支持。

> [AZURE.NOTE] 若要完成本教程，需要一个 Azure 帐户。如果没有帐户，可以在几分钟内创建一个试用帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=A7171371E" target="_blank">Azure 试用版</a>。

1. 安装 Azure 命令行接口 (Azure CLI)，并连接到 Azure 订阅。请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。

1. 如果首次在 Azure 中设置存储库，需要创建登录凭据。从 Azure CLI 输入以下命令：

		azure site deployment user set [username] [password]

1. 切换到 **\\node\\chat** 目录，然后使用以下命令创建新的 Azure Web 应用和本地 Git 存储库。此命令还会创建名为“azure”的 Git 远程连接。

		azure site create mysitename --git

	必须将“mysitename”替换为 Web 应用的唯一名称。

1. 使用以下命令将现有文件提交到本地存储库：

		git add .
		git commit -m "Initial commit"

1. 使用以下命令将这些文件推送到 Azure Web Apps 存储库：

		git push azure master

	系统出现提示时，请输入步骤 2 中的凭据。在服务器上导入模块时会收到状态消息。此过程完成后，应用程序将托管在 Azure Web 应用中。

 	> [AZURE.NOTE] 在模块安装过程中，可能会出现“找不到导入的项目...”错误。可以安全地忽略这些错误。

1. 在 Azure 上默认不启用 Socket.IO 使用的 WebSocket。若要启用 Web 套接字，请使用以下命令：

		azure site set -w

	如果系统提示，请输入 Web 应用的名称。

	>[AZURE.NOTE]
	“azure site set -w”命令仅适用于 Azure 命令行接口 0.7.4 版或更高版本。也可以使用 [Azure 门户预览](https://portal.azure.cn)启用 WebSocket 支持。<p>若要使用 Azure 门户预览启用 WebSocket，请在“Web 应用”边栏选项卡中单击该 Web 应用，然后单击“所有设置”>“应用程序设置”。在“Web 套接字”下，单击“打开”。然后，单击“保存”。

1. 要查看 Azure 上的 Web 应用，请使用以下命令启动 Web 浏览器，并导航到托管的 Web 应用：

		azure site browse

现在，应用在 Azure 上运行，可使用 Socket.IO 在不同客户端之间中继聊天消息。

## <a name="scale-out"></a>横向扩展

Socket.IO 应用程序可通过__适配器__实现向外扩展，以在多个应用程序实例之间发布消息和事件。可使用多个适配器，且可将 [socket.io redis] 适配器与 Azure Redis 缓存功能结合使用。

> [AZURE.NOTE] 向外扩展 Socket.IO 解决方案还要求支持粘滞会话。默认情况下，可通过 Azure 请求路由为 Azure Web 应用启用粘滞会话。有关详细信息，请参阅 [Instance Affinity in Azure Web Sites]（Azure 网站中的实例关联）。

### 创建 Redis 缓存

执行[在 Azure Redis 缓存中创建缓存]中的步骤，创建新缓存。

> [AZURE.NOTE] 保存缓存的__主机名__和__主密钥__，在后续步骤中需要用到。

### 添加 redis 和 socket.io redis 模块

1. 在命令行中，切换到 __\\node\\cha__ 目录，然后运行以下命令：

		npm install socket.io-redis@0.1.4 redis@0.12.1 --save

	> [AZURE.NOTE] 此命令中指定的版本是测试本文时使用的版本。

1. 修改 __app.js__ 文件，紧接在 `var io = require('socket.io')(server);` 后面添加以下行

		var pub = require('redis').createClient(6379,'redishostname', {auth_pass: 'rediskey', return_buffers: true});
		var sub = require('redis').createClient(6379,'redishostname', {auth_pass: 'rediskey', return_buffers: true});

		var redis = require('socket.io-redis');
		io.adapter(redis({pubClient: pub, subClient: sub}));

	使用 Redis 缓存的主机名和密钥替换 __redishostname__ 和 __rediskey__。

	这将创建之前创建的 Redis 缓存的发布和订阅客户端。然后，结合使用客户端和适配器，配置 Socket.IO 使用 Redis 缓存在应用程序实例之间传递消息和事件

	> [AZURE.NOTE] 尽管 __socket.io redis__ 适配器能够与 Redis 直接通信，但当前版本不支持 Azure Redis 缓存所需的身份验证。因此，使用 __redis__ 模块创建初始连接，然后将客户端传递到 __socket.io redis__ 适配器。<p>尽管 Azure Redis 缓存支持使用端口 6380 进行安全连接，但本示例中使用的模块不支持自 2014 年 7 月 14 日起的安全连接。上述代码使用默认的 6379 非安全端口。

1. 保存修改的 __app.js__

### 提交更改并重新部署

从 __\\node\\chat__ 目录的命令行中，使用以下命令提交更改，并重新部署该应用程序。

	git add .
	git commit -m "implementing scale out"
	git push azure master

将所做的更改推送到服务器后，可使用以下命令跨多个实例缩放网站。

	azure site scale instances --instances #

其中 __#__ 是要创建的实例数。

可从多个浏览器或计算机连接到 Web 应用，验证是否已正确将消息发送到所有客户端。

## 故障排除

### 连接限制

Azure Web 应用可提供多个 SKU，用于确定站点可用的资源。包括允许的 WebSocket 连接数。有关详细信息，请参阅 [Web Apps 定价页]。

### 不使用 WebSocket 发送消息

如果客户端浏览器保持回退到长轮询而不是使用 Websocket，可能有以下几种原因。

* **试图限制仅传输到 Websocket**

	为了使 Socket.IO 使用 Websocket 进行消息传输，服务器和客户端必须支持 Websocket。如果其中任一个不支持，则 Socket.IO 将协商其他传输，如长轮询。Socket.IO 使用的默认传输列表为 ` websocket, htmlfile, xhr-polling, jsonp-polling`。可将以下代码添加到 **app.js** 文件中包含 `, nicknames = {};` 的行后面，强制它仅使用 WebSocket。

		io.configure(function() {
		  io.set('transports', ['websocket']);
		});

	> [AZURE.NOTE] 注意，上述代码为活动状态时，不支持 Websocket 的低版本浏览器将无法连接到站点，因为代码将通信限制为仅使用 Websocket。

* **使用 SSL**

	Websocket 依赖某些较少使用的 HTTP 标头，如 **Upgrade** 标头。某些中间网络设备（例如 Web 代理）可能会删除这些标头。为避免发生此问题，可以建立基于 SSL 的 WebSocket 连接。

	实现此目的的简单方法是将 Socket.IO 配置到 `match origin protocol`。这会指示 Socket.IO 保护 Websocket 通信，与保护网页的原始 HTTP/HTTPS 请求一样。如果浏览器使用 HTTPS URL 访问网站，将基于 SSL 保护通过 Socket.IO 的后续 WebSocket 通信。

	要将此示例修改为启用此配置，请在 **app.js** 文件中包含 `, nicknames = {};` 的行后面添加以下代码。

		io.configure(function() {
		  io.set('match origin protocol', true);
		});

* **验证 web.config 设置**

	托管 Node.js 应用程序的 Azure Web 应用使用 **web.config** 文件，将传入请求路由到 Node.js 应用程序。为了使 Websocket 能够正常使用 Node.js 应用程序，**web.config** 必须包含以下条目。

		<webSocket enabled="false"/>

	这将禁用 IIS Websocket 模块，包括自身的 Websocket 实施和与 Node.js 特定 WebSocket 模块（如 Socket.IO）的冲突。如果此行不存在，或者已设置为 `true`，原因是 WebSocket 传输不适用于应用程序。

	Node.js 应用程序通常不包括 **web.config** 文件，因此 Azure 网站在部署 Node.js 应用程序时会自动生成 web.config 文件。由于此文件是在服务器上自动生成，因此必须使用网站的 FTP 或 FTPS URL 查看此文件。可通过选择 Web 应用，然后选择“仪表板”链接，在经典管理门户中查找站点的 FTP 和 FTPS URL。URL 显示在“速览”部分。

	> [AZURE.NOTE] 如果应用程序未提供 **web.config** 文件，则该文件将仅由 Azure 网站生成。如果在应用程序项目的根目录下提供了 **web.config** 文件，则 Azure Web 应用将使用该文件。

	如果该条目不存在，或者已设置为值 `true`，则应在 Node.js 应用程序的根目录中创建 **web.config** 并指定值 `false`。例如，使用 **app.js** 作为入口点的应用程序的默认 **web.config** 如下所示。

		<?xml version="1.0" encoding="utf-8"?>
		<!--
		     This configuration file is required if iisnode is used to run node processes behind
		     IIS or IIS Express.  For more information, visit:

		     https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config
		-->

		<configuration>
		  <system.webServer>
		    <!-- Visit http://blogs.msdn.com/b/windowsazure/archive/2013/11/14/introduction-to-websockets-on-windows-azure-web-sites.aspx for more information on WebSocket support -->
		    <webSocket enabled="false" />
		    <handlers>
		      <!-- Indicates that the server.js file is a node.js web app to be handled by the iisnode module -->
		      <add name="iisnode" path="app.js" verb="*" modules="iisnode"/>
		    </handlers>
		    <rewrite>
		      <rules>
		        <!-- Do not interfere with requests for node-inspector debugging -->
		        <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">
		          <match url="^app.js\/debug[\/]?" />
		        </rule>

		        <!-- First we consider whether the incoming URL matches a physical file in the /public folder -->
		        <rule name="StaticContent">
		          <action type="Rewrite" url="public{REQUEST_URI}"/>
		        </rule>

		        <!-- All other URLs are mapped to the node.js web app entry point -->
		        <rule name="DynamicContent">
		          <conditions>
		            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="True"/>
		          </conditions>
		          <action type="Rewrite" url="app.js"/>
		        </rule>
		      </rules>
		    </rewrite>
		    <!--
		      You can control how Node is hosted within IIS using the following options:
		        * watchedFiles: semi-colon separated list of files that will be watched for changes to restart the server
		        * node_env: will be propagated to node as NODE_ENV environment variable
		        * debuggingEnabled - controls whether the built-in debugger is enabled

		      See https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config for a full list of options
		    -->
		    <!--<iisnode watchedFiles="web.config;*.js"/>-->
		  </system.webServer>
		</configuration>

	如果应用程序使用入口点而不是 **app.js**，必须将出现的所有 **app.js** 替换为正确的入口点。例如，将 **app.js** 替换为 **server.js**。

## 后续步骤

在本教程中，已学习如何创建托管在 Azure Web 应用中的聊天应用程序。还可以将该应用程序作为 Azure 云服务托管。有关如何实现此目的的步骤，请参阅[在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序]。

有关详细信息，另请参阅 [Node.js 开发人员中心]。

## 更改内容

* 有关从网站更改为应用服务的指南，请参阅 [Azure App Service and Its Impact on Existing Azure Services]（Azure 应用服务及其对现有 Azure 服务的影响）。

<!-- URL List -->

[Azure Redis 缓存]: /documentation/services/redis-cache/
[应用服务 Web 应用]: /documentation/articles/app-service-changes-existing-services/
[Web Apps 定价页]: /documentation/articles/azure-subscription-service-limits/
[Build a Node.js Chat Application with Socket.IO on an Azure Cloud Service]: /documentation/articles/cloud-services-nodejs-chat-app-socketio/
[在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序]: /documentation/articles/cloud-services-nodejs-chat-app-socketio/
[Install and Configure the Azure CLI]: /documentation/articles/xplat-cli-install/
[Azure App Service and Its Impact on Existing Azure Services]: /documentation/articles/app-service-changes-existing-services/
[Node.js 开发人员中心]: /develop/nodejs/
[Instance Affinity in Azure Web Sites]: https://azure.microsoft.com/blog/2013/11/18/disabling-arrs-instance-affinity-in-windows-azure-web-sites/
[在 Azure Redis 缓存中创建缓存]: /documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/

[socket.io redis]: https://github.com/socketio/socket.io-redis
[Socket.IO GitHub 存储库]: https://github.com/socketio/socket.io
[ZIP 或 GZ 存档版本]: https://github.com/socketio/socket.io/releases

<!-- IMG List -->

[chat-example-view]: ./media/web-sites-nodejs-chat-app-socketio/socketio-2.png
[npm-output]: ./media/web-sites-nodejs-chat-app-socketio/socketio-7.png
[completed-app]: ./media/web-sites-nodejs-chat-app-socketio/websitesocketcomplete.png

<!---HONumber=Mooncake_Quality_Review_1118_2016-->