<properties
    pageTitle="在本地监视和诊断使用 Azure Service Fabric 编写的服务 | Azure"
    description="了解如何监视和诊断本地开发计算机上使用 Microsoft Azure Service Fabric 编写的服务。"
    services="service-fabric"
    documentationcenter=".net"
    author="mani-ramaswamy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="4eebe937-ab42-4429-93db-f35c26424321"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="11/14/2016"
    wacn.date="12/26/2016"
    ms.author="subramar" />

# 在本地计算机开发安装过程中监视和诊断服务


> [AZURE.SELECTOR]
- [Windows](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)
- [Linux](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux/)

监视、检测、诊断和故障排除允许服务继续运行，同时对用户体验造成最小中断。在实际部署的生产环境中，监视和诊断至关重要。如果在开发服务期间采用类似的模型，则转移到生产环境后，可以确保诊断管道能够正常工作。Service Fabric 使服务开发人员能够轻松实现可以跨单个计算机本地开发安装和实际生产群集安装无缝工作的诊断。


## 调试 Service Fabric Java 应用程序

对于 Java 应用程序，可以使用[多个日志记录框架](http://en.wikipedia.org/wiki/Java_logging_framework)。由于 `java.util.logging` 是 JRE 的默认选项，因此也适用于 [github 中的代码示例](http://github.com/Azure-Samples/service-fabric-java-getting-started)。以下内容说明如何配置 `java.util.logging` 框架。
 
使用 java.util.logging 可将应用程序日志重定向到内存、输出流、控制台文件或套接字。对于其中的每个选项，框架中已提供默认处理程序。可以创建 `app.properties` 文件来配置应用程序的文件处理程序，将所有日志重定向到本地文件。

以下代码片段包含一个示例配置：


	handlers = java.util.logging.FileHandler
 
	java.util.logging.FileHandler.level = ALL
	java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
	java.util.logging.FileHandler.limit = 1024000
	java.util.logging.FileHandler.count = 10
	java.util.logging.FileHandler.pattern = /tmp/servicefabric/logs/mysfapp%u.%g.log             


`app.properties` 文件指向的文件夹必须存在。创建 `app.properties` 文件后，还需要修改 `<applicationfolder>/<servicePkg>/Code/` 文件夹中的入口点脚本 `entrypoint.sh`，将属性 `java.util.logging.config.file` 设置为 `app.propertes` 文件。该入口点应如以下代码片段中所示：


	java -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=<path to app.properties> -jar <service name>.jar

 
 
此设置会导致在 `/tmp/servicefabric/logs/` 中以轮替方式收集日志。使用 **%u** 和 **%g** 可以创建更多文件，文件名为 mysfapp0.log、mysfapp1.log，依此类推。默认情况下，如果未显式配置处理程序，将会注册控制台处理程序。可以在 /var/log/syslog 下查看 syslog 中的日志。
 
有关详细信息，请参阅 [github 中的代码示例](http://github.com/Azure-Samples/service-fabric-java-getting-started)。


## 调试 Service Fabric C# 应用程序


多个框架可用于在 Linux 上跟踪 CoreCLR 应用程序。有关详细信息，请参阅 [GitHub：日志记录](http:/github.com/aspnet/logging)。C# 开发人员对 EventSource 十分熟悉，因此，本文使用 EventSource 在 Linux 上跟踪 CoreCLR 示例。

首先需要包括 System.Diagnostics.Tracing，以便将日志写入内存、输出流或控制台文件。若要使用 EventSource 进行日志记录，请将以下项目添加到 project.json：


    	"System.Diagnostics.StackTrace": "4.0.1"


可以使用自定义 EventListener 侦听服务事件，然后将它们相应地重定向到跟踪文件。下面的代码片段展示使用 EventSource 和自定义 EventListener 进行日志记录的实现示例：




	 public class ServiceEventSource : EventSource
	 {
	        public static ServiceEventSource Current = new ServiceEventSource();

	        [NonEvent]
	        public void Message(string message, params object[] args)
	        {
	            if (this.IsEnabled())
	            {
	                var finalMessage = string.Format(message, args);
	                this.Message(finalMessage);
	            }
	        }
        
	        // TBD: Need to add method for sample event.

	}





	   internal class ServiceEventListener : EventListener
	   {

	        protected override void OnEventSourceCreated(EventSource eventSource)
	        {
	            EnableEvents(eventSource, EventLevel.LogAlways, EventKeywords.All);
	        }
	        protected override void OnEventWritten(EventWrittenEventArgs eventData)
	        {
	            using (StreamWriter Out = new StreamWriter( new FileStream("/tmp/MyServiceLog.txt", FileMode.Append)))           
		    {  
	                 // report all event information               
	 		 Out.Write(" {0} ",  Write(eventData.Task.ToString(), eventData.EventName, eventData.EventId.ToString(), eventData.Level,""));
	                if (eventData.Message != null)              
			    Out.WriteLine(eventData.Message, eventData.Payload.ToArray());              
		        else             
			{ 
		                string[] sargs = eventData.Payload != null ? eventData.Payload.Select(o => o.ToString()).ToArray() : null; 
		                Out.WriteLine("({0}).", sargs != null ? string.Join(", ", sargs) : "");             
			}
	           }
	        }
	    }



上述代码片段将日志输出到 `/tmp/MyServiceLog.txt` 中的文件。此文件名需要相应地更新。如果想要将日志重定向到控制台，可在自定义 EventListener 类中使用以下片段：


	public static TextWriter Out = Console.Out;


[C# 示例](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started)中的示例使用 EventSource 和自定义 EventListener 将事件记录到文件。



## 后续步骤
添加到应用程序中的跟踪代码也可用于诊断 Azure 群集中的应用程序。请查看以下文章，其中介绍了不同的工具选项，以及如何设置这些选项。
* [如何使用 Azure 诊断收集日志](/documentation/articles/service-fabric-diagnostics-how-to-setup-lad/)

<!---HONumber=Mooncake_1219_2016-->