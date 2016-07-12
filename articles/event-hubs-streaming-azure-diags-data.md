<!-- Add ";EndpointSuffix=core.chinacloudapi.cn to stroage links -->
<properties 
   pageTitle="使用事件中心流式处理热路径中的 Azure 诊断数据"
   description="说明如何使用事件中心从端到端配置 Azure 诊断，包括常见方案的指导。"
   services="event-hubs"
   documentationCenter="na"
   authors="tomarcher"
   manager="douge"
   editor="" />
<tags 
   ms.service="event-hubs"
   ms.date="05/08/2016"
   wacn.date="06/20/2016" />

# 使用事件中心流式处理热路径中的 Azure 诊断数据

## 概述
Azure 诊断提供了灵活的方法用于收集来自计算 VM 的指标和日志，并将其传输到 Azure 存储空间。从 2016 年 3 月 (SDK 2.9) 版本开始，可以将 Azure 诊断接收为完全自定义的数据源，并使用 Azure 事件中心在数秒中传输热路径数据。

支持的数据类型包括：

- ETW 事件
- 性能计数器
- Windows 事件日志 
- 应用程序日志
- Azure 诊断基础结构日志
 
本文说明如何使用事件中心从端到端配置 Azure 诊断。此外，还提供了常见方案的指引，例如自定义要在事件中心接收哪些日志和指标、如何在每个环境中更改配置、举例说明如何查看事件中心流数据，以及如何排查连接问题。

## 先决条件
在 Azure 诊断中接收的事件中心受所有计算类型的支持 - 云服务、VM、VMSS 和 Servic Fabric - 支持 WAD，从 Azure SDK 2.9 和相应的 Azure Tools for Visual Studio 开始。
  
- Azure 诊断扩展 1.6（[Azure SDK for.NET 2.9 或更高版本](/downloads/)默认以此为目标）
- [Visual Studio 2013 或更高版本](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)
- 在使用以下方法通过 .wadcfgx 文件在应用程序中成功配置 Azure 诊断之前：
	- Windows PowerShell：[使用 PowerShell 在 Azure 云服务中启用诊断](/documentation/articles/cloud-services-diagnostics-powershell/)
- 根据文章[事件中心入门](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)预配的事件中心命名空间

## 将 Azure 诊断连接到事件中心接收器
默认情况下，Azure 诊断始终将日志和指标接收到 Azure 存储帐户。应用程序可能会额外接收到事件中心，方法是将 **Sinks** 节添加到 .wadcfgx 文件的 **PublicConfig** 节中的 **WadCfg** 元素。在 Visual Studio 中，.wadcfgx 文件存储在“云服务项目”>“角色”>“(RoleName)”>“diagnostics.wadcfgx”文件中。
  
	  <SinksConfig>
	    <Sink name="HotPath">
	      <EventHub Url="https://diags-mycompany-ns.servicebus.chinacloudapi.cn/diageventhub" SharedAccessKeyName="SendRule" />
	    </Sink>
	  </SinksConfig>

在本示例中，事件中心 URL 设置为事件中心的完全限定的命名空间（ServiceBus 命名空间 +“/”+ 事件中心名称）。

事件中心 URL 在[Azure 管理门户](https://manage.windowsazure.cn)中的“事件中心”仪表板上显示。

“接收器”名称可以设置为任何有效的字符串，前提是在整个配置文件中一致地使用相同的值。

**注意：**此节中可能配置了其他接收器，例如“applicationInsights”。Azure 诊断允许定义一个或多个接收器，前提是每个接收器也已在 **PrivateConfig** 节中声明。

此外，必须在 .wadcfgx 配置文件的**PrivateConfig** 节中声明并定义事件中心接收器。

	  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <StorageAccount name="" key="" endpoint="" />
	    <EventHub Url="https://diags-mycompany-ns.servicebus.chinacloudapi.cn/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="9B3SwghJOGEUvXigc6zHPInl2iYxrgsKHZoy4nm9CUI=" />
	  </PrivateConfig>

**SharedAccessKeyName** 必须匹配已在 **ServiceBus/EventHub** 命名空间中定义的 SAS 密钥和策略。此操作可通过以下方式完成：浏览到[Azure 管理门户](https://manage.windowsazure.cn)中的事件中心仪表板，单击“配置”选项卡，然后设置具有“发送”权限的命名策略（例如，“SendRule”）。**StorageAccount** 也已在 **PrivateConfig** 中声明。不需要在此处更改值，特别是当它们可正常运行时。在本示例中，我们将值保留空白，这是下游资产将设置值的符号，例如 ServiceConfiguration.Cloud.cscfg 环境配置文件将设置环境的适当名称和密钥。

>[AZURE.WARNING] 请注意，事件中心 SAS 密钥以纯文本存储在 .wadcfgx 文件中。有时这会签入源代码管理，或作为生成服务器中的资产，因此应该适当地保护。建议在此处使用具有“仅发送”权限的 SAS 密钥，使任何恶意用户最多只能写入事件中心而永远无法侦听或进行管理。

## 配置 Azure 诊断日志和指标以使用事件中心接收
如前所述，所有默认和自定义诊断数据（即指标和日志）在设置的间隔自动接收到 Azure 存储空间。使用事件中心（及任何其他接收器），可以指定要以事件中心接收的层次结构中的任何根节点或叶节点。这包括 ETW 事件、性能计数器、Windows 事件日志和应用程序日志。

请务必考虑实际上应该将多少数据点传输到事件中心。通常开发人员对于必须快速使用及解释的低延迟热路径数据（例如通过监视警报系统或自动缩放规则）使用此功能。它还可用于配置备用的分析或搜索存储 - 例如，流分析、ElasticSearch、自定义监视系统或偏好的第三方监视系统。

以下是一些示例配置。

        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
        </PerformanceCounters>

在以下示例中，接收器将应用到层次结构中的父 **PerformanceCounters**，这意味着所有子 **PerformanceCounters** 将发送到事件中心。

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" sinks="HotPath" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" sinks="HotPath"/>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" sinks="HotPath"/>
        </PerformanceCounters>

在上述示例中，接收器只应用到三个计数器：“已排队的请求数”、“拒绝的请求数”和“处理器时间百分比”。

以下示例演示开发人员如何控制和限制发送到用于此服务的运行状况的关键指标的数据量。

        <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />

在此示例中，接收器已应用到日志，并且只筛选为“错误”级别跟踪。
 
## 部署和更新云服务应用程序与诊断配置

部署应用程序以及事件中心接收器配置的最简单路径是使用 Visual Studio。若要查看和进行所需的编辑，请在 Visual Studio 中打开 .wadcfgx 文件，该文件存储在“云服务项目”->“角色”->“(RoleName)”->“diagnostics.wadcfgx”文件中，并且在完成时进行保存。

此时，Visual Studio、Visual Studio Team System 或基于 MSBUILD 的命令或脚本中的所有部署和部署更新（使用 /t:publish 目标），将在打包过程中包含 .wadcfgx，并确保使用与 VM 的适当 WAD 代理扩展将它部署到 Azure。
  
成功部署应用程序与 Azure 诊断配置后，你将立即在事件中心的仪表板中看到活动。这意味着你已准备好继续在侦听器客户端或选择的分析工具中查看热路径数据。
 
在下图中，当应用程序已部署更新的 .wadcfgx 并且正确设置接收时，事件中心仪表板显示从下午 11 点开始，对事件中心状况良好的诊断数据发送。

![][0]
  
>[AZURE.NOTE] 当你对 Azure 诊断配置文件 (.wadcfgx) 进行更新时，建议将更新推送到整个应用程序，并使用 Visual Studio 发布或 Windows PowerShell 脚本的配置。

## 查看热路径数据

如前文所述，侦听和处理事件中心数据有许多用例。
  
一种简单的方法是创建小型测试控制台应用程序用于侦听事件中心并列显输出流。可在控制台应用程序中插入以下代码（[事件中心入门](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)一文中已详细说明）。

请注意，控制台应用程序必须包含 [EventProcessor Nuget 包](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)。

请记得将以下 **Main** 函数中尖括号内的值替换为资源的值。

	//Console application code for EventHub test client
	using System;
	using System.Collections.Generic;
	using System.Diagnostics;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using Microsoft.ServiceBus.Messaging;
	
	namespace EventHubListener
	{
	    class SimpleEventProcessor : IEventProcessor
	    {
	        Stopwatch checkpointStopWatch;
	
	        async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
	        {
	            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
	            if (reason == CloseReason.Shutdown)
	            {
	                await context.CheckpointAsync();
	            }
	        }
	
	        Task IEventProcessor.OpenAsync(PartitionContext context)
	        {
	            Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
	            this.checkpointStopWatch = new Stopwatch();
	            this.checkpointStopWatch.Start();
	            return Task.FromResult<object>(null);
	        }
	
	        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
	        {
	            foreach (EventData eventData in messages)
	            {
	                string data = Encoding.UTF8.GetString(eventData.GetBytes());
	
	                Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
	                    context.Lease.PartitionId, data));
	
	                foreach (var x in eventData.Properties)
	                {
	                    Console.WriteLine(string.Format("    {0} = {1}", x.Key, x.Value));
	                }
	            }
	
	            //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
	            if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
	            {
	                await context.CheckpointAsync();
	                this.checkpointStopWatch.Restart();
	            }
	        }
	    }
	
	    class Program
	    {
	        static void Main(string[] args)
	        {
	            string eventHubConnectionString = "Endpoint= <Your Connection String>”
	            string eventHubName = "<EventHub Name>";
	            string storageAccountName = "<Storage Account Name>";
	            string storageAccountKey = "<Storage Account Key>”;
	            string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1};EndpointSuffix=core.chinacloudapi.cn", storageAccountName, storageAccountKey);
	
	            string eventProcessorHostName = Guid.NewGuid().ToString();
	            EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
	            Console.WriteLine("Registering EventProcessor...");
	            var options = new EventProcessorOptions();
	            options.ExceptionReceived += (sender, e) => { Console.WriteLine(e.Exception); };
	            eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>(options).Wait();
	
	            Console.WriteLine("Receiving. Press enter key to stop worker.");
	            Console.ReadLine();
	            eventProcessorHost.UnregisterEventProcessorAsync().Wait();
	        }
	    }
	}

## 对事件中心接收器进行故障排除

- 事件中心不按预期显示传入或传出事件活动

	检查是否已成功预配事件中心。.wadcfgx 中 **PrivateConfig** 节的所有连接信息必须与门户中显示的资源值匹配。请确保已在门户中定义 SAS 策略（本示例中为“SendRule”），并为其授予“发送”权限。

- 执行更新后，事件中心不再显示传入或传出事件活动

	首先确保事件中心和配置信息全部匹配上述项目。有些问题是由于在部署更新中重置了 **PrivateConfig** 所导致的。建议的解决方法是在项目中对 .wadcfgx 进行所有更改，然后推送完整的应用程序更新。如果不可行，请确保诊断更新推送完整的 **PrivateConfig**，包括 SAS 密钥。

- 我试过了上述做法，但事件中心仍无法正常运行

	请尝试查看 Azure 存储表，其中包含日志和 Azure 诊断本身的错误：**WADDiagnosticInfrastructureLogsTable**。一个选项是使用 [Azure 存储资源管理器](http://www.storageexplorer.com)等工具连接到此存储帐户，查看此表，然后添加过去 24 小时的时间戳查询。可以使用工具来导出 CSV，并在 Microsoft Excel 等应用程序中打开，因为 Excel 可让你轻松搜索类似于“EventHubs”的调用卡字符串，以查看报告了哪些错误。

## 后续步骤
• [详细了解事件中心](/documentation/services/event-hubs/)

## 附录：完整的 Azure 诊断配置文件 (.wadcfgx) 示例
	<?xml version="1.0" encoding="utf-8"?>
	<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <WadCfg>
	      <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
	        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
	        <Directories scheduledTransferPeriod="PT1M">
	          <IISLogs containerName="wad-iis-logfiles" />
	          <FailedRequestLogs containerName="wad-failedrequestlogs" />
	        </Directories>
	        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
	          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
	        </PerformanceCounters>
	        <WindowsEventLog scheduledTransferPeriod="PT1M">
	          <DataSource name="Application!*" />
	        </WindowsEventLog>
	        <CrashDumps>
	          <CrashDumpConfiguration processName="WaIISHost.exe" />
	          <CrashDumpConfiguration processName="WaWorkerHost.exe" />
	          <CrashDumpConfiguration processName="w3wp.exe" />
	        </CrashDumps>
	        <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />
	      </DiagnosticMonitorConfiguration>
	      <SinksConfig>
	        <Sink name="HotPath">
	          <EventHub Url="https://diageventhub-py-ns.servicebus.chinacloudapi.cn/diageventhub-py" SharedAccessKeyName="SendRule" />
	        </Sink>
	        <Sink name="applicationInsights">
	          <ApplicationInsights />
	          <Channels>
	            <Channel logLevel="Error" name="errors" />
	          </Channels>
	        </Sink>
	      </SinksConfig>
	    </WadCfg>
	    <StorageAccount />
	  </PublicConfig>
	  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <StorageAccount name="" key="" endpoint="" />
	    <EventHub Url="https://diageventhub-py-ns.servicebus.chinacloudapi.cn/diageventhub-py" SharedAccessKeyName="SendRule" SharedAccessKey="YOUR_KEY_HERE" />
	  </PrivateConfig>
	  <IsEnabled>true</IsEnabled>
	</DiagnosticsConfiguration>

本示例的补充 ServiceConfiguration.Cloud.cscfg 如下所示。

	<?xml version="1.0" encoding="utf-8"?>
	<ServiceConfiguration serviceName="MyFixItCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2015-04.2.6">
	  <Role name="MyFixIt.WorkerRole">
	    <Instances count="1" />
	    <ConfigurationSettings>
	      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="YOUR_CONNECTION_STRING" />
	    </ConfigurationSettings>
	  </Role>
	</ServiceConfiguration>

<!-- Images. -->
[0]: ./media/event-hubs-streaming-azure-diags-data/dashboard.png

<!---HONumber=Mooncake_0425_2016-->