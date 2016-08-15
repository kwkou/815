<properties 
   pageTitle="事件中心 API 概述 | Azure"
   description="汇总了一些重要的事件中心 .NET 客户端 API。"
   services="event-hubs"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" />
<tags 
   ms.service="event-hubs"
    ms.date="04/15/2016"
   wacn.date="05/23/2016" />

# 事件中心 API 概述

本文汇总了一些重要的事件中心 .NET 客户端 API。有两个类别：管理 API 和运行时 API。运行时 API 包括发送和接收消息所需的全部操作。使用管理操作，你可以通过创建、更新和删除实体来管理事件中心实体状态。

监视方案跨越了管理操作和运行时操作。有关 .NET API 的详细参考文档，请参阅[服务总线 .NET](https://msdn.microsoft.com/zh-cn/library/azure/mt419900.aspx) 和 [EventProcessorHost API](https://msdn.microsoft.com/zh-cn/library/azure/mt445521.aspx) 参考。

## 管理 API

若要执行以下管理操作，你必须对服务总线命名空间具有**管理**权限：

### 创建

```
// Create the Event Hub
EventHubDescription ehd = new EventHubDescription(eventHubName);
ehd.PartitionCount = SampleManager.numPartitions;
namespaceManager.CreateEventHubAsync(ehd).Wait();
```

### 更新

```
EventHubDescription ehd = await namespaceManager.GetEventHubAsync(eventHubName);

// Create a customer SAS rule with Manage permissions
ehd.UserMetadata = "Some updated info";
string ruleName = "myeventhubmanagerule";
string ruleKey = SharedAccessAuthorizationRule.GenerateRandomKey();
ehd.Authorization.Add(new SharedAccessAuthorizationRule(ruleName, ruleKey, new AccessRights[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send} )); 
namespaceManager.UpdateEventHubAsync(ehd).Wait();
```

### 删除

```
namespaceManager.DeleteEventHubAsync("Event Hub name").Wait();
```

## 运行时 API

### 创建发布者

```
// EventHubClient model (uses implicit factory instance, so all links on same connection)
EventHubClient eventHubClient = EventHubClient.Create("Event Hub name");
```

### 发布消息

```
// Create the device/temperature metric
MetricEvent info = new MetricEvent() { DeviceId = random.Next(SampleManager.NumDevices), Temperature = random.Next(100) };
EventData data = new EventData(new byte[10]); // Byte array
EventData data = new EventData(Stream); // Stream 
EventData data = new EventData(info, serializer) //Object and serializer 
    {
       PartitionKey = info.DeviceId.ToString()
    };

// Set user properties if needed
data.Properties.Add("Type", "Telemetry_" + DateTime.Now.ToLongTimeString());

// Send single message async
await client.SendAsync(data);
```

### 创建使用者

```
// Create the Event Hubs client
EventHubClient eventHubClient = EventHubClient.Create(EventHubName);

// Get the default consumer group
EventHubConsumerGroup defaultConsumerGroup = eventHubClient.GetDefaultConsumerGroup();

// All messages
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index);

// From one day ago
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index, startingDateTimeUtc:DateTime.Now.AddDays(-1));
                        
// From specific offset, -1 means oldest
EventHubReceiver consumer = await defaultConsumerGroup.CreateReceiverAsync(shardId: index,startingOffset:-1); 
```

### 使用消息

```
var message = await consumer.ReceiveAsync();

// Provide a serializer
var info = message.GetBody<Type>(Serializer)
                                    
// Get a byte[]
var info = message.GetBytes(); 
msg = UnicodeEncoding.UTF8.GetString(info);
```

## 事件处理程序主机 API

这些 API 通过在可用工作进程之间分布分片，为可能变为不可用的工作进程提供复原能力。

```
// Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.
// Use the EventData.Offset value for checkpointing yourself, this value is unique per partition.

string eventHubConnectionString = System.Configuration.ConfigurationManager.AppSettings["Microsoft.ServiceBus.ConnectionString"];
string blobConnectionString = System.Configuration.ConfigurationManager.AppSettings["AzureStorageConnectionString"]; // Required for checkpoint/state

EventHubDescription eventHubDescription = new EventHubDescription(EventHubName);
EventProcessorHost host = new EventProcessorHost(WorkerName, EventHubName, defaultConsumerGroup.GroupName, eventHubConnectionString, blobConnectionString);
            host.RegisterEventProcessorAsync<SimpleEventProcessor>();

// To close
host.UnregisterEventProcessorAsync().Wait();   
```

[IEventProcessor](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.ieventprocessor.aspx) 接口定义如下：

```
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
            Process messages here
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
```

## 后续步骤

若要了解有关事件中心方案的详细信息，请访问以下链接：

- [什么是 Azure 事件中心？](/documentation/articles/event-hubs-what-is-event-hubs/)
- [事件中心概述](/documentation/articles/event-hubs-overview/)
- [事件中心编程指南](/documentation/articles/event-hubs-programming-guide/)
- [事件中心代码示例](https://code.msdn.microsoft.com/site/search?query=event%20hub&f%5B0%5D.Value=event%20hub&f%5B0%5D.Type=SearchText&ac=5)

下面提供了 .NET API 参考：

- [服务总线和事件中心 .NET API 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt419900.aspx)
- [事件处理程序主机 API 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt445521.aspx)

<!---HONumber=Mooncake_0321_2016-->