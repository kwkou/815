<properties
    pageTitle="使用事件中心流式处理热路径中的 Azure 诊断数据 | Azure"
    description="说明如何使用事件中心从头到尾配置 Azure 诊断，包括对常见方案的指导。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="edeebaac-1c47-4b43-9687-f28e7e1e446a"
    ms.service="event-hubs"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="07/14/2016"
    wacn.date="03/24/2017"
    ms.author="sethm" />  

# 使用事件中心流式处理热路径中的 Azure 诊断数据

Azure 诊断提供了灵活的方法用于收集来自云服务虚拟机 (VM) 的指标和日志，并将结果传输到 Azure 存储。从 2016 年 3 月 (SDK 2.9) 开始，可以将诊断接收为完全自定义的数据源，并使用 [Azure 事件中心](/home/features/event-hubs/)在数秒内传输热路径数据。

支持的数据类型包括：

* Windows 事件跟踪 (ETW) 事件
* 性能计数器
* Windows 事件日志
* 应用程序日志
* Azure 诊断基础结构日志

本文说明如何使用事件中心从头到尾配置 Azure 诊断。另针对以下常见方案提供指导：

* 如何自定义接收到事件中心的日志和指标
* 如何更改每个环境中的配置
* 如何查看事件中心流数据
* 如何排查连接问题

## 先决条件

从 Azure SDK 2.9 和相应的 Azure Tools for Visual Studio 开始，使用事件中心接收 Azure 诊断就受云服务、VM、虚拟机规模集和 Service Fabric 支持。

* Azure 诊断扩展 1.6（[用于 .NET 的 Azure SDK 2.9 或更高版本](/downloads/)默认以此为目标）
* [Visual Studio 2013 或更高版本](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)
* 应用程序中使用 *.wadcfgx* 文件和以下任一方法的 Azure 诊断现有配置：
  * Visual Studio：[为 Azure 云服务和虚拟机配置诊断](/documentation/articles/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/)
  * Windows PowerShell：[使用 PowerShell 在 Azure 云服务中启用诊断](/documentation/articles/cloud-services-diagnostics-powershell/)
* 根据[事件中心入门](/documentation/articles/event-hubs-dotnet-standard-getstarted-send/)文章预配的事件中心命名空间

## 将 Azure 诊断连接到事件中心接收器
默认情况下，Azure 诊断始终将日志和指标接收到 Azure 存储帐户。通过将一个新的 **Sinks** 节添加到 *.wadcfgx* 文件的 **PublicConfig** 节中的 **WadCfg** 元素，应用程序可以额外接收到事件中心。在 Visual Studio 中，*.wadcfgx* 文件存储于以下路径：“云服务项目”>“角色”>“(RoleName)”>“diagnostics.wadcfgx”文件。

    <SinksConfig>
      <Sink name="HotPath">
        <EventHub Url="https://diags-mycompany-ns.servicebus.chinacloudapi.cn/diageventhub" SharedAccessKeyName="SendRule" />
      </Sink>
    </SinksConfig>

本例中，事件中心 URL 设置为事件中心的完全限定命名空间：事件中心命名空间 +“/”+ 事件中心名称。

事件中心 URL 显示在 [Azure 门户](http://manage.windowsazure.cn)中的“事件中心”仪表板上。

“接收器”名称可以设置为任何有效的字符串，前提是整个配置文件中始终使用同一值。

> [AZURE.NOTE]
> 此节中可能配置了其他接收器，例如 *applicationInsights*。Azure 诊断允许定义一个或多个接收器，前提是每个接收器也已在 **PrivateConfig** 节中声明。
> 
> 

此外，还必须在 *.wadcfgx* 配置文件的 **PrivateConfig** 节中声明并定义事件中心接收器。

    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
      <StorageAccount name="" key="" endpoint="" />
      <EventHub Url="https://diags-mycompany-ns.servicebus.chinacloudapi.cn/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="9B3SwghJOGEUvXigc6zHPInl2iYxrgsKHZoy4nm9CUI=" />
    </PrivateConfig>

`SharedAccessKeyName` 值必须与**事件中心**命名空间中定义的共享访问签名 (SAS) 密钥和策略相匹配。浏览到 [Azure 门户](https://manage.windowsazure.com)中的“事件中心”仪表板，单击“配置”选项卡，然后设置具有“发送”权限的命名策略（例如“SendRule”）。**StorageAccount** 也已在 **PrivateConfig** 中声明。如果这里的值有效，就不需要更改。在本示例中，我们将值保留为空，这表示下游资产将设置这些值。例如，*ServiceConfiguration.Cloud.cscfg* 环境配置文件会设置适合环境的名称和密钥。

> [AZURE.WARNING]
> 事件中心 SAS 密钥以纯文本形式存储在 *.wadcfgx* 文件中。通常，系统会将此密钥签入源代码管理，或作为生成服务器中的资产提供，因此应该适当地保护它。建议在此处使用具有“仅发送”权限的 SAS 密钥，使恶意用户只能写入事件中心，而无法侦听或进行管理。
> 
> 

## 配置 Azure 诊断日志和指标以使用事件中心接收
如前所述，所有默认和自定义诊断数据（即指标和日志）会在配置的时间间隔自动接收到 Azure 存储。使用事件中心及任何其他接收器时，可以指定层次结构中要使用事件中心接收的任何根节点或叶节点。这包括 ETW 事件、性能计数器、Windows 事件日志和应用程序日志。

请务必考虑实际上应该将多少数据点传输到事件中心。通常情况下，开发人员会传输必须快速使用和解释的低延迟热路径数据。监视警报或自动缩放规则的系统即是一例。开发人员也会配置备用的分析存储或搜索存储 — 例如，Azure 流分析、Elasticsearch、自定义监视系统或偏好的第三方监视系统。

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

下例中，接收器将应用到层次结构中的 **PerformanceCounters** 父节点，这意味着所有 **PerformanceCounters** 子节点将发送到事件中心。

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

上例中，接收器只应用到 3 个计数器：“已排队的请求数”、“拒绝的请求数”和“处理器时间百分比”。

以下示例演示开发人员如何限制发送的数据量，这些数据将作为此服务运行状况的关键指标。

    <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />

在此示例中，接收器已应用到日志，并且只筛选为错误级别跟踪。

## 部署和更新云服务应用程序与诊断配置

Visual Studio 提供最简单的路径供你部署应用程序和事件中心接收器配置。若要查看和编辑文件，请在 Visual Studio 中打开 *.wadcfgx* 文件，然后进行编辑并保存。其路径为“云服务项目”>“角色”>“(RoleName)”>“diagnostics.wadcfgx”。

此时，Visual Studio 和 Visual Studio Team System 中的所有部署和部署更新操作，以及基于 MSBuild 且使用 **/t:publish** 目标的所有命令或脚本均会在打包流程中包含 *.wadcfgx*。此外，部署和更新会使用 VM 上适当的 Azure 诊断代理扩展将文件部署到 Azure。

在部署应用程序与 Azure 诊断配置后，将立即在事件中心的仪表板中看到活动。这意味着你可以继续在侦听器客户端或选择的分析工具中查看热路径数据。

在下图中，事件中心仪表板显示从晚上 11 点之后开始发送到事件中心且状态良好的诊断数据发送操作。此时，应用程序通过更新后的 *.wadcfgx* 文件进行部署且接收器配置正确。

![][0]  


> [AZURE.NOTE]
> 当你更新 Azure 诊断配置文件 (.wadcfgx) 时，建议使用 Visual Studio 发布或 Windows PowerShell 脚本将更新推送到整个应用程序以及配置。
> 
> 

## 查看热路径数据

如前文所述，侦听和处理事件中心数据有许多用例。

一种简单的方法是创建小型测试控制台应用程序用于侦听事件中心并列显输出流。可在控制台应用程序中插入以下代码（更多详情请参见[事件中心入门](/documentation/articles/event-hubs-dotnet-standard-getstarted-send/)。

请注意，控制台应用程序必须包含[事件处理器主机 Nuget 包](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)。

请记住将 **Main** 函数中尖括号内的值替换为资源值。

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
                string eventHubConnectionString = "Endpoint= <your connection string>"
                string eventHubName = "<Event Hub name>";
                string storageAccountName = "<Storage account name>";
                string storageAccountKey = "<Storage account key>";
                string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);

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

## 排查事件中心接收器问题

* 事件中心不按预期显示传入或传出事件活动。

    检查是否已成功预配事件中心。*.wadcfgx* 中 **PrivateConfig** 节的所有连接信息必须与门户中显示的资源值匹配。请确保已在门户中定义 SAS 策略（本示例中为“SendRule”），并为其授予“发送”权限。

* 进行更新后，事件中心不再显示传入或传出事件活动。

    首先，确保事件中心和配置信息如先前所述的那样准确无误。系统有时会在部署更新时重置 **PrivateConfig**。建议的解决方法是在项目中对 *.wadcfgx* 进行所有更改，然后推送完整的应用程序更新。如果不可行，请确保诊断更新推送带 SAS 密钥的完整 **PrivateConfig**，。

* 我试过了上述建议，但事件中心仍无法正常运行。

    请尝试查看 Azure 存储表，其中包含日志和 Azure 诊断本身的错误：**WADDiagnosticInfrastructureLogsTable**。可使用 [Azure 存储资源管理器](http://www.storageexplorer.com)等工具连接到此存储帐户，查看此表，然后添加过去 24 小时的时间戳查询。你可以使用此工具导出 .csv 文件，并在 Microsoft Excel 之类的应用程序中打开它。Excel 可轻松搜索电话卡字符串（如 **EventHubs**），查看系统报告了哪些错误。

## 后续步骤
• [了解有关事件中心的详细信息](/home/features/event-hubs/)

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

本示例的补充 *ServiceConfiguration.Cloud.cscfg* 如下所示。

    <?xml version="1.0" encoding="utf-8"?>
    <ServiceConfiguration serviceName="MyFixItCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2015-04.2.6">
      <Role name="MyFixIt.WorkerRole">
        <Instances count="1" />
        <ConfigurationSettings>
          <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="YOUR_CONNECTION_STRING" />
        </ConfigurationSettings>
      </Role>
    </ServiceConfiguration>

## 后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!-- Images. -->

[0]: ./media/event-hubs-streaming-azure-diags-data/dashboard.png

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update mate properties; wording update;add next steps feature-->