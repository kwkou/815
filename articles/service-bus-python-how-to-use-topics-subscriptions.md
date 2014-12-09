<properties linkid="develop-python-service-bus-topics" urlDisplayName="Service Bus Topics" pageTitle="How to use Service Bus topics (Python) - Azure" metaKeywords="Get started Azure Service Bus topics publising subscribe messaging Python" description="Learn how to use Service Bus topics and subscriptions in Azure. Code samples are written for Python applications." metaCanonical="" services="service-bus" documentationCenter="Python" title="How to Use Service Bus Topics/Subscriptions" authors="" solutions="" manager="" editor="" />

# 如何使用 Service Bus 主题/订阅

本指南将演示如何从 Python 应用程序使用 Service Bus 主题和订阅。涉及的应用场景包括**创建主题和订阅、创建订阅筛选器、将消息发送到主题**、**从订阅接收消息**以及**删除主题和订阅**。有关主题和订阅的详细信息，请参阅[后续步骤][后续步骤]一节。

## 目录

-   [什么是 Service Bus 主题和订阅？][什么是 Service Bus 主题和订阅？]
-   [创建服务命名空间][创建服务命名空间]
-   [获得命名空间的默认管理凭据][获得命名空间的默认管理凭据]
-   [如何：创建主题][如何：创建主题]
-   [如何：创建订阅][如何：创建订阅]
-   [如何：将消息发送到主题][如何：将消息发送到主题]
-   [如何：从订阅接收消息][如何：从订阅接收消息]
-   [如何：处理应用程序崩溃和不可读消息][如何：处理应用程序崩溃和不可读消息]
-   [如何：删除主题和订阅][如何：删除主题和订阅]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-topics](../includes/howto-service-bus-topics.md)]

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南][Python 安装指南]。

## <a name="How_to_Create_a_Topic"></a>如何创建主题

可以通过 **ServiceBusService** 对象处理主题。将以下代码添加到任何 Python 文件的顶部附近，你希望在其中以编程方式访问 Azure Service Bus：

    from azure.servicebus import *

以下代码将创建 **ServiceBusService** 对象。使用实际命名空间、密钥和颁发者替换“mynamespace”、“mykey”和“myissuer”。

    bus_service = ServiceBusService(service_namespace='mynamespace', account_key='mykey', issuer='myissuer')

    bus_service.create_topic('mytopic')

**create\_topic** 还支持其他选项，以允许你重写默认主题设置，例如消息生存时间或最大主题大小。下面的示例演示如何将最大主题大小设置为 5 GB，将生存时间设置为 1 分钟：

    topic_options = Topic()
    topic_options.max_size_in_megabytes = '5120'
    topic_options.default_message_time_to_live = 'PT1M'

    bus_service.create_topic('mytopic', topic_options)

## <a name="How_to_Create_Subscriptions"></a>如何创建订阅

主题订阅也是使用 **ServiceBusService** 对象创建的。订阅已命名，并且具有一个限制传递到订阅的虚拟队列的消息集的可选筛选器。

**注意**：订阅是永久性的，并且除非删除它或删除与之相关的主题，否则订阅将一直存在。

### 创建具有默认 (MatchAll) 筛选器的订阅

**MatchAll** 筛选器是默认筛选器，在创建新订阅时未指定筛选器的情况下使用。使用 **MatchAll** 筛选器时，发布到主题的所有消息都将置于订阅的虚拟队列中。以下示例创建了名为“AllMessages”的订阅并使用了默认的 **MatchAll**筛选器。

    bus_service.create_subscription('mytopic', 'AllMessages')

### 创建具有筛选器的订阅

还可以设置筛选器，以确定发送到主题的哪些消息应该在特定主题订阅中显示。

订阅支持的最灵活的一种筛选器是**SqlFilter**，它实现了一部分 SQL92 功能。SQL 筛选器将对发布到主题的消息的属性进行操作。有关
可用于 SQL 筛选器的表达式的更多详细信息，请参阅 SqlFilter.SqlExpression 语法。

可以使用 **ServiceBusService** 对象的 **create\_rule** 方法向订阅中添加筛选器。此方法允许你向现有订阅中添加新筛选器。

**注意**：由于默认筛选器会自动应用到所有新订阅，因此，你必须首先删除默认筛选器，否则**MatchAll** 会替代你可能指定的任何其他筛选器。可以使用 **ServiceBusService** 对象的 **delete\_rule** 方法删除默认规则。

下面的示例创建了一个名为“HighMessages”的订阅（带有只选择具有大于 3 的**messagenumber** 属性的 **SqlFilter**）：

    bus_service.create_subscription('mytopic', 'HighMessages')

    rule = Rule()
    rule.filter_type = 'SqlFilter'
    rule.filter_expression = 'messagenumber > 3'

    bus_service.create_rule('mytopic', 'HighMessages', 'HighMessageFilter', rule)
    bus_service.delete_rule('mytopic', 'HighMessages', DEFAULT_RULE_NAME)

类似地，下面的示例创建一个名为 “LowMessages”的订阅，其 **SqlFilter** 只选择 **messagenumber** 属性小于或等于 3 的消息：

    bus_service.create_subscription('mytopic', 'LowMessages')

    rule = Rule()
    rule.filter_type = 'SqlFilter'
    rule.filter_expression = 'messagenumber <= 3'

    bus_service.create_rule('mytopic', 'LowMessages', 'LowMessageFilter', rule)
    bus_service.delete_rule('mytopic', 'LowMessages', DEFAULT_RULE_NAME)

现在，在将消息发送到“mytopic”时，始终会将它传送到订阅了“AllMessages”主题订阅的接收者，并选择性地传送到订阅了“HighMessages”和 “LowMessages”主题订阅的接收者（具体取决于消息内容）。

## <a name="How_to_Send_Messages_to_a_Topic"></a>如何将消息发送到主题

若要将消息发送到 Service Bus 主题，你的应用程序必须使用**ServiceBusService** 对象的 **send\_topic\_message** 方法。

下面的示例演示如何向“mytopic”发送五条测试消息。请注意，每条消息的 **messagenumber** 属性值因循环迭代而异（这将确定哪些订阅接收它）：

    for i in range(5):
        msg = Message('Msg ' + str(i), custom_properties={'messagenumber':i})
        bus_service.send_topic_message('mytopic', msg)

Service Bus 主题支持最大为 256 MB 的消息（标头最大为 64 MB，其中包括标准和自定义应用程序属性）。一个主题中包含的消息数量不受限制，但消息的总大小受限制。此主题大小是在创建时定义的，上限为 5 GB。

## <a name="How_to_Receive_Messages_from_a_Subscription"></a>如何从订阅接收消息

对 **ServiceBusService** 对象使用**receive\_subscription\_message** 方法可从订阅接收消息：

    msg = bus_service.receive_subscription_message('mytopic', 'LowMessages')
    print(msg.body)

在读取消息后将从订阅中删除它们；不过，通过将可选参数 **peek\_lock** 设置为 **True**，你可以读取（扫视）并锁定消息，以避免将其从订阅中删除。

在接收过程中读取并删除消息的默认行为是最简单的模式，并且最适合在发生故障时应用程序可以容忍不处理消息的情况。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

如果将 **peek\_lock** 参数设置为 **True**，则接收将变成一个两阶段操作，这样就可以支持无法容忍遗漏消息的应用程序。当 Service Bus 接收请求时，它会查找要耗用的下一条消息，将其锁定以防止其他使用者接收它，然后将它返回到应用程序。在应用程序完成处理该消息（或将其可靠存储以备将来处理）后，它通过对 **Message** 对象调用 **delete** 方法完成接收过程的第二阶段。**delete** 方法会将消息标记为已使用，并从订阅中删除它。

    msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=True)
    print(msg.body)

    msg.delete()

## <a name="How_to_Handle_Application_Crashes_and_Unreadable_Messages"></a>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，则它可以对**Message** 对象调用 **unlock** 方法。这会导致 Service Bus 解锁订阅中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与订阅中的锁定消息关联的超时，如果应用程序未能在锁定超时过期前处理消息（例如，应用程序崩溃），Service Bus 将自动解锁该消息并使之重新可供接收。

如果应用程序在处理消息之后，但在调用 **delete** 方法之前崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的**MessageId** 属性来实现，该属性在多次传送尝试中保持不变。

## <a name="How_to_Delete_Topics_and_Subscriptions"></a>如何删除主题和订阅

主题和订阅具有持久性，必须通过Azure 管理门户或以编程方式显式删除。下面的示例演示如何删除名为“mytopic”的主题：

    bus_service.delete_topic('mytopic')

删除某个主题也会删除向该主题注册的所有订阅。也可以单独删除订阅。下面的代码演示如何从“mytopic”主题中删除名为“HighMessages”的订阅：

    bus_service.delete_subscription('mytopic', 'HighMessages')

## <a name="Next_Steps"></a> 后续步骤

现在，你已了解有关 Service Bus 主题的基础知识，单击下面的链接可了解更多信息。

-   查看 MSDN 参考：[队列、主题和订阅][队列、主题和订阅]。
-   [SqlFilter][SqlFilter] 的 API 参考。

  [后续步骤]: #Next_Steps
  [什么是 Service Bus 主题和订阅？]: #what-are-service-bus-topics
  [创建服务命名空间]: #create-a-service-namespace
  [获得命名空间的默认管理凭据]: #obtain-default-credentials
  [如何：创建主题]: #How_to_Create_a_Topic
  [如何：创建订阅]: #How_to_Create_Subscriptions
  [如何：将消息发送到主题]: #How_to_Send_Messages_to_a_Topic
  [如何：从订阅接收消息]: #How_to_Receive_Messages_from_a_Subscription
  [如何：处理应用程序崩溃和不可读消息]: #How_to_Handle_Application_Crashes_and_Unreadable_Messages
  [如何：删除主题和订阅]: #How_to_Delete_Topics_and_Subscriptions
  [howto-service-bus-topics]: ../includes/howto-service-bus-topics.md
  [Python 安装指南]: ../python-how-to-install/
  [队列、主题和订阅]: http://msdn.microsoft.com/zh-cn/library/hh367516.aspx
  [SqlFilter]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.servicebus.messaging.sqlfilter.aspx
