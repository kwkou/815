<properties
    pageTitle="Azure 事件中心 .NET Framework API 概述 | Azure"
    description="汇总了一些重要的事件中心 .NET Framework 客户端 API。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="7f3b6cc0-9600-417f-9e80-2345411bd036"
    ms.service="event-hubs"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/30/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub;sethm" />  


# 事件中心 .NET Framework API 概述
本文汇总了一些重要的事件中心 .NET Framework 客户端 API。有两个类别：管理 API 和运行时 API。运行时 API 包括发送和接收消息所需的全部操作。借助管理操作，可以通过创建、更新和删除实体来管理事件中心实体状态。

监视方案跨越管理操作和运行时操作。有关 .NET API 的详细参考文档，请参阅[服务总线 .NET](https://docs.microsoft.com/zh-cn/dotnet/api/) 和 [EventProcessorHost API](https://docs.microsoft.com/zh-cn/dotnet/api/) 参考。

## 管理 API
若要执行以下管理操作，必须对事件中心命名空间具有**管理**权限：

### 创建

    // Create the Event Hub
    var ehd = new EventHubDescription(eventHubName);
    ehd.PartitionCount = SampleManager.numPartitions;
    await namespaceManager.CreateEventHubAsync(ehd);

### 更新

    var ehd = await namespaceManager.GetEventHubAsync(eventHubName);

    // Create a customer SAS rule with Manage permissions
    ehd.UserMetadata = "Some updated info";
    var ruleName = "myeventhubmanagerule";
    var ruleKey = SharedAccessAuthorizationRule.GenerateRandomKey();
    ehd.Authorization.Add(new SharedAccessAuthorizationRule(ruleName, ruleKey, new AccessRights[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send} )); 
    await namespaceManager.UpdateEventHubAsync(ehd);

### 删除

    await namespaceManager.DeleteEventHubAsync("Event Hub name");

## 运行时 API
### 创建发布者

    // EventHubClient model (uses implicit factory instance, so all links on same connection)
    var eventHubClient = EventHubClient.Create("Event Hub name");

### 发布消息

    // Create the device/temperature metric
    var info = new MetricEvent() { DeviceId = random.Next(SampleManager.NumDevices), Temperature = random.Next(100) };
    var data = new EventData(new byte[10]); // Byte array
    var data = new EventData(Stream); // Stream 
    var data = new EventData(info, serializer) //Object and serializer 
    {
        PartitionKey = info.DeviceId.ToString()
    };

    // Set user properties if needed
    data.Properties.Add("Type", "Telemetry_" + DateTime.Now.ToLongTimeString());

    // Send single message async
    await client.SendAsync(data);

### 创建使用者

    // Create the Event Hubs client
    var eventHubClient = EventHubClient.Create(EventHubName);

    // Get the default consumer group
    var defaultConsumerGroup = eventHubClient.GetDefaultConsumerGroup();

    // All messages
    var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index);

    // From one day ago
    var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index, startingDateTimeUtc:DateTime.Now.AddDays(-1));

    // From specific offset, -1 means oldest
    var consumer = await defaultConsumerGroup.CreateReceiverAsync(partitionId: index,startingOffset:-1); 

### 使用消息

    var message = await consumer.ReceiveAsync();

    // Provide a serializer
    var info = message.GetBody<Type>(Serializer)

    // Get a byte[]
    var info = message.GetBytes(); 
    msg = UnicodeEncoding.UTF8.GetString(info);

## 事件处理程序主机 API
这些 API 通过在可用工作进程之间分布分区，为可能变为不可用的工作进程提供复原能力。

    // Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.
    // Use the EventData.Offset value for checkpointing yourself, this value is unique per partition.

    var eventHubConnectionString = System.Configuration.ConfigurationManager.AppSettings["Microsoft.ServiceBus.ConnectionString"];
    var blobConnectionString = System.Configuration.ConfigurationManager.AppSettings["AzureStorageConnectionString"]; // Required for checkpoint/state

    var eventHubDescription = new EventHubDescription(EventHubName);
    var host = new EventProcessorHost(WorkerName, EventHubName, defaultConsumerGroup.GroupName, eventHubConnectionString, blobConnectionString);
    await host.RegisterEventProcessorAsync<SimpleEventProcessor>();

    // To close
    await host.UnregisterEventProcessorAsync();

[IEventProcessor](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.ieventprocessor) 接口定义如下：

    public class SimpleEventProcessor : IEventProcessor
    {
        IDictionary<string, string> map;
        PartitionContext partitionContext;

        public SimpleEventProcessor()
        {
            this.map = new Dictionary<string, string>();
        }

        public Task OpenAsync(PartitionContext context)
        {
            this.partitionContext = context;

            return Task.FromResult<object>(null);
        }

        public async Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            foreach (EventData message in messages)
            {
                // Process messages here
            }

            // Checkpoint when appropriate
            await context.CheckpointAsync();

        }

        public async Task CloseAsync(PartitionContext context, CloseReason reason)
        {
            if (reason == CloseReason.Shutdown)
            {
                await context.CheckpointAsync();
            }
        }
    }

## 后续步骤
若要了解有关事件中心方案的详细信息，请访问以下链接：

* [什么是 Azure 事件中心？](/documentation/articles/event-hubs-what-is-event-hubs/)
* [事件中心编程指南](/documentation/articles/event-hubs-programming-guide/)

下面提供了 .NET API 参考：

* [Microsoft.ServiceBus.Messaging](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging)
<!-- Wrong reference here * [Microsoft.Azure.ServiceBus.EventProcessorHost](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.servicebus.eventprocessorhost)-->

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:new article about dotnet framework api for event hubs-->