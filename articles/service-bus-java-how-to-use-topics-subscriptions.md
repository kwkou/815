<properties linkid="dev-java-how-to-service-bus-topics" urlDisplayName="Service Bus Topics" pageTitle="如何使用 Service Bus 主题 (Java) - Azure" metaKeywords="Get started Azure Service Bus topics, Get Started Service Bus topics, Azure publish subscribe messaging, Azure messaging topics and subscriptions, Service Bus topic Java" description="了解如何在 Azure 中使用 Service Bus 主题和订阅。代码示例是针对 Java 应用程序编写的。" metaCanonical="" services="service-bus" documentationCenter="Java" title="如何使用 Service Bus 主题/订阅" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" scriptId="" videoId="" />

# 如何使用 Service Bus 主题/订阅

本指南演示如何使用 Service Bus 主题和
订阅。这些示例是采用 Java 编写的并且使用了 [Azure SDK for Java][Azure SDK for Java]。所涉及的方案包括**创建主题和订阅
**、**创建订阅筛选器**、**将消息发送到主题
**、**从订阅接收消息**以及
**删除主题和订阅**。

## 目录

-   [什么是 Service Bus 主题和订阅？][什么是 Service Bus 主题和订阅？]
-   [创建服务命名空间][创建服务命名空间]
-   [获得命名空间的默认管理凭据][获得命名空间的默认管理凭据]
-   [配置应用程序以使用 Service Bus][配置应用程序以使用 Service Bus]
-   [如何：创建主题][如何：创建主题]
-   [如何：创建订阅][如何：创建订阅]
-   [如何：将消息发送到主题][如何：将消息发送到主题]
-   [如何：从订阅接收消息][如何：从订阅接收消息]
-   [如何：处理应用程序崩溃和不可读消息][如何：处理应用程序崩溃和不可读消息]
-   [如何：删除主题和订阅][如何：删除主题和订阅]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-topics](../includes/howto-service-bus-topics.md)]

## <a name="bkmk_ConfigYourApp"> </a>配置应用程序以使用 Service Bus

将以下导入语句添加到 Java 文件顶部：

    // Include the following imports to use service bus APIs
    import com.microsoft.windowsazure.services.serviceBus.*;
    import com.microsoft.windowsazure.services.serviceBus.models.*;
    import com.microsoft.windowsazure.services.core.*;
    import javax.xml.datatype.*;

向您的生成路径添加 Azure Libraries for Java，并将其包含在您的项目部署程序集中。

## <a name="bkmk_HowToCreateTopic"> </a>如何创建主题

Service Bus 主题的管理操作可通过
**ServiceBusContract** 类执行。**ServiceBusContract** 对象是
使用封装了用于管理对象本身的令牌权限的适当
配置构造的，而 **ServiceBusContract** 类是
与 Azure 的单一通信点。

**ServiceBusService** 类提供了用于创建、枚举
和删除主题的方法。下面的示例演示了如何通过名为“HowToSample”的命名空间，使用 **ServiceBusService** 对象
创建名为“TestTopic”的主题：

    Configuration config = 
        ServiceBusConfiguration.configureWithWrapAuthentication(
          "HowToSample",
          "your_service_bus_owner",
          "your_service_bus_key",
          ".servicebus.windows.net",
          "-sb.accesscontrol.windows.net/WRAPv0.9");

    ServiceBusContract service = ServiceBusService.create(config);
    TopicInfo topicInfo = new TopicInfo("TestTopic");
    try  
    {
        CreateTopicResult result = service.createTopic(topicInfo);
    }
    catch (ServiceException e) {
        System.out.print("ServiceException encountered: ");
        System.out.println(e.getMessage());
        System.exit(-1);
    }

**TopicInfo** 上有一些支持调整主题属性的方法
（例如：将默认的“生存时间”值设置为
应用于发送到主题的消息）。以下示例演示了如何
创建最大大小为 5GB 且名为“TestTopic”的主题：

    long maxSizeInMegabytes = 5120;  
    TopicInfo topicInfo = new TopicInfo("TestTopic");  
    topicInfo.setMaxSizeInMegabytes(maxSizeInMegabytes); 
    CreateTopicResult result = service.createTopic(topicInfo);

请注意，您可以对 **ServiceBusContract** 对象使用 **listTopics** 方法
来检查具有指定名称的主题
是否已存在于服务命名空间中。

## <a name="bkmk_HowToCreateSubscrip"> </a>如何创建订阅

主题订阅也是使用 **ServiceBusService**
 类创建的。订阅已命名，并且具有一个限制传递到
订阅的虚拟队列的消息集的可选
筛选器。

### 创建具有默认 (MatchAll) 筛选器的订阅

**MatchAll** 筛选器是默认筛选器，在创建新订阅时
未指定筛选器的情况下使用。使用 **MatchAll**
 筛选器时，发布到主题的所有消息都将置于
订阅的虚拟队列中。以下示例创建了名为
“AllMessages”的订阅并使用了默认的 **MatchAll**
筛选器。

    SubscriptionInfo subInfo = new SubscriptionInfo("AllMessages");
    CreateSubscriptionResult result = 
        service.createSubscription("TestTopic", subInfo);

### 创建具有筛选器的订阅

还可以设置筛选器，以确定发送到主题的
哪些消息应该在特定主题订阅中显示。

订阅支持的最灵活的一种筛选器是
**SqlFilter**，它实现了一部分 SQL92 功能。SQL 筛选器将对发布
到主题的消息的属性进行操作。有关
可用于 SQL 筛选器的表达式的详细信息，
请参阅 SqlFilter.SqlExpression 语法。

下面的示例创建了一个名为“HighMessages”的订阅（带有只选择具有大于 3 的
**MessageNumber** 属性的
**SqlFilter**）：

    // Create a "HighMessages" filtered subscription  
    SubscriptionInfo subInfo = new SubscriptionInfo("HighMessages");
    CreateSubscriptionResult result = 
        service.createSubscription("TestTopic", subInfo);
    RuleInfo ruleInfo = new RuleInfo("myRuleGT3");
    ruleInfo = ruleInfo.withSqlExpressionFilter("MessageNumber > 3");
    CreateRuleResult ruleResult = 
        service.createRule("TestTopic", "HighMessages", ruleInfo);
    // Delete the default rule, otherwise the new rule won't be invoked.
    service.deleteRule("TestTopic", "HighMessages", "$Default");

类似地，下面的示例将使用一个 SqlFilter 创建一个名为
“LowMessages”的订阅
，该筛选器只选择具有
小于
或等于 3 的 MessageNumber 属性的消息：

    // Create a "LowMessages" filtered subscription
    SubscriptionInfo subInfo = new SubscriptionInfo("LowMessages");
    CreateSubscriptionResult result = 
        service.createSubscription("TestTopic", subInfo);
    RuleInfo ruleInfo = new RuleInfo("myRuleLE3");
    ruleInfo = ruleInfo.withSqlExpressionFilter("MessageNumber <= 3");
    CreateRuleResult ruleResult = 
        service.createRule("TestTopic", "LowMessages", ruleInfo);
    // Delete the default rule, otherwise the new rule won't be invoked.
    service.deleteRule("TestTopic", "LowMessages", "$Default");

现在，当消息发送到“TestTopic”时，始终会将它
传送到订阅了“AllMessages”主题
订阅的接收者，并选择性地传送到订阅了
“HighMessages”和“LowMessages”主题订阅的接收者（具体取决于
消息内容）。

## <a name="bkmk_HowToSendMsgs"> </a>如何将消息发送到主题

若要将消息发送到 Service Bus 主题，则您的应用程序将获得
 **ServiceBusContract** 对象。下面的代码演示了如何为
为我们之前在
“HowToSample”服务命名空间中创建的“TestTopic“主题发送消息：

    BrokeredMessage message = new BrokeredMessage("MyMessage");
    service.sendTopicMessage("TestTopic", message);

发送到 Service Bus 主题的消息是
**BrokeredMessage** 类的实例。**BrokeredMessage** 对象包含一组
标准方法（如 **setLabel** 和 **TimeToLive**）、一个
用来保留自定义应用程序特定属性的词典以及大量
任意的应用程序数据。应用程序可通过将任何可序列化
对象传递到
**BrokeredMessage** 的构造函数中来设置消息的正文，然后将使用适当的 **DataContractSerializer** 序列化
该对象。或者，也可以提供
**java.io.InputStream**。

以下示例演示了如何将五条测试消息发送到我们在上面的代码片段中获得的
“TestTopic”**MessageSender**。
请注意，每条消息的 **MessageNumber** 属性值在
循环迭代上有怎样的不同（这一点将确定哪些订阅
接收该消息）：

    for (int i=0; i<5; i++)  {
        // Create message, passing a string message for the body
        BrokeredMessage message = new BrokeredMessage("Test message " + i);
        // Set some additional custom app-specific property
        message.setProperty("MessageNumber", i);
        // Send message to the topic
        service.sendTopicMessage("TestTopic", message);
    }

Service Bus 主题支持最大为 256 MB 的消息（标头
最大为 64 MB，其中包括标准和自定义
应用程序属性）。一个主题中包含的消息数量不受
限制，但消息的总大小
受限制。此主题大小是在创建时定义的，上限
为 5 GB。

## <a name="bkmk_HowToReceiveMsgs"> </a>如何从订阅接收消息

从订阅接收消息的主要方法是使用
**ServiceBusContract** 对象。收到的消息可在两种
不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一项单步
操作，即当 Service Bus 接收
某条消息的读取请求时，它会将该消息标记为“已使用”并将其
返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式
，并且最适合应用程序允许出现故障时不处理
消息的情况。为了理解这一点，可以考虑
这样一种情形：使用方发出接收请求，但
在处理该请求前发生了崩溃。由于 Service Bus 会将
消息标记为“已使用”，因此当应用程序重新启动并重新开始
使用消息时，它会丢失在发生崩溃前使用的
消息。

在 **PeekLock** 模式下，接收变成了一个两阶段操作，从而
有可能支持无法容忍遗漏消息的应用
程序。当 Service Bus 收到请求时，它会查找下一条
要使用的消息，锁定该消息以防其他使用者接收，然后
将该消息返回到应用程序。应用程序完成
消息处理（或可靠地存储消息以供将来处理）后，它
将通过对收到的消息调用 **Delete**
 完成接收过程的第二个阶段。当 Service Bus 发现 **Delete** 调用时，它
会将消息标记为“已使用”并将其从主题中删除。

下面的示例演示了如何使用 **PeekLock** 模式
（非默认模式）接收和处理消息。下面的示例
执行一个循环并处理“HighMessages”订阅中的消息，然后在处理完所有消息后退出循环（或者，可将其设置为等待新消息）。

    try
    {
        ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
        opts.setReceiveMode(ReceiveMode.PEEK_LOCK);

        while(true)  { 
            ReceiveSubscriptionMessageResult  resultSubMsg = 
                service.receiveSubscriptionMessage("TestTopic", "HighMessages", opts);
            BrokeredMessage message = resultSubMsg.getValue();
            if (message != null && message.getMessageId() != null)
            {
                System.out.println("MessageID: " + message.getMessageId());    
                // Display the topic message.
                System.out.print("From topic: ");
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
                    message.getProperty("MessageNumber"));
                // Delete message.
                System.out.println("Deleting this message.");
                service.deleteMessage(message);
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

## <a name="bkmk_HowToHandleAppCrash"> </a>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助您轻松地从
应用程序错误或消息处理问题中恢复。如果
接收方应用程序出于某种原因无法处理消息，
它可以对收到的消息调用 **unlockMessage** 方法（而不
是 **deleteMessage** 方法）。这将导致 Service Bus
 解锁主题中的消息并使其能够
重新被同一个正在使用的应用程序
或其他正在使用的应用程序接收。

还存在与主题中已锁定消息关联的
超时，并且如果应用程序无法在锁定超时
到期之前处理消息（例如，如果应用程序崩溃）
，Service Bus 将自动解锁该消息
并使其可重新被接收。

如果在处理消息之后但在发出 **deleteMessage** 请求
之前应用程序发生崩溃，该消息将在
应用程序重新启动时重新传送给它。此情况通常
称作**至少处理一次**，即每条消息将
至少被处理一次，但在某些情况下，同一消息可能会被
重新传送。如果方案无法容忍重复处理，
则应用程序开发人员应向其应用程序
添加更多逻辑以处理重复消息传送。通常可使用消息
的 **getMessageId** 方法实现此操作，这在
多个传送尝试中保持不变。

## <a name="bkmk_HowToDeleteTopics"> </a>如何删除主题和订阅

删除主题和订阅的主要方法是使用
**ServiceBusContract** 对象。删除某个主题也会删除向该主题注册的
所有订阅。也可以单独删除订阅。

    // Delete subscriptions
    service.deleteSubscription("TestTopic", "AllMessages");
    service.deleteSubscription("TestTopic", "HighMessages");
    service.deleteSubscription("TestTopic", "LowMessages");

    // Delete a topic
    service.deleteTopic("TestTopic");

# <a name="bkmk_NextSteps"> </a> 后续步骤

现在，您已了解 Service Bus 队列的基础知识，请参阅 MSDN
 主题 [Service Bus 队列、主题和订阅][Service Bus 队列、主题和订阅]以了解详细信息。

  [Azure SDK for Java]: http://www.windowsazure.cn/zh-cn/develop/java/
  [什么是 Service Bus 主题和订阅？]: #what-are-service-bus-topics
  [创建服务命名空间]: #create-a-service-namespace
  [获得命名空间的默认管理凭据]: #obtain-default-credentials
  [配置应用程序以使用 Service Bus]: #bkmk_ConfigYourApp
  [如何：创建主题]: #bkmk_HowToCreateTopic
  [如何：创建订阅]: #bkmk_HowToCreateSubscrip
  [如何：将消息发送到主题]: #bkmk_HowToSendMsgs
  [如何：从订阅接收消息]: #bkmk_HowToReceiveMsgs
  [如何：处理应用程序崩溃和不可读消息]: #bkmk_HowToHandleAppCrash
  [如何：删除主题和订阅]: #bkmk_HowToDeleteTopics
  [后续步骤]: #bkmk_NextSteps
  [Service Bus 队列、主题和订阅]: http://msdn.microsoft.com/library/windowsazure/hh367516.aspx
