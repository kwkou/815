## 使用 EventProcessorHost 接收消息

[EventProcessorHost] 是一个 .NET 类，它通过从事件中心管理持久检查点和并行接收来简化从这些事件中心接收事件。通过使用 [EventProcessorHost]，可跨多个接收方拆分事件，即便是承载于不同节点中也是如此。此示例演示如何为单一接收方使用 [EventProcessorHost]。[经过扩展的事件处理示例]演示了如何将 [EventProcessorHost] 与多个接收方结合使用。

有关事件中心接收模式的详细信息，请参阅[事件中心概述]。

若要使用 [EventProcessorHost]，您必须具有一个 [Azure 存储帐户]：

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的"新建"。

2. 依次单击"数据服务"、"存储"和"快速创建"，然后为您的存储帐户键入一个名称。选择所需的区域，然后单击"创建存储帐户"。

   	![][11]

3. 单击新创建的存储帐户，然后单击"管理访问密钥"：

   	![][12]

	请复制该访问密钥，以供将来使用。

4. 在 Visual Studio 中，使用"控制台应用程序"项目模板创建一个新的 Visual C# 桌面应用项目。将该项目命名为 **Receiver**。

   	![][14]

5. 在"解决方案资源管理器"中，右键单击该解决方案，然后单击"管理 NuGet 程序包"。 

	此时将显示"管理 NuGet 程序包"对话框。

6. 搜索  `Microsoft Azure Service Bus Event Hub - EventProcessorHost`、单击"安装"，并接受使用条款。 

	![][13]

	这样，便会下载、安装 <a href="https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost">Azure 服务总线事件中心 - EventProcessorHost NuGet 程序包</a>并添加对该程序包的引用，包括其所有依赖项。

7. 创建一个名为 **SimpleEventProcessor** 的新类，并在该文件的顶部添加以下语句：

		using Microsoft.ServiceBus.Messaging;
		using System.Diagnostics;
		using System.Threading.Tasks;

	然后, 插入以下代码作为该类的正文：

		class SimpleEventProcessor : IEventProcessor
	    {
	        Stopwatch checkpointStopWatch;
	        
	        async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
	        {
	            Console.WriteLine(string.Format("Processor Shuting Down.  Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason.ToString()));
	            if (reason == CloseReason.Shutdown)
	            {
	                await context.CheckpointAsync();
	            }
	        }
	
	        Task IEventProcessor.OpenAsync(PartitionContext context)
	        {
	            Console.WriteLine(string.Format("SimpleEventProcessor initialize.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset));
	            this.checkpointStopWatch = new Stopwatch();
	            this.checkpointStopWatch.Start();
	            return Task.FromResult<object>(null);
	        }
	
	        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
	        {
	            foreach (EventData eventData in messages)
	            {
	                string data = Encoding.UTF8.GetString(eventData.GetBytes());
	                
	                Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{2}'",
	                    context.Lease.PartitionId, data));
	            }
	
	            //Call checkpoint every 5 minutes, so that worker can resume processing from the 5 minutes back if it restarts.
	            if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
	            {
	                await context.CheckpointAsync();
	                lock (this)
	                {
	                    this.checkpointStopWatch.Reset();
	                }
	            }
	        }
	    }

	此类将由 **EventProcessorHost** 调用，以处理从事件中心接收的事件。请注意， `SimpleEventProcessor` 类使用秒表以定期对 **EventProcessorHost** 上下文调用检查点方法。这将确保接收方重新启动时将会丢失的处理工作不会超过五分钟。

8. 在 **Program** 类中，在顶部添加以下  `using` 语句：

		using Microsoft.ServiceBus.Messaging;
		using System.Threading.Tasks;
	
	然后，在 **Main** 方法中添加以下代码，从而替换事件中心名称和连接字符串，以及存储帐户和您在前面章节中复制的密钥：

		string eventHubConnectionString = "{event hub connection string}";
        string eventHubName = "{event hub name}";
        string storageAccountName = "{storage account name}";
        string storageAccountKey = "{storage account key}";
        string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                    storageAccountName, storageAccountKey);

        string eventProcessorHostName = Guid.NewGuid().ToString();
        EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
        eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>().Wait();
            
        Console.WriteLine("Receiving. Press enter key to stop worker.");
        Console.ReadLine();

> [AZURE.NOTE] 本教程使用了一个 [EventProcessorHost] 实例。若要增加吞吐量，建议运行多个 [EventProcessorHost] 实例，如[经过扩展的事件处理示例]中所示。在这些情况下，为了对接收到的事件进行负载平衡，各个不同实例会自动相互协调。如果希望多个接收方都各自处理  *all* 事件，则必须使用 **ConsumerGroup** 概念。在从不同计算机中接收事件时，根据部署 [EventProcessorHost] 实例的计算机（或角色）来指定该实例的名称可能会很有用。有关这些主题的详细信息，请参阅[事件中心概述]。

<!-- Links -->
[事件中心概述]: http://msdn.microsoft.com/zh-cn/library/azure/dn821413.aspx
[经过扩展的事件处理示例]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-45f43fc3
[Azure 存储帐户]: http://www.windowsazure.cn/zh-cn/documentation/articles/storage-create-storage-account/
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx 

<!-- Images -->

[11]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp2.png
[12]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp3.png
[13]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp1.png
[14]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp1.png
<!--HONumber=41-->
