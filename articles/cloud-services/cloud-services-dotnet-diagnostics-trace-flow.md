<properties
	pageTitle="使用 Azure 诊断跟踪云服务应用程序中的流 | Azure"
	description="将跟踪消息添加到 Azure 应用程序，以帮助调试、性能度量、监视和流量分析等。"
	services="cloud-services"
	documentationCenter=".net"
	authors="rboucher"
	manager="jwhit"
	editor=""/>  


<tags
	ms.service="cloud-services"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="02/20/2016"
	wacn.date="12/26/2016"
	ms.author="robb"/>



# 使用 Azure 诊断跟踪云服务应用程序的流

跟踪是在应用程序运行时监视其执行情况的一种方式。可以使用 [System.Diagnostics.Trace](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.trace.aspx)、[System.Diagnostics.Debug](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.debug.aspx) 和 [System.Diagnostics.TraceSource](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracesource.aspx) 类在日志、文本文件或其他设备中记录有关错误和应用程序执行情况的信息，供以后进行分析。有关跟踪的详细信息，请参阅[跟踪和检测应用程序](https://msdn.microsoft.com/zh-cn/library/zs6s4h68.aspx)。


## 使用 Trace 语句和 Trace 开关

通过将 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 添加到应用程序配置，并在应用程序代码中调用 System.Diagnostics.Trace 或 System.Diagnostics.Debug，在云服务应用程序中实施跟踪。对辅助角色使用配置文件 *app.config*，对 Web 角色使用配置文件 *web.config*。使用 Visual Studio 模板创建新的托管服务时，系统会针对添加的角色将 Azure 诊断自动添加到项目，并将 DiagnosticMonitorTraceListener 添加到相应的配置文件。

有关如何放置 Trace 语句的信息，请参阅[如何：向应用程序代码添加 Trace 语句](https://msdn.microsoft.com/zh-cn/library/zd83saa2.aspx)。

通过在代码中放置 [Trace 开关](https://msdn.microsoft.com/zh-cn/library/3at424ac.aspx)，可以控制是否进行跟踪以及跟踪的范围。这样可以在生产环境中监视应用程序的状态。这在使用多台计算机上运行的多个组件的业务应用程序中尤其重要。有关详细信息，请参阅[如何：配置 Trace 开关](https://msdn.microsoft.com/zh-cn/library/t06xyy08.aspx)。

## 在 Azure 应用程序中配置跟踪侦听器

Trace、Debug 和 TraceSource 都需要设置“侦听器”来收集和记录发送的消息。侦听器可收集、存储和路由跟踪消息。它们将跟踪输出传输到适当的目标，如日志、窗口或文本文件。Azure 诊断使用 [DiagnosticMonitorTraceListener](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.diagnostics.diagnosticmonitortracelistener.aspx) 类。

完成以下过程之前，必须初始化 Azure 诊断监视器。若要执行此操作，请参阅[在 Azure 中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)。

请注意，如果使用 Visual Studio 提供的模板，将自动添加侦听器的配置。


### 添加跟踪侦听器

1. 打开角色的 web.config 或 app.config 文件。
2. 将以下代码添加到文件。更改 Version 属性，以使用引用的程序集的版本号。除非有更新，否则程序集版本不一定随着每个 Azure SDK 发行版而改变。

	```
	<system.diagnostics>
		<trace>
			<listeners>
				<add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener,
		          Microsoft.WindowsAzure.Diagnostics,
		          Version=2.8.0.0,
		          Culture=neutral,
		          PublicKeyToken=31bf3856ad364e35"
		          name="AzureDiagnostics">
			  	  <filter type="" />
				</add>
			</listeners>
		</trace>
	</system.diagnostics>
	```
	>[AZURE.IMPORTANT] 确保与 Microsoft.WindowsAzure.Diagnostics 程序集建立项目引用。更新上述 xml 中的版本号，以便与引用的 Microsoft.WindowsAzure.Diagnostics 程序集的版本匹配。

3. 保存 config 文件。

有关侦听器的详细信息，请参阅[跟踪侦听器](https://msdn.microsoft.com/zh-cn/library/4y5y10s7.aspx)。

完成添加侦听器步骤后，可以将 Trace 语句添加到代码。


### 将 Trace 语句添加到代码

1. 打开应用程序的源文件。例如，用于辅助角色或 Web 角色的 <RoleName>.cs 文件。
2. 添加以下 using 语句（如果尚未添加）：
	```
	    using System.Diagnostics;
	```
3. 添加 Trace 语句，以便捕获有关应用程序状态的信息。可以使用多种方法格式化 Trace 语句的输出。有关详细信息，请参阅[如何：向应用程序代码添加 Trace 语句](https://msdn.microsoft.com/zh-cn/library/zd83saa2.aspx)。
4. 保存源文件。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->