<properties 
	pageTitle="流式传输日志和控制台" 
	description="流式传输日志和控制台概述" 
	authors="btardif" 
	manager="wpickett" 
	editor="" 
	services="app-service\web" 
	documentationCenter=""/>  


<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="multiple" 
	ms.topic="article" 
	ms.date="10/12/2016" 
	wacn.date="10/31/2016" 
	ms.author="byvinyal"/>  


# 流式传输日志和控制台

## 流式传输日志

**Azure 门户预览**提供集成的流式传输日志查看器，可用于实时查看来自**应用服务**应用的跟踪事件。

设置此功能需要几个简单的步骤：

- 在你的代码中编写跟踪
- 启用应用的应用程序**诊断日志**
- 从 **Azure 门户预览**中的内置“流式传输日志”UI 中查看流。

### 如何在你的代码中编写跟踪 ###

在你的代码中编写跟踪是很容易的。在 C# 中，可以很简单地编写这样的代码：

`````````````````````````
Trace.TraceInformation("My trace statement");
`````````````````````````

`````````````````````````
Trace.TraceWarning("My warning statement");
`````````````````````````

`````````````````````````
Trace.TraceError("My error statement");
`````````````````````````

Trace 类驻留在 System.Diagnostics 命名空间中。

在 node.js 应用中，你可以编写此代码以获得同样的结果：

`````````````````````````
console.log("My trace statement").
`````````````````````````

### 如何启用和查看流式传输日志
![][BrowseSitesScreenshot]诊断基于每个应用启用。首先浏览到想要在其中启用此功能的站点。
  
![][DiagnosticsLogs]
从设置菜单中，向下滚动到“监视”部分，单击“(1) 诊断日志”。然后 **(2) 启用**“应用程序日志记录(文件系统)”或“应用程序日志记录(Blob)”，“级别”选项可供更改要捕获的跟踪的严重性级别。如果只是尝试熟悉该功能，请将级别设置为“详细”，以确保收集所有跟踪语句。

单击边栏选项卡顶部的“保存”，然后就可以开始查看日志了。

>[AZURE.NOTE] **严重性级别**越高，日志消耗的资源就越多，但生成的跟踪也越多。确保将**严重性级别**配置为适用于生产或高流量站点的正确详细级别。

![][StreamingLogsScreenshot]
若要从 Azure 门户预览中查看**流式传输日志**，请单击同样位于设置菜单的“监视”部分中的“(1) 日志流”。如果应用主动写入跟踪语句，则会以近乎实时的方式在 **(2) 流式传输日志 UI** 中看到它们。

## 控制台
**Azure 门户预览**提供对应用的控制台访问。可以浏览应用的文件系统并运行 Powershell/cmd 脚本。执行控制台命令时，受到与运行中的应用代码相同的权限集的约束。访问受保护的目录或正在运行的脚本需要提升的权限，此类访问被阻止。

![][ConsoleScreenshot]
从设置菜单中，向下滚动到“开发工具”部分，单击“(1) 控制台”，**(2) 控制台 UI** 会在右侧打开。

若要熟悉**控制台**，请尝试执行以下基本命令：

`````````````````````````
dir
`````````````````````````

`````````````````````````
cd
`````````````````````````

<!-- Images. -->

[DiagnosticsLogs]: ./media/web-sites-streaming-logs-and-console/diagnostic-logs.png
[BrowseSitesScreenshot]: ./media/web-sites-streaming-logs-and-console/browse-sites.png
[StreamingLogsScreenshot]: ./media/web-sites-streaming-logs-and-console/streaming-logs.png
[ConsoleScreenshot]: ./media/web-sites-streaming-logs-and-console/console.png

<!---HONumber=Mooncake_1024_2016-->