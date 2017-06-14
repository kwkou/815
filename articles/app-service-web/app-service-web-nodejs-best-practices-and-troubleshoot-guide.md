<properties
	pageTitle="Azure Web Apps 上节点应用程序的最佳做法和故障排除指南"
	description="了解 Azure Web Apps 上节点应用程序的最佳做法和故障排除步骤。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="ranjithr"
	manager="wadeh"
	editor=""/>  


<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="article"
	ms.date="06/06/2016"
	wacn.date="09/26/2016"
	ms.author="ranjithr;wadeh"/>  

    
# Azure Web Apps 上节点应用程序的最佳做法和故障排除指南

[AZURE.INCLUDE [选项卡](../../includes/app-service-web-get-started-nav-tabs.md)]

在本文中，你将了解 Azure Webapps 上运行的[节点应用程序](/documentation/articles/app-service-web-get-started-nodejs/)的最佳做法和故障排除步骤（通过 [iisnode](https://github.com/azure/iisnode)）。

>[AZURE.WARNING] 在生产站点上使用故障排除步骤时，请格外小心。建议在非生产安装（例如过渡槽）上排查应用问题，当问题修复后，请交换过渡槽与生产槽。

## IISNODE 配置

此[架构文件](https://github.com/Azure/iisnode/blob/master/src/config/iisnode_schema_x64.xml)会显示可针对 iisnode 配置的所有设置。适用于你的应用程序的一些设置如下：

* nodeProcessCountPerApplication

    此设置控制每个 IIS 应用程序启动的节点进程数目。默认值为 1。你可以将此值设置为 0，以启动与 VM 核心计数一样多的 node.exe。大部分应用程序的建议值为 0，以便你利用计算机上的所有核心。Node.exe 是单线程，因此，一个 node.exe 最多耗用 1 个核心；若要让节点应用程序发挥最大性能，你会想利用所有核心。

* nodeProcessCommandLine

    此设置控制 node.exe 的路径。你可以设置此值以指向你的 node.exe 版本。

* maxConcurrentRequestsPerProcess

    此设置控制 iisnode 发送给每个 node.exe 的并发请求数目上限。在 Azure Webapps 上，此设置的默认值为“无限”。你不必担心此设置。在 Azure Webapps 外部，默认值为 1024。你可能想要根据应用程序获取的请求数目以及应用程序处理每个请求的速度，来配置此设置。

* maxNamedPipeConnectionRetry

    此设置控制 iisnode 将在命名管道上重试连接，以将请求发送至 node.exe 的次数上限。此设置与 namedPipeConnectionRetryDelay 结合使用，可确定 iisnode 内每个请求的总超时。在 Azure Webapps 上，默认值为 200。总超时（秒）= (maxNamedPipeConnectionRetry * namedPipeConnectionRetryDelay) / 1000

* namedPipeConnectionRetryDelay

    此设置控制 iisnode 在每次重试通过命名管道将请求发送到 node.exe 之间所要等待的时间（毫秒）。默认值为 250 毫秒。
    总超时（秒）= (maxNamedPipeConnectionRetry * namedPipeConnectionRetryDelay) / 1000

    在 Azure Webapps 上，iisnode 的总超时默认为 200 * 250 毫秒 = 50 秒。

* logDirectory

    此设置控制 iisnode 用于记录 stdout/stderr 的目录。默认值是相对于主要脚本目录（主要 server.js 所在的目录）的 iisnode

* debuggerExtensionDll

    此设置控制 iisnode 在调试节点应用程序时将使用的节点检查器版本。iisnode-inspector-0.7.3.dll 和 iisnode-inspector.dll 目前是此设置仅有的 2 个有效值。默认值为 iisnode-inspector-0.7.3.dll。iisnode-inspector-0.7.3.dll 版本会使用 node-inspector-0.7.3 并使用 WebSocket，因此，需启用 Azure Webapps 上的 WebSocket，才能使用此版本。有关如何将 iisnode 配置为使用新节点检查器的更多详细信息，请参阅 <http://www.ranjithr.com/?p=98>。

* flushResponse

    IIS 的默认行为是在刷新之前或直到响应结束时（以较早出现者为准），缓冲最多 4MB 的响应数据。iisnode 会提供配置设置来替代此行为：若要在 iisnode 从 node.exe 收到响应实体主体片段时立刻刷新该片段，需在 web.config 中将 iisnode/@flushResponse 属性设置为“true”：
    
    ```
    <configuration>    
        <system.webServer>    
            <!-- ... -->    
            <iisnode flushResponse="true" />    
        </system.webServer>    
    </configuration>
    ```

    支持刷新每个响应实体主体片段会增加性能开销，而降低约 5% 的系统吞吐量（从 v0.1.13 起），因此，最好让此设置的范围仅限于需要响应流式处理的终结点（例如，在 web.config 中使用 <location> 元素）

    除此之外，对于流式处理应用程序，还需要将 iisnode 处理程序的 responseBufferLimit 设置为 0。
    
    ```
    <handlers>    
        <add name="iisnode" path="app.js" verb="*" modules="iisnode" responseBufferLimit="0"/>    
    </handlers>
    ```

* watchedFiles

    这是一个以分号分隔的文件列表，系统将监视其更改。文件更改会导致应用程序回收。每个条目都包含可选目录名称，再加上相对于主要应用程序入口点所在目录的必要文件名。只有文件名部分可以使用通配符。默认值为“*.js;web.config”

* recycleSignalEnabled

    默认值为 false。如果启用，节点应用程序可以连接到命名管道（环境变量 IISNODE\_CONTROL\_PIPE）并发送“回收”消息。这会导致正常回收 w3wp。

* idlePageOutTimePeriod

    默认值为 0，这表示已禁用此功能。如果设置为大于 0 的值，iisnode 会每隔“idlePageOutTimePeriod”毫秒将其所有子进程移出分页。若要了解移出分页的意思，请参考此[文档](https://msdn.microsoft.com/zh-cn/library/windows/desktop/ms682606.aspx)。对于耗用大量内存并且偶尔想要将内存页出至磁盘以释放一些 RAM 的应用程序，此设置很有用。

>[AZURE.WARNING] 在生产应用程序上启用以下配置设置时，请格外小心。建议不要在实际生产应用程序上启用它们。

* debugHeaderEnabled

    默认值为 false。如果设置为 true，iisnode 会将 HTTP 响应标头 iisnode-debug 添加到它所发送的每个 HTTP 响应，而 iisnode-debug 标头值是 URL。查看 URL 片段即可收集个别诊断信息，但在浏览器中打开 URL 可达到更好的视觉效果。

* loggingEnabled

    此设置控制 iisnode 记录 stdout 和 stderr 的功能。Iisnode 将从它所启动的节点进程捕获 stdout/stderr，并写入“logDirectory”设置中指定的目录。一旦启用，你的应用程序便会将日志写入文件系统，这可能会影响性能，但具体要视应用程序完成的日志记录量而定。

* devErrorsEnabled

    默认值为 false。如果设置为 true，iisnode 会在浏览器上显示 HTTP 状态代码和 Win32 错误代码。在调试特定类型的问题时，win32 代码很有用。
    
* debuggingEnabled（请勿在实际生产站点上启用）

    此设置控制调试功能。Iisnode 与节点检查器集成。通过启用此设置，可启用节点应用程序的调试功能。启用此设置后，iisnode 会在对节点应用程序发出第一个调试请求时，在“debuggerVirtualDir”目录中布置所需的节点检查器文件。你可以将请求发送到 http://yoursite/server.js/debug，以加载节点检查器。你可以使用“debuggerPathSegment”设置来控制调试 URL 段。默认情况下，debuggerPathSegment =“debug”。你可以将其设置为 GUID 之类的值，这样，其他人就更难发现。

    有关调试的更多详细信息，请查看此[链接](https://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html)。

## 方案和建议/故障排除

### 节点应用程序发出的出站调用太多。

许多应用程序想要在其定期操作中进行出站连接。例如，当请求传入时，节点应用程序会想连接别处的 REST API，并获取一些信息来处理请求。你想要在进行 http 或 https 调用时使用保持连接代理。例如，你可以在进行这些出站调用时，使用 agentkeepalive 模块作为保持连接代理。这可确保在 Azure WebApps VM 上重复使用套接字，从而减少为每个出站请求创建新套接字的开销。此外，还可确保使用较少的套接字来发出许多出站请求，因此，你不会超出每个 VM 分配的 maxSockets。对于 Azure Webapps 的建议是将 agentKeepAlive maxSockets 值设置为每个 VM 总共 160 个套接字。这意味着，如果你有 4 个 node.exe 在 VM 上运行，你需要将每个 node.exe 的 agentKeepAlive maxSockets 设置为 40，也就是每个 VM 总共 160 个。

agentKeepALive 配置示例：

```
var keepaliveAgent = new Agent({    
    maxSockets: 40,    
    maxFreeSockets: 10,    
    timeout: 60000,    
    keepAliveTimeout: 300000    
});
```

此示例假设你有 4 个 node.exe 在 VM 上运行。如果你有不同数目的 node.exe 在 VM 上运行，则必须相应地修改 maxSockets 设置。

### 节点应用程序耗用太多 CPU。

你可能会在门户上获取 Azure Webapps 针对高 CPU 耗用量提供的建议。你也可以设置监视器以监视某些[度量值](/documentation/articles/web-sites-monitor/)。在 Azure 门户仪表板上检查 CPU 使用率时，请检查 CPU 的 MAX 值，你才不会错过峰值。
在你认为应用程序耗用太多 CPU，但你无法解释的情况下，必须分析节点应用程序。

#### 在 Azure Webapps 上使用 V8 探查器分析节点应用程序

例如，假设你有一个要分析的 hello world 应用，如下所示：

```
var http = require('http');    
function WriteConsoleLog() {    
    for(var i=0;i<99999;++i) {    
        console.log('hello world');    
    }    
}

function HandleRequest() {    
    WriteConsoleLog();    
}

http.createServer(function (req, res) {    
    res.writeHead(200, {'Content-Type': 'text/html'});    
    HandleRequest();    
    res.end('Hello world!');    
}).listen(process.env.PORT);
```

转到 scm 站点 https://yoursite.scm.chinacloudsites.cn/DebugConsole

你会看到一个命令提示符，如下所示。进入 site/wwwroot 目录

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_install_v8.png)  


运行“npm install v8-profiler”命令

这应该会在 node\_modules 目录及其所有依赖项之下安装 v8 探查器。现在，编辑 server.js 以分析你的应用程序。

```
var http = require('http');    
var profiler = require('v8-profiler');    
var fs = require('fs');

function WriteConsoleLog() {    
    for(var i=0;i<99999;++i) {    
        console.log('hello world');    
    }    
}

function HandleRequest() {    
    profiler.startProfiling('HandleRequest');    
    WriteConsoleLog();    
    fs.writeFileSync('profile.cpuprofile', JSON.stringify(profiler.stopProfiling('HandleRequest')));    
}

http.createServer(function (req, res) {    
    res.writeHead(200, {'Content-Type': 'text/html'});    
    HandleRequest();    
    res.end('Hello world!');    
}).listen(process.env.PORT);
```

上述更改将分析 WriteConsoleLog 函数，然后将配置文件输出写入站点 wwwroot 下的“profile.cpuprofile”文件。将请求发送到你的应用程序。你会在站点 wwwroot 下看到所创建的“profile.cpuprofile”文件。

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_profile.cpuprofile.png)

下载此文件，你需要使用 Chrome F12 工具打开此文件。在 Chrome 上按 F12，然后单击“Profiles”（配置文件）选项卡。单击“Load”（加载）按钮。选择你刚下载的 profile.cpuprofile 文件。单击你刚加载的配置文件。

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/chrome_tools_view.png)  


你会看到 WriteConsoleLog 函数已耗用 95% 的时间，如下所示。这也会显示造成此问题的确切行号和源文件。

### 节点应用程序耗用太多内存。

你可能会在门户上获取 Azure Webapps 针对高内存耗用量提供的建议。你也可以设置监视器以监视某些[度量值](/documentation/articles/web-sites-monitor/)。在 Azure 门户仪表板上检查内存使用率时，请检查内存的 MAX 值，你才不会错过峰值。

#### node.js 的泄漏检测和堆区分 

你可以使用 [node-memwatch](https://github.com/lloyd/node-memwatch) 帮助找出内存泄漏。
你可以像安装 v8 探查器一样安装 memwatch，并编辑代码来捕获和区分堆，以找出应用程序中的内存泄漏。

### node.exe 会随机终止 

发生这种情况有几个原因：

1.  你的应用程序正在引发未捕获的异常 — 请检查 d:\\home\\LogFiles\\Application\\logging-errors.txt 文件，以了解关于所引发异常的详细信息。此文件有堆栈跟踪，因此，你可以据此修复应用程序。

2.  你的应用程序耗用太多内存，从而使其他进程无法开始运行。如果 VM 内存总量接近 100%，则进程管理器会终止 node.exe，让其他进程有机会执行一些操作。若要解决此问题，请确保应用程序未泄漏内存，或者如果应用程序真的需要使用大量内存，请增加为具有更多 RAM 的较大 VM。

### 节点应用程序未启动

如果应用程序在启动时返回 500 错误，可能有几个原因：

1.  Node.exe 未出现在正确的位置。检查 nodeProcessCommandLine 设置。

2.  主要脚本文件未出现在正确的位置。检查 web.config，并确保处理程序节中的主要脚本文件名与主要脚本文件匹配。

3.  Web.config 配置不正确 — 检查设置名称/值。

4.  冷启动 –— 应用程序的启动时间太长。如果应用程序所花的时间超过 (maxNamedPipeConnectionRetry * namedPipeConnectionRetryDelay) / 1000 秒，则 iisnode 会返回 500 错误。增加这些设置的值，以便与应用程序启动时间匹配，从而防止 iisnode 超时并返回 500 错误。

### 节点应用程序崩溃

你的应用程序正在引发未捕获的异常 — 请检查 d:\\home\\LogFiles\\Application\\logging-errors.txt 文件，以了解关于所引发异常的详细信息。此文件有堆栈跟踪，因此，你可以据此修复应用程序。

### 节点应用程序的启动时间太长（冷启动）

最常见的原因是应用程序在 node\_modules 中有大量文件，而且应用程序尝试在启动期间加载其中大多数文件。默认情况下，因为你的文件位于 Azure Webapps 的网络共享上，所以加载太多文件可能需要一些时间。
让加载速度变快的解决方案如下：

1.  使用 npm3 来安装模块，确保你有平面依赖关系结构，并且没有重复的依赖项。

2.  试着延迟加载 node\_modules，而不要在启动时加载所有模块。这表示对 require('module') 的调用应该在尝试使用模块的函数中有实际需要时进行。

3.  Azure Webapps 会提供一项称为本地缓存的功能。此功能会将你的内容从网络共享复制到 VM 上的本地磁盘。由于文件位于本地，因此，node\_modules 的加载时间快很多。 - 这篇[文档](/documentation/articles/app-service-local-cache/)更加详细地解释了如何使用本地缓存。

## IISNODE http 状态和子状态

此[源文件](https://github.com/Azure/iisnode/blob/master/src/iisnode/cnodeconstants.h)列出了 iisnode 可在发生错误时返回的所有可能的状态/子状态组合。

为应用程序启用 FREB 以查看 win32 错误代码（基于性能方面的原因，请确保只在非生产站点上启用 FREB）。

| Http 状态 | Http 子状态 | 可能的原因？                                                                                                                                                                                            
|-------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------
| 500 | 1000 | 将请求分派至 IISNODE 时发生一些问题 — 检查 node.exe 是否已启动。Node.exe 可能在启动时已崩溃。检查 web.config 配置是否有错误。 |
| 500 | 1001 | - Win32Error 0x2 - 应用未响应 URL。检查 URL 重写规则，或检查你的快速应用是否已定义正确的路由。- Win32Error 0x6d – 命名管道繁忙 — Node.exe 不接受请求，因为管道繁忙。检查高 CPU 使用率。- 其他错误 — 检查 node.exe 是否已崩溃。
| 500 | 1002 | Node.exe 崩溃 — 检查 d:\\home\\LogFiles\\logging-errors.txt 中的堆栈跟踪。 |
| 500 | 1003 | 管道配置问题 — 你不可能看到此问题，但如果看到，则表示命名管道配置不正确。 |
| 500 | 1004-1018 | 发送请求或处理 node.exe 的相关响应时发生某个错误。检查 node.exe 是否已崩溃。检查 d:\\home\\LogFiles\\logging-errors.txt 中的堆栈跟踪。 |
| 503 | 1000 | 内存不足，无法分配更多命名管道连接。检查应用为何耗用这么多内存。检查 maxConcurrentRequestsPerProcess 设置值。如果此值并非无限，而且你有许多请求，则增加此值以免出现该错误。 |
| 503 | 1001 | 无法将请求分派至 node.exe，因为应用程序正在回收。应用程序回收之后，应该会正常处理请求。 |
| 503 | 1002 | 检查 win32 错误代码的实际原因 — 无法将请求分派至 node.exe。 |
| 503 | 1003 | 命名管道太忙 — 检查节点是否正耗用大量 CPU                                                                                                                                                                                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
NODE.exe 内有名为 NODE\_PENDING\_PIPE\_INSTANCES 的设置。默认情况下，此值在 Azure Webapps 外部为 4。这表示 node.exe 在命名管道上一次只能接受 4 个请求。在 Azure Webapps 上，此值设置为 5000，应足以满足 Azure Webapps 上运行的大多数节点应用程序。你不应该在 Azure Webapps 上看到 503.1003，因为我们 NODE\_PENDING\_PIPE\_INSTANCES 的值较高。|

## 更多资源

请访问以下链接，了解有关 Azure Web Apps 上的 node.js 应用程序的详细信息。

* [Azure App Service 中的 Node.js Web 应用入门](/documentation/articles/app-service-web-get-started-nodejs/)
* [如何在 Azure App Service 中调试 Node.js Web 应用](/documentation/articles/web-sites-nodejs-debug/)
* [将 Node.js 模块与 Azure 应用程序一起使用](/documentation/articles/nodejs-use-node-modules-azure-apps/)
* [Azure App Service Web Apps: Node.js](https://blogs.msdn.microsoft.com/silverlining/2012/06/14/windows-azure-websites-node-js/)
* [Node.js 开发人员中心](/documentation/articles/nodejs-use-node-modules-azure-apps/)
* [探索神秘无比的 Kudu 调试控制台](/documentation/articles/aog-web-app-diagnostics-kudu/)

<!---HONumber=Mooncake_0815_2016-->