<properties linkid="dev-nodejs-website-with-socketio" urlDisplayName="Website Using Socket.IO" pageTitle="使用 Socket.io 构建 Node.js 网站 - Azure 教程" metaKeywords="Azure Node.js socket.io tutorial, Azure Node.js socket.io, Azure Node.js tutorial" description="本教程将演示如何在托管在 Azure 上的 node.js 网站中使用 socket.io。" metaCanonical="" services="web-sites" documentationCenter="Node.js" title="在 Azure 网站中使用 Socket.IO 构建 Node.js 聊天应用程序" authors="larryfr" solutions="" videoId="" scriptId="" manager="paulettm" editor="mollybos" />

# 在 Azure 网站中使用 Socket.IO 构建 Node.js 聊天应用程序

Socket.IO 使用 WebSocket 在 node.js 服务器和客户端之间提供实时通信。还支持回退到使用低版本浏览器的其他传输（如长轮询）。本教程将指导您将基于 Socket.IO 的聊天应用程序作为 Azure 网站托管。有关 Socket.IO 的详细信息，请参阅 <http://socket.io/>。

> [WACOM.NOTE] 此任务中的过程适用于 Azure 网站；对于云服务，请参阅[在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序][在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序]。

以下是已完成应用程序的屏幕快照：

![显示聊天应用程序的浏览器][显示聊天应用程序的浏览器]

## <span id="Download"></span></a>下载聊天示例

对于此项目，我们将使用 [Socket.IO GitHub 存储库][Socket.IO GitHub 存储库]中的
聊天示例。执行以下步骤下载该示例并将其
添加到你前面创建的项目中。

1.  使用“克隆”按钮创建存储库的本地副本。你还可以使用“ZIP”按钮下载项目。

    ![显示 https://github.com/LearnBoost/socket.io/tree/master/examples/chat 的浏览器窗口，其中 ZIP 下载图标突出显示][显示 https://github.com/LearnBoost/socket.io/tree/master/examples/chat 的浏览器窗口，其中 ZIP 下载图标突出显示]

2.  浏览本地存储库的目录结构，找到 **examples\\chat** 目录
    将该目录的内容复制到单独的目录中，例如
    **\\node\\chat**。

## <span id="Modify"></span></a>修改 App.js 并安装模块

在将该应用程序部署到 Azure 之前，必须进行少量修改。
对 app.js 文件执行以下步骤：

1.  在记事本或其他文本编辑器中打开 app.js 文件。

2.  在 app.js 的开头处查找 **Module dependencies** 部分，并将包含 **sio = require('..//..//lib//socket.io')** 的行更改为 **sio = require('socket.io')**，如下所示：

        var express = require('express')
        , stylus = require('stylus')
        , nib = require('nib')
        //, sio = require('..//..//lib//socket.io'); //Original
        , sio = require('socket.io');                //Updated

3.  为了确保应用程序侦听正确的端口，请在记事本或您喜爱的
    编辑器中打开 app.js，然后将 **3000** 替换为 **process.env.port** 来
    更改以下行，如下所示：

        //app.listen(3000, function () {            //Original
        app.listen(process.env.PORT, function () {  //Updated
          var addr = app.address();
          console.log('   app listening on http://' + addr.address + ':' + addr.port);
        });

在保存对 app.js 的更改后，使用以下步骤安装所需模块：

1.  在命令行中，将目录更改为 **\\node\\chat** 目录，然后使用以下命令安装此应用程序所需的模块：

        npm install

    这将安装 package.json 文件中所列的模块。命令
    完成后，你应该看到类似下面的
    输出：

    ![npm install 命令的输出][npm install 命令的输出]

2.  因为此示例最初是 Socket.IO GitHub 存储库的
    一部分，并通过相对路径直接引用了 Socket.IO 库，
    而 package.json 文件中未引用 Socket.IO，所以我们必须
    通过发出以下命令来安装它：

        npm install socket.io -save

## <span id="Publish"></span></a>创建 Azure 网站

按照以下步骤创建 Azure 网站、启用 Git 发布，然后为网站启用 WebSocket 支持。

> [WACOM.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 \<a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/\>Azure 免费试用版\</a\>。\</p\> \</blockquote\> \<ol\> \<li\>\<p\>在命令行中，将目录更改为 \<strong\>\\node\\chat\</strong\> 目录，然后使用以下命令创建一个新的 Azure 网站并为该网站和本地目录启用 Git 存储库。这还将创建一个名为“azure”的 Git remote。\</p\> \<pre\>\<code data-inline="1"\>azure site create mysitename --git\</code\>\</pre\> \<p\>您必须将“mysitename”替换为网站的唯一名称。\</p\>\</li\> \<li\>\<p\>使用以下命令将现有文件提交到本地存储库：\</p\> \<pre\>\<code\>git add . git commit -m "Initial commit"\</code\>\</pre\>\</li\> \<li\>\<p\>使用以下命令将这些文件推送到 Azure 网站存储库：\</p\> \<pre\>\<code data-inline="1"\>git push azure master\</code\>\</pre\> \<p\>模块导入服务器时您将收到状态消息。此过程完成后，应用程序将在 Azure 网站上托管。\</p\> \<blockquote\> \<p\>[WACOM.NOTE] 在模块安装过程中，您可能会注意到“找不到导入的项目...”错误。这些错误可放心地忽略。\</p\> \</blockquote\>\</li\> \<li\>\<p\>Socket.IO 使用的 WebSocket 在 Azure 上默认不启用。要启用 Web 套接字，请使用以下命令：\</p\> \<pre\>\<code data-inline="1"\>azure site set -w\</code\>\</pre\> \<p\>如果系统提示，请输入网站名称。\</p\> \<blockquote\> \<p\>[WACOM.NOTE]\<br /\>“azure site set -w”命令仅适用于 Azure 跨平台命令行接口版本 0.7.4 或更高版本。您还可以使用 Azure 管理门户启用 WebSocket 支持。\</p\> \<p\>要使用 \<a href="https://manage.windowsazure.cn"\>Azure 管理门户\</a\>启用 WebSocket，请选择网站的“配置”页，为 Web 套接字条目选择“打开”，然后单击“保存”。\</p\> \<p\>\<img src="./media/web-sites-nodejs-chat-app-socketio/websockets.png" alt="WebSocket" /\>\</p\> \</blockquote\>\</li\> \<li\>\<p\>要查看 Azure 上的网站，请使用以下命令启动 Web 浏览器并导航到托管的网站：\</p\> \<pre\>\<code data-inline="1"\>azure site browse\</code\>\</pre\>\</li\> \</ol\> \<p\>您的应用程序现在即可在 Azure 上运行，并可使用 Socket.IO 在\<br /\>不同客户端之间中继聊天消息。\</p\> \<blockquote\> \<p\>[WACOM.NOTE] 为简单起见，此示例仅限于连接到同一实例的用户之间的聊天。这意味着如果云服务创建两个辅助角色实例，用户将只能够与连接到同一辅助角色实例的其他用户聊天。要缩放应用程序以使用多个角色实例，你可以使用像 Service Bus 这样的技术在实例之间共享 Socket.IO 存储状态。有关示例，请参阅 \<a href="https://github.com/WindowsAzure/azure-sdk-for-node"\>Azure SDK for Node.js GitHub 存储库的\</a\>中的 Service Bus 队列和主题使用示例。\</p\> \</blockquote\> \<h2 id="troubleshooting"\>\<a id="tshooting"\>\</a\>故障排除\</h2\> \<h3 id="connection-limits"\>连接限制\</h3\> \<p\>Azure 网站提供多个 SKU，这些 SKU 用于确定您的站点可用的资源。包括允许的 WebSocket 连接数。有关详细信息，请参阅\<a href="/zh-cn/pricing/details/web-sites/"\>网站定价页\</a\>。\</p\> \<h3 id="messages-arent-being-sent-using-websockets"\>未使用 WebSocket 发送消息\</h3\> \<p\>如果客户端浏览器一直回退到长轮询而不是使用 Websocket，可能有以下几种原因：\</p\> \<ul\> \<li\>\<p\>\<strong\>尝试只允许传输到 Websocket\</strong\>\</p\> \<p\>为了使 Socket.IO 使用 Websocket 进行消息传输，服务器和客户端必须支持 Websocket。如果其中任一个不支持，则 Socket.IO 将协商其他传输，如长轮询。Socket.IO 使用的默认传输列表为 \<code data-inline="1"\>websocket, htmlfile, xhr-polling, jsonp-polling\</code\>。您可以在 \<strong\>app.js\</strong\> 文件中包含 \<code data-inline="1"\>, nicknames = {};\</code\> 的行后面添加以下代码，强制其只使用 WebSocket。\</p\> \<pre\>\<code\>io.configure(function() { io.set('transports', ['websocket']); });\</code\>\</pre\> \<blockquote\> \<p\>[WACOM.NOTE] 注意，上述代码为活动状态时，不支持 Websocket 的低版本浏览器将无法连接到站点，因为此代码将通信限制为仅支持 Websocket 通信。\</p\> \</blockquote\>\</li\> \<li\>\<p\>\<strong\>使用 SSL\</strong\>\</p\> \<p\>Websocket 依赖于某些较少使用的 HTTP 标头，如 \<strong\>Upgrade\</strong\> 标头。某些中间网络设备（例如 Web 代理）可能会删除这些标头。为避免发生此问题，可以建立基于 SSL 的 WebSocket 连接。\</p\> \<p\>实现此目的的一种简单的方法是将 Socket.IO 配置为 \<code data-inline="1"\>match origin protocol\</code\>。这会指示 Socket.IO 保护 Websocket 通信，使之和网页原始 HTTP/HTTPS 请求一样。如果浏览器使用 HTTPS URL 访问您的网站，将基于 SSL 保护通过 Socket.IO 的后续 WebSocket 通信\</p\> \<p\>要将此示例修改为启用此配置，请在 \<strong\>app.js\</strong\> 文件中包含 \<code data-inline="1"\>, nicknames = {};\</code\> 的行后面添加以下代码：\</p\> \<pre\>\<code\>io.configure(function() { io.set('match origin protocol', true); });\</code\>\</pre\>\</li\> \<li\>\<p\>\<strong\>确认 web.config 设置\</strong\>\</p\> \<p\>托管 Node.js 应用程序的 Azure 网站使用 \<strong\>web.config\</strong\> 文件将传入请求路由到 Node.js 应用程序。为了使 Websocket 对 Node.js 应用程序正常运行，\<strong\>web.config\</strong\> 必须包含以下条目。\</p\> \<pre\>\<code data-inline="1"\>\<webSocket enabled="false"/\>\</code\>\</pre\> \<p\>此条目将禁用 IIS Websocket 模块，该模块包含自己的 Websocket 实现，并与 Node.js 特定的 WebSocket 模块（如 Socket.IO）冲突。如果此行不存在，或者已设置为 \<code data-inline="1"\>true\</code\>，这可能是 WebSocket 传输不适用于您的应用程序的原因。\</p\> \<p\>Node.js 应用程序通常不包括 \<strong\>web.config\</strong\> 文件，因此 Azure 网站将在部署 Node.js 应用程序时自动生成一个。由于此文件是在服务器上自动生成的，因此必须使用网站的 FTP 或 FTPS URL 来查看此文件。您可以通过选择您的网站，然后选择“仪表板”\<strong\>\</strong\>链接，在 Azure 管理门户中查找网站的 FTP 和 FTPS URL。URL 将显示在“速览”\<strong\>\</strong\>部分。\</p\> \<blockquote\> \<p\>[WACOM.NOTE] 如果应用程序未提供 \<strong\>web.config\</strong\> 文件，则该文件将仅由 Azure 网站生成。如果在应用程序项目的根目录下提供了 \<strong\>web.config\</strong\> 文件，则 Azure 网站将使用该文件。\</p\> \</blockquote\> \<p\>如果该条目不存在，或者已设置为值 \<code data-inline="1"\>true\</code\>，则您应在 Node.js 应用程序的根目录中创建 \<strong\>web.config\</strong\> 并指定值 \<code data-inline="1"\>false\</code\>。例如，使用 \<strong\>app.js\</strong\> 作为入口点的应用程序的默认 \<strong\>web.config\</strong\> 如下所示。\</p\> \<pre\>\<code\>\<?xml version="1.0" encoding="utf-8"?\> \<!-- 如果使用 iisnode 在 IIS 或 IIS Express 后面运行节点进程， 则需要此配置文件。有关详细信息，请访问： https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config --\> \<configuration\> \<system.webServer\> \<!-- 有关 WebSocket 支持的详细信息，请访问 http://blogs.msdn.com/b/windowsazure/archive/2013/11/14/introduction-to-websockets-on-windows-azure-web-sites.aspx --\> \<webSocket enabled="false" /\> \<handlers\> \<!-- 表示 server.js 文件为 iisnode 模块将要处理的 node.js 站点 --\> \<add name="iisnode" path="app.js" verb="\*" modules="iisnode"/\> \</handlers\> \<rewrite\> \<rules\> \<!-- 不要影响 node-inspector 调试的请求 --\> \<rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true"\> \<match url="^app.js\\/debug[\\/]?" /\> \</rule\> \<!-- 我们首先考虑传入 URL 与 /public 文件夹中的物理文件是否匹配 --\> \<rule name="StaticContent"\> \<action type="Rewrite" url="public{REQUEST\_URI}"/\> \</rule\> \<!-- 所有其他 URL 将映射到 node.js 站点入口点 --\> \<rule name="DynamicContent"\> \<conditions\> \<add input="{REQUEST\_FILENAME}" matchType="IsFile" negate="True"/\> \</conditions\> \<action type="Rewrite" url="app.js"/\> \</rule\> \</rules\> \</rewrite\> \<!-- 您可以使用以下选项控制 Node 在 IIS 中的托管方式： \* watchedFiles：以分号分隔的文件列表，将监视其更改以重新启动服务器 \* node\_env：将作为 NODE\_ENV 环境变量传播到节点 \* debuggingEnabled - 控制是否启用内置调试器 有关选项的完整列表，请参阅 https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config --\> \<!--\<iisnode watchedFiles="web.config;\*.js"/\>--\> \</system.webServer\> \</configuration\>\</code\>\</pre\> \<blockquote\> \<p\>[WACOM.NOTE] 如果您的应用程序使用的入口点不是 \<strong\>app.js\</strong\>，您必须将出现的所有 \<strong\>app.js\</strong\> 替换为正确的入口点。例如，将 \<strong\>app.js\</strong\> 替换为 \<strong\>server.js\</strong\>。\</p\> \</blockquote\>\</li\> \</ul\> \<h2 id="next-steps"\>后续步骤\</h2\> \<p\>在本教程中，您将了解如何创建在 Azure 网站中托管的聊天应用程序。您还可以将该应用程序作为 Azure 云服务托管。有关如何实现此目的的步骤，请参阅\<a href="/zh-cn/develop/nodejs/tutorials/app-using-socketio/"\>在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序\</a\>。\</p\>

  [在 Azure 云服务中使用 Socket.IO 构建 Node.js 聊天应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-nodejs-chat-app-socketio/
  [显示聊天应用程序的浏览器]: ./media/web-sites-nodejs-chat-app-socketio/websitesocketcomplete.png
  [Socket.IO GitHub 存储库]: https://github.com/LearnBoost/socket.io/tree/0.9.14
  [显示 https://github.com/LearnBoost/socket.io/tree/master/examples/chat 的浏览器窗口，其中 ZIP 下载图标突出显示]: ./media/web-sites-nodejs-chat-app-socketio/socketio-2.png
  [npm install 命令的输出]: ./media/web-sites-nodejs-chat-app-socketio/socketio-7.png
