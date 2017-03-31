<properties
    pageTitle="Azure 事件中心 .NET Standard API 概述 | Azure"
    description=".NET Standard API 概述"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="a173f8e4-556c-42b8-b856-838189f7e636"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/02/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub" />  


# 事件中心 .NET Standard API 概述
本文汇总了一些重要的事件中心 .NET Standard 客户端 API。目前，有两个 .NET Standard 客户端库：

* [Microsoft.Azure.EventHubs](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs)
  *  此库提供所有基本运行时操作。
* [Microsoft.Azure.EventHubs.Processor](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor)
  * 此库添加了一项附加功能，该功能可用于跟踪已处理的事件，是从事件中心读取事件的最简单方法。

## 事件中心客户端
[**EventHubClient**](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient) 是发送事件、创建接收器以及获取运行时信息时使用的主对象。此客户端链接到特定的事件中心，并创建与事件中心终结点的新连接。

### 创建事件中心
[**EventHubClient**](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient) 对象从连接字符串创建。以下示例演示实例化新客户端的最简单方法：

    var eventHubClient = EventHubClient.CreateFromConnectionString("{Event Hub connection string}");

若要以编程方式编辑连接字符串，可以使用 [**EventHubsConnectionStringBuilder**](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubsconnectionstringbuilder) 类，并将连接字符串作为参数传递给 [**EventHubClient.CreateFromConnectionString**](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient#Microsoft_Azure_EventHubs_EventHubClient_CreateFromConnectionString_System_String_)。

    var connectionStringBuilder = new EventHubsConnectionStringBuilder("{Event Hub connection string}")
    {
        EntityPath = EhEntityPath
    };

    var eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

### 发送事件
若要将事件发送到事件中心，请使用 [**EventData**](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventdata) 类。主体必须是 `byte` 数组，或 `byte` 数组段。

    // Create a new EventData object by encoding a string as a byte array
    var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));
    // Set user properties if needed
    data.Properties.Add("Type", "Informational");
    // Send single message async
    await eventHubClient.SendAsync(data);

### 接收事件
从事件中心接收事件的建议方法是使用 [**EventProcessorHost**](#event-processor-host-apis)，它提供相关功能来自动跟踪偏移量和分区信息。但是，在某些情况下，可能需要利用核心事件中心库的灵活性来接收事件。

#### 创建接收器
接收器绑定到特定分区，因此，为了接收事件中心内的所有事件，需要创建多个实例。一般来说，好的做法是以编程方式获取分区信息，而不是对分区 ID 进行硬编码。为此，可以使用 [**GetRuntimeInformationAsync**](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.eventhubclient#Microsoft_Azure_EventHubs_EventHubClient_GetRuntimeInformationAsync) 方法。

    // Create a list to keep track of the receivers
    var receivers = new List<PartitionReceiver>();
    // Use the eventHubClient created above to get the runtime information
    var runTimeInformation = await eventHubClient.GetRuntimeInformationAsync();
    // Loop over the resulting partition ids
    foreach (var partitionId in runTimeInformation.PartitionIds)
    {
        // Create the receiver
        var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, PartitionReceiver.EndOfStream);
        // Add the receiver to the list
        receivers.Add(receiver);
    }

由于事件永远不会从事件中心删除（而只会过期），因此需要指定正确的起始点。以下示例介绍可能的组合。

    // partitionId is assumed to come from GetRuntimeInformationAsync()

    // Using the constant 'PartitionReceiver.EndOfStream' will only receive all messages from this point forward.
    var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, PartitionReceiver.EndOfStream);

    // All messages available
    var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, "-1");

    // From one day ago
    var receiver = eventHubClient.CreateReceiver(PartitionReceiver.DefaultConsumerGroupName, partitionId, DateTime.Now.AddDays(-1));

#### 使用事件

    // Receive a maximum of 100 messages in this call to ReceiveAsync
    var ehEvents = await receiver.ReceiveAsync(100);
    // ReceiveAsync can return null if there are no messages
    if (ehEvents != null)
    {
        // Since ReceiveAsync can return more than a single event you will need a loop to process
        foreach (var ehEvent in ehEvents)
        {
            // Decode the byte array segment
            var message = UnicodeEncoding.UTF8.GetString(ehEvent.Body.Array);
            // Load the custom property that we set in the send example
            var customType = ehEvent.Properties["Type"];
            // Implement processing logic here
        }
    }        

## <a id="event-processor-host-apis"></a> 事件处理程序主机 API
这些 API 通过在可用工作进程之间分布分区，为可能变为不可用的工作进程提供复原能力。

    // Checkpointing is done within the SimpleEventProcessor and on a per-consumerGroup per-partition basis, workers resume from where they last left off.

    // Read these connection strings from a secure location
    var ehConnectionString = "{Event Hubs connection string}";
    var ehEntityPath = "{Event Hub path/name}";
    var storageConnectionString = "{Storage connection string}";
    var storageContainerName = "{Storage account container name}";

    var eventProcessorHost = new EventProcessorHost(
        ehEntityPath,
        PartitionReceiver.DefaultConsumerGroupName,
        ehConnectionString,
        storageConnectionString,
        storageContainerName);

    // Start/register an EventProcessorHost
    await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

    // Disposes of the Event Processor Host
    await eventProcessorHost.UnregisterEventProcessorAsync();

以下是 [IEventProcessor](http://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) 的示例实现。

    public class SimpleEventProcessor : IEventProcessor
    {
        public Task CloseAsync(PartitionContext context, CloseReason reason)
        {
            Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
            return Task.CompletedTask;
        }

        public Task OpenAsync(PartitionContext context)
        {
            Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
            return Task.CompletedTask;
        }

        public Task ProcessErrorAsync(PartitionContext context, Exception error)
        {
            Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
            return Task.CompletedTask;
        }

        public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            foreach (var eventData in messages)
            {
                var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
                Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
            }

            return context.CheckpointAsync();
        }
    }

## 后续步骤
若要了解有关事件中心方案的详细信息，请访问以下链接：

* [什么是 Azure 事件中心？](/documentation/articles/event-hubs-what-is-event-hubs/)
* [可用的事件中心 API](/documentation/articles/event-hubs-api-overview/)

下面提供了 .NET API 参考：

* [Microsoft.Azure.EventHubs](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs)
* [Microsoft.Azure.EventHubs.Processor](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.processor)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:new article about illustrate the dotnet framework api in Azure event hubs-->