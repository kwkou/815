<properties
	pageTitle="在 Azure 网站中启用 Web 应用的诊断日志记录"
	description="了解如何启用诊断日志记录和将检测添加到应用程序中，以及如何访问由 Azure 记录的信息。"
	services="app-service\web"
	documentationCenter=".net"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="09/16/2015"
	wacn.date="10/22/2015"/>

# 为 Azure 网站启用诊断日志记录

Azure 提供内置的诊断以帮助调试在 Azure 网站中托管的应用程序。在本文中，你将了解如何启用诊断日志记录和将检测添加到应用程序中，以及如何访问由 Azure 记录的信息。

> [AZURE.NOTE]本文介绍了如何通过 Azure 管理门户、Azure PowerShell 和 Azure 跨平台命令行接口使用诊断日志。有关通过 Visual Studio 使用诊断日志的信息，请参阅[在 Visual Studio 中对 Azure 网站进行故障排除](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/)。

## 目录##

- [网站诊断是什么？](#whatisdiag)
- [如何：启用诊断](#enablediag)
- [如何：下载日志](#download)
- [如何：流式传输日志](#streamlogs)
- [如何：了解诊断日志](#understandlogs)
- [后续步骤](#nextsteps)

<a name="whatisdiag"></a><h2>网站诊断是什么？</h2>

Azure 网站为 Web 服务器和 Web 应用程序中的日志记录信息提供了诊断功能。这些诊断功能按逻辑分为**站点诊断**和**应用程序诊断**。

### 网站诊断

利用网站诊断，你可以启用或禁用以下功能：

- **详细错误日志记录** - 指示故障的 HTTP 状态代码（状态代码 400 或更大数字）的详细错误消息。其中可能包含有助于确定服务器返回错误代码的原因的信息。
- **失败请求跟踪** - 有关失败请求的详细信息，包括对用于处理请求的 IIS 组件和每个组件所用的时间的跟踪。在尝试提高站点性能或隔离导致要返回特定 HTTP 错误的内容时，此信息很有用。
- **Web 服务器日志记录** - 使用 [W3C 扩展日志文件格式](http://msdn.microsoft.com/zh-cn/library/windows/desktop/aa814385.aspx)的 HTTP 事务信息。这在确定总体站点度量值（如处理的请求数量或来自特定 IP 地址的请求数）时非常有用。

### 应用程序诊断

应用程序诊断可以捕获由 Web 应用程序产生的信息。ASP.NET 应用程序可使用 [System.Diagnostics.Trace](http://msdn.microsoft.com/zh-cn/library/36hhw2t6.aspx) 类将信息记录到应用程序诊断日志。例如：

	System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");

应用程序诊断允许你在使用某些代码片段时发出信息来对正在运行的应用程序进行故障排除。在尝试确定代码使用某个特定路径的原因（通常为该路径导致了错误或其他意外行为）时，这个诊断非常有用。

有关通过 Visual Studio 使用应用程序诊断的信息，请参阅[在 Visual Studio 中对 Azure 网站进行故障排除](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio)。

> [WACOM.NOTE]不同于更改 Web.config 文件，启用应用程序诊断或更改诊断日志级别不会回收应用程序运行于其中的应用域。

当你将应用程序发布到某个网站时，Azure 网站还将记录部署信息。此操作将自动执行，不会对部署日志记录进行配置设置。部署日志记录允许你确定部署失败的原因。例如，如果你使用的是自定义部署脚本，你可能会使用部署日志记录来确定该脚本失败的原因。

<a name="enablediag"></a><h2>如何：启用诊断</h2>

可通过在 [Azure 管理门户](https://manage.windowsazure.cn)中访问 Azure 网站的“配置”页启用诊断。在“配置”页面上，使用“应用程序诊断”和“网站诊断”部分来启用或禁用日志记录。

启用“应用程序诊断”时，还必须选择“日志记录级别”，并确定是将日志记录到“文件系统”、“表存储”还是记录到“Blob 存储”。三种存储位置都为记录的事件提供同样的基本信息，但与记录到“文件系统”相比，“表存储”和“Blob 存储”记录有额外信息，如实例 ID、线程 ID 以及更详细的时间戳（刻度格式）。

启用“网站诊断”时，必须为“Web 服务器日志记录”选择“存储”或“文件系统”。选择“存储”允许你选择存储帐户，然后日志会写入一个 Blob 容器。“站点诊断”的所有其他日志仅写入文件系统。

在 [Azure 门户](https://manage.windowsazure.cn) Web 应用的“配置”选项卡中，你可以选择“存储”或“文件系统”进行“Web 服务器日志记录”。选择“存储”允许你选择存储帐户，然后日志会写入一个 Blob 容器。“站点诊断”的所有其他日志仅写入文件系统。

[Azure 门户](https://manage.windowsazure.cn) Web 应用的“配置”选项卡还包含用于应用程序诊断的其他设置：

* **文件系统** - 将应用程序诊断信息存储到 Web 应用文件系统。可以通过 FTP 访问这些文件，或者，使用 Azure PowerShell 或 Azure 命令行界面 (Azure CLI) 将这些文件作为 Zip 存档下载。
* **表存储** - 将应用程序诊断信息存储在指定的 Azure 存储帐户和表名中。
* **Blob 存储** - 将应用程序诊断信息存储在指定的 Azure 存储帐户和 Blob 容器中。
* **保留期** - 默认情况下，日志不会自动从“Blob 存储”中删除。如果你希望自动删除日志，可选择“设置保留”并输入保存日志的天数。

文件系统、 表存储或 Blob 存储的任意组合可同时启用，并具有单独的日志级别配置。例如，你可能希望将错误和警告记录到 Blob 存储中作为长期的日志记录解决方案，同时将文件系统日志记录级别设置为“详细”。

三种存储位置都为记录的事件提供同样的基本信息，但与记录到“文件系统”相比，“表存储”和“Blob 存储”记录有额外信息，如实例 ID、线程 ID 以及更详细的时间戳（刻度格式）。

> [AZURE.NOTE]存储在“表存储”或“Blob 存储”中的信息只能使用存储客户端访问或由直接使用这些存储系统的应用程序访问。例如，Visual Studio 2013 包含的存储资源管理器可用于浏览表或 Blob 存储，而 HDInsight 可以访问存储在 Blob 存储中的数据。你还可以编写通过使用 [Azure SDK](/downloads/#) 之一访问 Azure 存储的应用程序。

> [AZURE.NOTE]也可以从 Azure PowerShell 中使用 **Set-AzureWebsite** cmdlet 来启用诊断。如果尚未安装 Azure PowerShell，或者尚未将其配置为使用 Azure 订阅，请参阅[如何使用 Azure PowerShell](/documentation/articles/install-configure-powershell)。

## <a name="download"></a> 如何：下载日志

存储到 Web 应用文件系统的诊断信息可使用 FTP 直接访问。还可使用 Azure PowerShell 或 Azure 命令行界面将这些信息作为 Zip 存档下载。

存储日志采用的目录结构如下：

* **应用程序日志** - /LogFiles/Application/。此文件夹包含一个或多个包含应用程序日志记录生成的信息的文本文件。

* **失败请求跟踪** - /LogFiles/W3SVC#########/。此文件夹包含一个 XSL 文件和一个或多个 XML 文件。请确保将 XSL 文件下载到 XML 文件所在的目录中，因为 XSL 文件提供了在 Internet Explorer 中查看 XML 文件时格式化和筛选这些文件的内容的功能。

* **详细错误日志** - /LogFiles/DetailedErrors/。此文件夹包含一个或多个 .htm 文件，这些文件提供大量有关所有已出现 HTTP 错误的信息。

* **Web 服务器日志** - /LogFiles/http/RawLogs。此文件夹包含使用 [W3C 扩展日志文件格式](http://msdn.microsoft.com/zh-cn/library/windows/desktop/aa814385.aspx)进行格式化的一个或多个文本文件。

* **部署日志** - /LogFiles/Git。此文件夹包含由 Azure Web 应用使用的内部部署过程生成的日志以及 Git 部署的日志。

### FTP

若要使用 FTP 访问诊断信息，请在 [Azure 管理门户](https://manage.windowsazure.cn)中访问你的 Web 应用的“仪表板”。在“速览”部分，使用“FTP 诊断日志”链接通过 FTP 访问日志文件。“部署/FTP 用户”项列出了应该用于访问 FTP 站点的用户名。

> [AZURE.NOTE]如果“部署/FTP 用户”未设置或忘记了该用户的密码，可通过使用“仪表板”中“速览”部分的“重置部署凭据”链接创建新用户和密码。

### 使用 Azure PowerShell 下载

若要下载日志文件，请启动 Azure PowerShell 的新实例并使用以下命令：

	Save-AzureWebSiteLog -Name webappname

这会将 **-Name** 参数指定的 Web 应用的日志保存到当前目录中名为 **logs.zip** 的文件中。

> [WACOM.NOTE]如果尚未安装 Azure PowerShell，或者尚未将其配置为使用 Azure 订阅，请参阅[如何使用 Azure PowerShell](/documentation/articles/install-configure-powershell/)。

### 使用 Azure 命令行界面下载

若要使用 Azure 命令行界面下载日志文件，请打开新的命令提示符、PowerShell、Bash 或终端会话并输入以下命令：

	azure site log download webappname

这会将名为“webappname”的 Web 应用的日志保存到当前目录中名为 **diagnostics.zip** 的文件中。

> [AZURE.NOTE]如果尚未安装 Azure 命令行界面 (Azure CLI)，或者尚未将其配置为使用 Azure 订阅，请参阅[如何使用 Azure CLI](/documentation/articles/xplat-cli)。

## <a name="streamlogs"></a> 如何：流式传输日志

开发应用程序时，以近乎实时的方式查看日志记录信息通常很有用。通过使用 Azure PowerShell 或 Azure 命令行界面将日志记录信息流式传输到你的开发环境，可以实现这一点。

> [AZURE.NOTE]某些类型的日志记录缓冲区会对日志文件进行写入操作，这可能导致流中的事件无序。例如，用户访问页面时出现的应用程序日志项可能显示在该页面请求所对应的 HTTP 日志项的前面。

> [AZURE.NOTE]日志流式输出还会流式传输写入任何存储在 **D:\\home\\LogFiles** 文件夹中的文本文件的信息。

### 使用 Azure PowerShell 进行流式传输

若要流式传输日志记录信息，请启动新的 Azure PowerShell 实例并使用以下命令：

	Get-AzureWebSiteLog -Name webappname -Tail

这将连接到 **-Name** 参数指定的 Web 应用，并在该 Web 应用上出现日志事件时开始将信息流式传输到 PowerShell 窗口。任何写入以 .txt、.log 或 .htm 结尾并存储在 /LogFiles 目录 (d:/home/logfiles) 中的文件的信息将流式传输至本地控制台。

若要筛选特定事件（如错误），请使用 **-Message** 参数。例如：

	Get-AzureWebSiteLog -Name webappname -Tail -Message Error

若要筛选特定日志类型（如 HTTP），请使用 **-Path** 参数。例如：

	Get-AzureWebSiteLog -Name websitename -Tail -Path http

若要查看可用的路径列表，请使用 -ListPath 参数。

> [WACOM.NOTE]如果尚未安装 Azure PowerShell，或者尚未将其配置为使用 Azure 订阅，请参阅[如何使用 Azure PowerShell](/documentation/articles/install-configure-powershell/)。

### 使用 Azure 命令行工具进行流式传输

若要流式传输日志记录信息，请打开新的命令行提示、PowerShell、Bash 或终端会话并输入以下命令：

	azure site log tail websitename

这将连接到名为“websitename”的网站，并在该网站上出现日志事件时开始将信息流式传输到 PowerShell 窗口。任何写入以 .txt、.log 或 .htm 结尾并存储在 /LogFiles 目录 (d:/home/logfiles) 中的文件的信息将流式传输至本地控制台。

若要筛选特定事件（如错误），请使用 **--Filter** 参数。例如：

	azure site log tail websitename --filter Error

若要筛选特定日志类型（如 HTTP），请使用 **--Path** 参数。例如：

	azure site log tail websitename --path http

> [WACOM.NOTE]如果尚未安装 Azure 命令行工具，或者尚未将其配置为使用 Azure 订阅，请参阅[如何使用 Azure 命令行工具](/documentation/articles/xplat-cli/)。

<a name="understandlogs"></a><h2>如何：了解诊断日志</h2>

### 应用程序诊断日志

应用程序诊断将信息以特定格式存储在 .NET 应用程序中，具体取决于是将日志存储到文件系统、 表存储还是 Blob 存储。三种存储类型所存储的基本数据信息是相同的 — 事件发生的日期和时间、产生事件的进程 ID、事件类型（信息，警告，错误）以及事件消息。

__文件系统__

记录到文件系统或使用流式传输收到的每一行都采用以下格式：

	{Date}  PID[{process id}] {event type/level} {message}

例如，一个错误事件可能类似如下所示：

	2014-01-30T16:36:59  PID[3096] Error       Fatal error on the page!

记录到文件系统提供了三种可用方法的最基本信息，仅包括时间、进程 ID、事件级别以及消息。

__表存储__

如果记录到表存储，可通过额外属性简化表中存储数据的搜索以及获得更详尽的事件信息。以下属性（列）可用于表中存储的所有实体（行）。

<table style="width:100%;border-collapse:collapse">
<thead>
<tr>
<th style="width:45%;border:1px solid black;background-color:#0099dd">属性名称</th>
<th style="border:1px solid black;vertical-align:top;background-color:#0099dd">值/格式</th>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">PartitionKey</td>
<td style="border:1px solid black;vertical-align:top">事件的日期/时间，格式为 yyyyMMddHH</td>
</tr>
</thead>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">RowKey</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">唯一标识该实体的 GUID 值</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">Timestamp</td>
<td style="border:1px solid black;vertical-align:top">事件发生的日期和时间</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">EventTickCount</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">事件发生的日期和时间，刻度格式（精度更高）</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">ApplicationName</td>
<td style="border:1px solid black;vertical-align:top">网站名称</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">级别</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">事件级别（例如错误、警告、信息）</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">EventId</td>
<td style="border:1px solid black;vertical-align:top">此事件的事件 ID<br>如果未指定，则默认为 0</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">InstanceId</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">其上发生事件的网站实例</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">Pid</td>
<td style="border:1px solid black;vertical-align:top">进程 ID</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">Tid</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">产生事件的线程的线程 ID</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">消息</td>
<td style="border:1px solid black;vertical-align:top">事件详细消息</td>
</tr>
</table>

__Blob 存储__

如果记录到 Blob 存储，数据以逗号分隔值 (CSV) 格式存储。类似于表存储，将记录额外字段以提供更详尽的事件相关信息。以下属性适用于每一行（以 CSV 格式）：

<table style="width:100%;border-collapse:collapse">
<thead>
<tr>
<th style="width:45%;border:1px solid black;background-color:#0099dd">属性名称</th>
<th style="border:1px solid black;vertical-align:top;background-color:#0099dd">值/格式</th>
</tr>
</thead>
<tr>
<td style="border:1px solid black;vertical-align:top">日期</td>
<td style="border:1px solid black;vertical-align:top">事件发生的日期和时间</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">级别</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">事件级别（例如错误、警告、信息）</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">ApplicationName</td>
<td style="border:1px solid black;vertical-align:top">网站名称</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">InstanceId</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">其上发生事件的网站实例</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">EventTickCount</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">事件发生的日期和时间，刻度格式（精度更高）</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">EventId</td>
<td style="border:1px solid black;vertical-align:top">此事件的事件 ID<br>如果未指定，则默认为 0</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">Pid</td>
<td style="border:1px solid black;vertical-align:top">进程 ID</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">Tid</td>
<td style="border:1px solid black;vertical-align:top;background-color:#8ddaf6">产生事件的线程的线程 ID</td>
</tr>
<tr>
<td style="border:1px solid black;vertical-align:top">消息</td>
<td style="border:1px solid black;vertical-align:top">事件详细消息</td>
</tr>
</table>

存储在 Blob 中的数据类似如下所示：

	date,level,applicationName,instanceId,eventTickCount,eventId,pid,tid,message
	2014-01-30T16:36:52,Error,mywebapp,6ee38a,635266966128818593,0,3096,9,An error occurred

> [AZURE.NOTE]日志的第一行将包含列标题，如示例中所示。

### 失败请求跟踪

失败请求跟踪存储在名为 __fr######.xml__ 的 XML 文件中。为了简化记录信息的查看，在该 XML 文件所在目录中提供了一个名为 __freb.xsl__ 的 XSL 样式表。在 Internet Explorer 中打开其中一个 XML 文件时，将使用 XSL 样式表以易于阅读的方式显示跟踪信息。显示结果类似如下所示：

![在浏览器中查看的失败的请求](./media/web-sites-enable-diagnostic-log/tws-failedrequestinbrowser.png)

### 详细的错误日志

详细的错误日志都是 HTML 文档，其中提供了有关已发生的 HTTP 错误的详细信息。因为这些都是 HTML 文档，所以使用 Web 浏览器查看即可。

### Web 服务器日志

Web 服务器日志使用 [W3C 扩展日志文件格式](http://msdn.microsoft.com/zh-cn/library/windows/desktop/aa814385.aspx)进行格式化。此信息可以使用文本编辑器读取，或使用诸如 [Log Parser](http://go.microsoft.com/fwlink/?LinkId=246619) 之类的实用工具进行解析。

> [AZURE.NOTE]Azure 网站生成的日志不支持 __s-computername__、__s-ip__ 或 __cs-version__ 字段。

<a name="nextsteps"></a><h2>后续步骤</h2>

- [如何监视网站](/documentation/articles/web-sites-monitor/)
- [教程 - 排除网站故障](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio)
- [在 Visual Studio 中对 Azure 网站进行故障排除](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio//)

<!---HONumber=74-->