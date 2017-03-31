<properties
    pageTitle="使用 Java 从 Azure 事件中心接收事件 | Azure"
    description="使用 Java 从事件中心接收入门"
    services="event-hubs"
    documentationcenter=""
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="38e3be53-251c-488f-a856-9a500f41b6ca"
    ms.service="event-hubs"
    ms.workload="core"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/30/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub;sethm" />  


# 使用 Java 从 Azure 事件中心接收事件

## 介绍
事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析连接的设备和应用程序所产生的海量数据。数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][Event Hubs overview]。

本教程演示如何使用以 Java 编写的控制台应用程序从事件中心接收事件。

若要完成本教程，你需要以下各项：

* Java 开发环境。对于本教程，我们将采用 Eclipse。
* 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。

## 使用 Java 中的 EventProcessorHost 接收消息

EventProcessorHost 是一个 Java 类，通过从事件中心管理持久检查点和并行接收来简化从事件中心接收事件的过程。使用 EventProcessorHost，可跨多个接收方拆分事件，即使在不同节点中托管也是如此。此示例演示如何为单一接收方使用 EventProcessorHost。

### 创建存储帐户
若要使用 EventProcessorHost，必须具有一个 [Azure 存储帐户][Azure Storage account]：

1. 登录 [Azure 经典管理门户][Azure Classic Management Portal]，然后单击屏幕底部的“新建”。

2. 依次单击“数据服务”、“存储”、“快速创建”，然后为你的存储帐户键入一个名称。选择所需的区域，然后单击“创建存储帐户”。
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage2.png)  


3. 单击新创建的存储帐户，然后单击“管理访问密钥”：
   
    ![](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-storage3.png)  


    复制主访问密钥，供本教程后面使用。

### EventProcessor Host 创建一个 Java 项目
事件中心的 Java 客户端库可用于 [Maven 中央存储库][Maven Package]中的 Maven 项目，并且可以使用 Maven 项目文件中的以下依赖项声明进行引用：

    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-eventhubs</artifactId>
        <version>{VERSION}</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-eventhubs-eph</artifactId>
        <version>{VERSION}</version>
    </dependency>

对于不同类型的生成环境，你可以从 [Maven 中央存储库][Maven Package]或 [GitHub 上的版本分发点](https://github.com/Azure/azure-event-hubs/releases)显式获取最新发布的 JAR 文件。

1. 对于下面的示例，请首先在你最喜欢的 Java 开发环境中为控制台/shell 应用程序创建一个新的 Maven 项目。该类将称为 ```ErrorNotificationHandler```。

        import java.util.function.Consumer;
        import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   
        public class ErrorNotificationHandler implements Consumer<ExceptionReceivedEventArgs>
        {
            @Override
            public void accept(ExceptionReceivedEventArgs t)
            {
                System.out.println("SAMPLE: Host " + t.getHostname() + " received general error notification during " + t.getAction() + ": " + t.getException().toString());
            }
        }

2. 使用以下代码创建名为 ```EventProcessor``` 的新类。

        import com.microsoft.azure.eventhubs.EventData;
        import com.microsoft.azure.eventprocessorhost.CloseReason;
        import com.microsoft.azure.eventprocessorhost.IEventProcessor;
        import com.microsoft.azure.eventprocessorhost.PartitionContext;
   
        public class EventProcessor implements IEventProcessor
        {
            private int checkpointBatchingCount = 0;
   
            @Override
            public void onOpen(PartitionContext context) throws Exception
            {
                System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is opening");
            }
   
            @Override
            public void onClose(PartitionContext context, CloseReason reason) throws Exception
            {
                System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is closing for reason " + reason.toString());
            }
   
            @Override
            public void onError(PartitionContext context, Throwable error)
            {
                System.out.println("SAMPLE: Partition " + context.getPartitionId() + " onError: " + error.toString());
            }
   
            @Override
            public void onEvents(PartitionContext context, Iterable<EventData> messages) throws Exception
            {
                System.out.println("SAMPLE: Partition " + context.getPartitionId() + " got message batch");
                int messageCount = 0;
                for (EventData data : messages)
                {
                    System.out.println("SAMPLE (" + context.getPartitionId() + "," + data.getSystemProperties().getOffset() + "," +
                            data.getSystemProperties().getSequenceNumber() + "): " + new String(data.getBody(), "UTF8"));
                    messageCount++;
   
                    this.checkpointBatchingCount++;
                    if ((checkpointBatchingCount % 5) == 0)
                    {
                        System.out.println("SAMPLE: Partition " + context.getPartitionId() + " checkpointing at " +
                            data.getSystemProperties().getOffset() + "," + data.getSystemProperties().getSequenceNumber());
                        context.checkpoint(data);
                    }
                }
                System.out.println("SAMPLE: Partition " + context.getPartitionId() + " batch size was " + messageCount + " for host " + context.getOwner());
            }
        }

3. 使用以下代码创建一个名为 ```EventProcessorSample``` 的 final 类。

        import com.microsoft.azure.eventprocessorhost.*;
        import com.microsoft.azure.servicebus.ConnectionStringBuilder;
        import com.microsoft.azure.eventhubs.EventData;
   
        public class EventProcessorSample
        {
            public static void main(String args[])
            {
                final String consumerGroupName = "$Default";
                final String namespaceName = "----ServiceBusNamespaceName-----";
                final String eventHubName = "----EventHubName-----";
                final String sasKeyName = "-----SharedAccessSignatureKeyName-----";
                final String sasKey = "---SharedAccessSignatureKey----";
   
                final String storageAccountName = "---StorageAccountName----";
                final String storageAccountKey = "---StorageAccountKey----";
                final String storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=" + storageAccountName + ";AccountKey=" + storageAccountKey;
   
                ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder(namespaceName, eventHubName, sasKeyName, sasKey);
   
                EventProcessorHost host = new EventProcessorHost(eventHubName, consumerGroupName, eventHubConnectionString.toString(), storageConnectionString);
   
                System.out.println("Registering host named " + host.getHostName());
                EventProcessorOptions options = new EventProcessorOptions();
                options.setExceptionNotification(new ErrorNotificationHandler());
                try
                {
                    host.registerEventProcessor(EventProcessor.class, options).get();
                }
                catch (Exception e)
                {
                    System.out.print("Failure while registering: ");
                    if (e instanceof ExecutionException)
                    {
                        Throwable inner = e.getCause();
                        System.out.println(inner.toString());
                    }
                    else
                    {
                        System.out.println(e.toString());
                    }
                }
   
                System.out.println("Press enter to stop");
                try
                {
                    System.in.read();
                    host.unregisterEventProcessor();
   
                    System.out.println("Calling forceExecutorShutdown");
                    EventProcessorHost.forceExecutorShutdown(120);
                }
                catch(Exception e)
                {
                    System.out.println(e.toString());
                    e.printStackTrace();
                }
   
                System.out.println("End of sample");
            }
        }

4. 将以下字段替换为创建事件中心和存储帐户时所使用的值。

        final String namespaceName = "----ServiceBusNamespaceName-----";
        final String eventHubName = "----EventHubName-----";
   
        final String sasKeyName = "-----SharedAccessSignatureKeyName-----";
        final String sasKey = "---SharedAccessSignatureKey----";
   
        final String storageAccountName = "---StorageAccountName----"
        final String storageAccountKey = "---StorageAccountKey----";

> [AZURE.NOTE]
本教程使用了一个 EventProcessorHost 实例。若要增加吞吐量，建议运行多个 EventProcessorHost 实例。在那些情况下，为了对接收到的事件进行负载均衡，各个不同实例会自动相互协调。如果希望多个接收方都各自处理*全部*事件，则必须使用 **ConsumerGroup** 概念。在从不同计算机中接收事件时，根据部署 EventProcessorHost 实例的计算机（或角色）来指定该实例的名称可能会很有用。
> 
> 

<!-- Links -->

[Event Hubs overview]: /documentation/articles/event-hubs-overview/
[Azure Storage account]: /documentation/articles/storage-create-storage-account/
[Azure Classic Management Portal]: http://manage.windowsazure.cn
[Maven Package]: https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22

<!-- Images -->
[11]: ./media/service-bus-event-hubs-get-started-receive-ephjava/create-eph-csharp2.png
[12]: ./media/service-bus-event-hubs-get-started-receive-ephjava/create-eph-csharp3.png

## 后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!-- Links -->

[Event Hubs overview]: /documentation/articles/event-hubs-overview/

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:new article about how to receiving event from Azure event hubs with Java-->