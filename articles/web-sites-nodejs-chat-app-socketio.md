<properties
	pageTitle="在 Azure 中使用 Socket.IO 创建 Node.js 聊天应用程序"
	description="此教程演示如何在托管于 Azure 上的 node.js Web 应用中使用 socket.io。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="MikeWasson"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service-web"
	ms.date="04/29/2016"
	wacn.date="06/29/2016"/>




# 在 Azure 中使用 Socket.IO 创建 Node.js 聊天应用程序

Socket.IO 使用 WebSocket 在 node.js 服务器和客户端之间提供实时通信。还支持回退到使用低版本浏览器的其他传输（如长轮询）。本教程将演示如何以 Azure Web 应用的的形式托管基于 Socket.IO 的聊天应用程序，并说明如何通过 [Azure Redis 缓存](/documentation/services/redis-cache)来[向外扩展](#scale-out)应用程序。有关 Socket.IO 的详细信息，请参阅 [http://socket.io/][socketio]。

> [AZURE.NOTE]此任务中的过程适用于 [Azure Web 应用](/documentation/services/web-sites/)；对于云服务，请参阅<a href="/documentation/articles/cloud-services-nodejs-chat-app-socketio/">在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序</a>。


## 下载聊天示例

对于此项目，我们将使用 [Socket.IO GitHub 存储库]中的聊天示例。执行以下步骤下载该示例并将其添加到您先前创建的项目中。

1.  下载 Socket.IO 项目的 [ZIP 或 GZ 存档版本][release]（本文档使用版本 1.3.5）


3.  解压缩存档并将 **examples\\chat** 目录复制到新位置。例如，**\\node\\chat**。

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


3. 打开 **package.json** 文件，并在 `dependencies` 下面添加对 socket.io 的引用，如下所示：

        "dependencies": {
		  "express": "3.4.8",
		  "socket.io": "1.3.5"
		}

4. 从命令行中，切换到 **\\node\\chat** 目录，然后使用 npm 安装此应用程序所需的模块：

        npm install

    这会将模块安装到名为 **node_modules** 的子文件夹。

## 创建 Azure Web 应用

按照以下步骤创建 Azure Web 应用、启用 Git 发布，然后为 Web 应用启用 WebSocket 支持。

> [AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=A7171371E" target="_blank">Azure 试用</a>。

1. 安装 Azure 命令行界面 (Azure CLI) 并连接到 Azure 订阅。请参阅[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)。

2. 如果这是你第一次在 Azure 中设置存储库，则需要创建登录凭据。从 Azure CLI 输入以下命令：

		azure site deployment user set [username] [password]


3. 切换到 **\\node\chat** 目录，然后使用以下命令创建新的 Azure Web 应用和本地 Git 存储库。此命令还会创建名为 'azure' 的 Git 远程连接。

		azure site create mysitename --git

	必须将“mysitename”替换为 Web 应用的唯一名称。

2. 通过使用以下命令将现有文件提交到本地存储库：

		git add .
		git commit -m "Initial commit"

3. 使用以下命令将这些文件推送到 Azure Web 应用存储库：

		git push azure master

	系统出现提示时，请输入步骤 2 中的凭据。在服务器上导入模块时您将收到状态消息。此过程完成后，应用程序将在 Azure Web 应用上托管。

 	> [AZURE.NOTE]在模块安装过程中，您可能会注意到“找不到导入的项目...”错误。这些错误可放心地忽略。

4. Socket.IO 使用的 WebSocket 在 Azure 上默认不启用。若要启用 Web 套接字，请使用以下命令：

		azure site set -w

	如果系统提示，请输入 Web 应用的名称。

	>[AZURE.NOTE]
	“azure site set -w”命令仅适用于 Azure 命令行界面 0.7.4 或更高版本。你还可以使用 [Azure 经典管理门户](https://manage.windowsazure.cn)启用 WebSocket 支持。
	><p>若要使用 [Azure 经典管理门户](https://manage.windowsazure.cn)启用 WebSocket，请选择你的 Web 应用的“配置”页，针对 Web 套接字条目选择“ON”，然后单击“保存”。
	><p>![websockets](./media/web-sites-nodejs-chat-app-socketio/websockets.png)
	
5. 要查看 Azure 上的 Web 应用，请使用以下命令启动 Web 浏览器并导航到托管的 Web 应用：

		azure site browse

你的应用现在即可在 Azure 上运行，并可使用 Socket.IO 在不同客户端之间中继聊天消息。

##<a name="scale-out"></a>向外扩展

Socket.IO 应用程序可通过__适配器__实现向外扩展，以在多个应用程序实例之间发布消息和事件。尽管有几个适配器可用，[socket.io redis](https://github.com/automattic/socket.io-redis) 适配器可轻松与 Azure Redis 缓存功能一同使用。

> [AZURE.NOTE]向外扩展 Socket.IO 解决方案还要求支持粘滞会话。默认情况下，可以通过 Azure 请求路由为 Azure Web 应用启用粘滞会话。有关详细信息，请参阅 [Azure 中的实例关联](http://azure.microsoft.com/blog/2013/11/18/disabling-arrs-instance-affinity-in-windows-azure-web-sites/)

###创建 Redis 缓存

执行[在 Azure Redis 缓存中创建缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#create-a-cache)中的步骤，以创建新的缓存。

> [AZURE.NOTE]保存用于缓存的__主机名__和__主密钥__，因为接下来的步骤需要这些信息。

###添加 redis 和 socket.io redis 模块

1. 在命令行中，切换到 __\\node\\chat__ 目录，然后运行以下命令：

		npm install socket.io-redis@0.1.4 redis@0.12.1 --save

	> [AZURE.NOTE]此命令中指定的版本是测试本文时使用的版本。

2. 修改 __app.js__ 文件，紧接在 `var io = require('socket.io')(server);` 后面添加以下行

		var pub = require('redis').createClient(6379,'redishostname', {auth_pass: 'rediskey', return_buffers: true});
		var sub = require('redis').createClient(6379,'redishostname', {auth_pass: 'rediskey', return_buffers: true});

		var redis = require('socket.io-redis');
		io.adapter(redis({pubClient: pub, subClient: sub}));

	用你的 Redis 缓存的主机名和密钥替换 __redishostname__ 和 __rediskey__。

	这将创建到之前创建的 Redis 缓存的发布和订阅客户端。客户端然后与适配器一同使用，以配置 Socket.IO 将 Redis 缓存用于在应用程序的实例之间传递消息和事件

	> [AZURE.NOTE]尽管 __socket.io redis__ 适配器能够与 Redis 直接通信，但当前版本不支持 Azure Redis 缓存所需的身份验证。因此需使用 __redis__ 模块创建初始连接，将客户端传递给 __socket.io redis__ 适配器。
	> <p>尽管 Azure Redis Cache 支持使用端口 6380 进行安全连接，但此示例中使用的模块不支持自 2014 年 7 月 14 日起的安全连接。上述代码使用默认的 6379 非安全端口。

3. 保存已修改的 __app.js__

###提交更改并重新部署

从 __\\node\\chat__ 目录中的命令行中，使用以下命令提交更改，并重新部署该应用程序。

	git add .
	git commit -m "implementing scale out"
	git push azure master

将所做的更改推送到服务器后，您可以使用以下命令跨多个实例缩放您的 Web 应用。

	azure site scale instances --instances #

其中 __#__ 是要创建的实例数。

可以从多个浏览器或计算机连接到你的 Web 应用，以验证消息是否已正确发送给所有客户端。

## 故障排除

###连接限制

Azure Web 应用提供多个 SKU，这些 SKU 用于确定你的站点可用的资源。包括允许的 WebSocket 连接数。有关详细信息，请参阅 [ Web 应用定价页][pricing]。

###未使用 WebSocket 发送消息

如果客户端浏览器一直回退到长轮询而不是使用 Websocket，可能有以下几种原因。

* **尝试限制传输到该 Websocket**

	为了使 Socket.IO 使用 Websocket 进行消息传输，服务器和客户端必须支持 Websocket。如果其中任一个不支持，则 Socket.IO 将协商其他传输，如长轮询。Socket.IO 使用的默认传输列表为 ` websocket, htmlfile, xhr-polling, jsonp-polling`。你可以在包含 `, nicknames = {};` 的行后面将以下代码添加到 **app.js** 文件，以强制其仅使用 WebSocket。

		io.configure(function() {
		  io.set('transports', ['websocket']);
		});

	> [AZURE.NOTE]注意，上述代码为活动状态时，不支持 Websocket 的低版本浏览器将无法连接到站点，因为此代码将通信限制为仅支持 Websocket 通信。

* **使用 SSL**

	Websocket 依赖于某些较少使用的 HTTP 标头，如 **Upgrade** 标头。某些中间网络设备（例如 Web 代理）可能会删除这些标头。为避免发生此问题，可以建立基于 SSL 的 WebSocket 连接。

	完成此操作的简单方法是将 Socket.IO 配置到 `match origin protocol`。这会指示 Socket.IO 保护 Websocket 通信，使之和网页原始 HTTP/HTTPS 请求一样。如果浏览器使用 HTTPS URL 访问您的 Web 应用，将基于 SSL 保护通过 Socket.IO 的后续 WebSocket 通信。

	若要将此示例修改为启用此配置，请在 **app.js** 文件中包含 `, nicknames = {};` 的行后面添加以下代码。

		io.configure(function() {
		  io.set('match origin protocol', true);
		});

* **验证 web.config 设置**

	托管 Node.js 应用程序的 Azure Web 应用使用 **web.config** 文件将传入请求路由到 Node.js 应用程序。为了使 Websocket 对 Node.js 应用程序正常运行，**web.config** 必须包含以下条目。

		<webSocket enabled="false"/>

	这将禁用 IIS Websocket 模块，其中包括自身的 Websocket 实施以及与 Node.js 特定 WebSocket 模块（如 Socket.IO）的冲突。如果此行不存在，或者设置为 `true`，其原因主要是 WebSocket 传输不适用于你的应用程序。

	Node.js 应用程序通常不包括 **web.config** 文件，因此 Azure Web 应用将在部署 Node.js 应用程序时自动生成一个。由于此文件是在服务器上自动生成的，因此必须使用 Web 应用的 FTP 或 FTPS URL 来查看此文件。你可以通过选择你的 Web 应用，然后选择“仪表板”链接，在 Azure 经典管理门户中查找 Web 应用的 FTP 和 FTPS URL。URL 将显示在“速览”部分。

	> [AZURE.NOTE]如果应用程序未提供 **web.config** 文件，则该文件将仅由 Azure Web 应用生成。如果在应用程序项目的根目录下提供了 **web.config** 文件，则 Azure Web 应用将使用该文件。

	如果该条目不存在，或者已设置为值 `true`，则你应在 Node.js 应用程序的根目录中创建 **web.config** 并指定值 `false`。例如，使用 **app.js** 作为入口点的应用程序的默认 **web.config** 如下所示。

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

	如果你的应用程序使用的入口点不是 **app.js**，你必须将出现的所有 **app.js** 替换为正确的入口点。例如，将 **app.js** 替换为 **server.js**。

##后续步骤

在本教程中，你已学习如何创建一个在 Azure 中托管的聊天应用程序。您还可以将该应用程序作为 Azure 云服务托管。有关如何实现此目的的步骤，请参阅[在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序][cloudservice]。

[socketio]: http://socket.io/
[completed-app]: ./media/web-sites-nodejs-chat-app-socketio/websitesocketcomplete.png
[Socket.IO GitHub 存储库]: https://github.com/Automattic/socket.io
[release]: https://github.com/Automattic/socket.io/releases
[cloudservice]: /documentation/articles/cloud-services-nodejs-chat-app-socketio/

[chat-example-view]: ./media/web-sites-nodejs-chat-app-socketio/socketio-2.png
[npm-output]: ./media/web-sites-nodejs-chat-app-socketio/socketio-7.png
[pricing]: /home/features/web-site/pricing/
 

<!---HONumber=Mooncake_1207_2015-->