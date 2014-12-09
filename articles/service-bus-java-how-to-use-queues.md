<properties linkid="dev-java-how-to-service-bus-queues" urlDisplayName="Service Bus Queues" pageTitle="How to use Service Bus queues (Java) - Azure" metaKeywords="Azure Service Bus queues, Azure queues, Azure messaging, Azure queues Java" description="Learn how to use Service Bus queues in Azure. Code samples written in Java." metaCanonical="" services="service-bus" documentationCenter="Java" title="How to Use Service Bus Queues" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何使用 Service Bus 队列

本指南演示如何使用 Service Bus 队列。这些示例用 Java 编写并使用 [Azure SDK for Java][Azure SDK for Java]。所涉及的任务包括**创建队列**、**发送和接收消息**和**删除队列**。

## 目录

-   [什么是 Service Bus 队列？][什么是 Service Bus 队列？]
-   [创建服务命名空间][创建服务命名空间]
-   [获得命名空间的默认管理凭据][获得命名空间的默认管理凭据]
-   [配置应用程序以使用 Service Bus][配置应用程序以使用 Service Bus]
-   [如何：创建安全令牌提供程序][如何：创建安全令牌提供程序]
-   [如何：创建队列][如何：创建安全令牌提供程序]
-   [如何：向队列发送消息][如何：向队列发送消息]
-   [如何：从队列接收消息][如何：从队列接收消息]
-   [如何：处理应用程序崩溃和不可读消息][如何：处理应用程序崩溃和不可读消息]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## <a name="bkmk_ConfigApp"> </a>配置应用程序以使用 Service Bus

将以下导入语句添加到 Java 文件顶部：

    // Include the following imports to use service bus APIs
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*; 
    import com.microsoft.windowsazure.services.core.*; 
    import javax.xml.datatype.*;

## <a name="bkmk_HowToCreateQueue"> </a>如何创建队列

Service Bus 队列的管理操作可通过**ServiceBusContract** 类执行。**ServiceBusContract** 对象是使用封装了用于管理对象本身的令牌权限的适当配置构造的，而 **ServiceBusContract** 类是与 Azure 的单一通信点。

**ServiceBusService** 类提供了创建、枚举和删除队列的方法。下面的示例演示了如何通过名为“HowToSample”的命名空间，使用 **ServiceBusService** 对象创建名为“TestQueue”的队列：

    Configuration config = 
        ServiceBusConfiguration.configureWithWrapAuthentication(
          "HowToSample",
          "your_service_bus_owner",
          "your_service_bus_key",
          ".servicebus.windows.net",
          "-sb.accesscontrol.windows.net/WRAPv0.9");

    ServiceBusContract service = ServiceBusService.create(config);
    QueueInfo queueInfo = new QueueInfo("TestQueue");
    try
    {     
        CreateQueueResult result = service.createQueue(queueInfo);
    }
    catch (ServiceException e)
    {
        System.out.print("ServiceException encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }

QueueInfo 上有一些支持调整队列属性的方法（例如：将默认的“生存时间”值设置为应用于发送到队列的消息）。以下示例演示了如何创建最大大小为 5GB 且名为“TestQueue”的队列：

    long maxSizeInMegabytes = 5120;
    QueueInfo queueInfo = new QueueInfo("TestQueue");
    queueInfo.setMaxSizeInMegabytes(maxSizeInMegabytes); 
    CreateQueueResult result = service.createQueue(queueInfo);

请注意，你可对 **ServiceBusContract** 对象使用 **listQueues** 方法来检查具有指定名称的队列在某个服务命名空间中是否已存在。

## <a name="bkmk_HowToSendMsgs"> </a>如何向队列发送消息

若要将消息发送到 Service Bus 队列，你的应用程序将获得 **ServiceBusContract** 对象。下面的代码演示了如何为我们之前在“HowToSample”服务命名空间中创建的“TestQueue”队列发送消息：

    try
    {
        BrokeredMessage message = new BrokeredMessage("MyMessage");
        service.sendQueueMessage("TestQueue", message);
    }
    catch (ServiceException e) 
    {
        System.out.print("ServiceException encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }

发送到（和接收自）Service Bus 队列的消息是 **BrokeredMessage** 类的实例。**BrokeredMessage** 对象包含一组
标准方法（如 **getLabel**、**getTimeToLive**、**setLabel** 和 **setTimeToLive**）、一个用来保存自定义应用程序特定属性的词典以及一组任意的应用程序数据。应用程序可通过将任何可序列化对象传入到 **BrokeredMessage** 的构造函数中
来设置消息的正文，然后将使用适当的序列化程度来序列化对象。或者，也可以提供 **java.IO.InputStream**。

以下示例演示了如何将五条测试消息发送到我们在上面的代码段中获得的“TestQueue”**MessageSender**：

    for (int i=0; i<5; i++)
    {
         // Create message, passing a string message for the body.
         BrokeredMessage message = new BrokeredMessage("Test message " + i);
         // Set an additional app-specific property.
         message.setProperty("MyProperty", i); 
         // Send message to the queue
         service.sendQueueMessage("TestQueue", message);
    }

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## <a name="bkmk_HowToReceiveMsgs"> </a>如何从队列接收消息

从队列接收消息的主要方法是使用 **ServiceBusContract** 对象。收到的消息可在两种不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一个单击操作 - 即，当 Service Bus 接收队列中的消息读取请求时，它会将消息标记为“将使用”并将其返回应用程序。**ReceiveAndDelete** 模式（默认模式）是最简单的模式，最适合应用程序可容忍出现故障时不处理消息的情景。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

在 **PeekLock** 模式下，接收变成了一个两阶段操作，从而有可能支持无法容忍遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 **Delete** 完成接收过程的第二个阶段。当 Service Bus 发现 **Delete** 调用时，它会将消息标记为“已使用”并将其从队列中删除。

下面的示例演示了如何使用 **PeekLock** 模式（非默认模式）接收和处理消息。下面的示例将执行无限循环并在消息达到我们的“TestQueue”后进行处理：

        try
    {
        ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
        opts.setReceiveMode(ReceiveMode.PEEK_LOCK);

        while(true)  { 
             ReceiveQueueMessageResult resultQM = 
                    service.receiveQueueMessage("TestQueue", opts);
            BrokeredMessage message = resultQM.getValue();
            if (message != null && message.getMessageId() != null)
            {
                System.out.println("MessageID: " + message.getMessageId());    
                // Display the queue message.
                System.out.print("From queue: ");
                byte[] b = new byte[200];
                String s = null;
                int numRead = message.getBody().read(b);
                while (-1 != numRead)
                {
                    s = new String(b);
                    s = s.trim();
                    System.out.print(s);
                    numRead = message.getBody().read(b);
                }
                System.out.println();
                System.out.println("Custom Property: " + 
                    message.getProperty("MyProperty"));
                // Remove message from queue.
                System.out.println("Deleting this message.");
                //service.deleteMessage(message);
            }  
            else  
            {        
                System.out.println("Finishing up - no more messages.");        
                break; 
                // Added to handle no more messages.
                // Could instead wait for more messages to be added.
            }
        }
    }
    catch (ServiceException e) {
        System.out.print("ServiceException encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }
    catch (Exception e) {
        System.out.print("Generic exception encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }   

## <a name="bkmk_HowToHandleAppCrashes"> </a>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序出于某种原因无法处理消息，
它可以对收到的消息调用 **unlockMessage** 方法（而不是 **deleteMessage** 方法）。这将导致 Service Bus解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃）
，Service Bus 将自动解锁该消息并使它可再次被接收。

如果在处理消息之后但在发出 **deleteMessage** 请求之前应用程序发生崩溃，该消息将在应用程序重新启动时重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。通常可使用消息的 **getMessageId** 方法实现此操作，这在多个传送尝试中保持不变。

## <a name="bkmk_NextSteps"> </a> 后续步骤

现在，你已了解 Service Bus 队列的基础知识，请参阅 MSDN主题[队列、主题和订阅][队列、主题和订阅]以获取更多信息。

  [Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
  [什么是 Service Bus 队列？]: #what-are-service-bus-queues
  [创建服务命名空间]: #create-a-service-namespace
  [获得命名空间的默认管理凭据]: #obtain-default-credentials
  [配置应用程序以使用 Service Bus]: #bkmk_ConfigApp
  [如何：创建安全令牌提供程序]: #bkmk_HowToCreateQueue
  [如何：向队列发送消息]: #bkmk_HowToSendMsgs
  [如何：从队列接收消息]: #bkmk_HowToReceiveMsgs
  [如何：处理应用程序崩溃和不可读消息]: #bkmk_HowToHandleAppCrashes
  [后续步骤]: #bkmk_NextSteps
  [howto-service-bus-queues]: ../includes/howto-service-bus-queues.md
  [队列、主题和订阅]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367516.aspx
