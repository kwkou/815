## 使用 Java 中的 EventProcessorHost 接收消息

EventProcessorHost 是一个 Java 类，通过从事件中心管理持久检查点和并行接收来简化从事件中心接收事件的过程。使用 EventProcessorHost，可跨多个接收方拆分事件，即使在不同节点中托管也是如此。此示例演示如何为单一接收方使用 EventProcessorHost。

###创建存储帐户

若要使用 EventProcessorHost，必须具有一个 [Azure 存储帐户][]：

1. 登录到 [Azure 经典门户][]，然后单击屏幕底部的“新建”。

2. 依次单击“数据服务”、“存储”、“快速创建”，然后为你的存储帐户键入一个名称。选择所需的区域，然后单击“创建存储帐户”。

    ![][11]

3. 单击新创建的存储帐户，然后单击“管理访问密钥”：

    ![][12]

    复制主访问密钥，供本教程后面使用。

###EventProcessor Host 创建一个 Java 项目

事件中心的 Java 客户端库可用于 [Maven 中央存储库](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22)中的 Maven 项目，并且可以使用 Maven 项目文件中的以下依赖项声明进行引用：

``` XML
<dependency>
	<groupId>com.microsoft.azure</groupId>
	<artifactId>azure-eventhubs</artifactId>
	<version>0.7.2</version>
</dependency>
```
 
对于不同类型的生成环境，你可以从 [Maven 中央存储库](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22)或 [GitHub 上的版本分发点](https://github.com/Azure/azure-event-hubs/releases)显式获取最新发布的 JAR 文件。

1. 对于下面的示例，请首先在你最喜欢的 Java 开发环境中为控制台/shell 应用程序创建一个新的 Maven 项目。该类将称为 ```ErrorNotificationHandler```。

	``` Java
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
	```

2. 使用以下代码创建名为 ```EventProcessor``` 的新类。

	```Java
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
	```

3. 使用以下代码创建一个名为 ```EventProcessorSample``` 的最终类。

	```Java
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
	```

4. 将以下字段替换为创建事件中心和存储帐户时所使用的值。

	``` Java
	final String namespaceName = "----ServiceBusNamespaceName-----";
	final String eventHubName = "----EventHubName-----";

	final String sasKeyName = "-----SharedAccessSignatureKeyName-----";
	final String sasKey = "---SharedAccessSignatureKey----";

	final String storageAccountName = "---StorageAccountName----"
	final String storageAccountKey = "---StorageAccountKey----";
	```

> [AZURE.NOTE] 本教程使用了一个 EventProcessorHost 实例。若要增加吞吐量，建议运行多个 EventProcessorHost 实例。在那些情况下，为了对接收到的事件进行负载均衡，各个不同实例会自动相互协调。如果希望多个接收方都各自处理全部事件，则必须使用 **ConsumerGroup** 概念。在从不同计算机中接收事件时，根据部署 EventProcessorHost 实例的计算机（或角色）来指定该实例的名称可能会很有用。

<!-- Links -->
[Event Hubs Overview]: /documentation/articles/event-hubs-overview/
[Azure 存储帐户]: /documentation/articles/storage-create-storage-account/
[Azure 经典门户]: http://manage.windowsazure.cn

<!-- Images -->
[11]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp2.png
[12]: ./media/service-bus-event-hubs-getstarted/create-eph-csharp3.png


<!---HONumber=Mooncake_0718_2016-->