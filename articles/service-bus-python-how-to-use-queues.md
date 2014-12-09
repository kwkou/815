<properties linkid="develop-python-service-bus-queues" urlDisplayName="Service Bus Queues" pageTitle="How to use Service Bus queues (Python) - Azure" metaKeywords="Azure Service Bus queues, Azure queues, Azure messaging, Azure queues Python" description="Learn how to use Service Bus queues in Azure. Code samples written in Python." metaCanonical="" services="service-bus" documentationCenter="Python" title="How to Use Service Bus Queues" authors="" solutions="" manager="" editor="" />

# 如何使用 Service Bus 队列

本指南演示如何使用 Service Bus 队列。相关示例是使用 Python 编写的，并使用 Python Azure 模块。所涉及的任务包括**创建队列、发送和接收消息**和**删除队列**。有关队列的详细信息，请参阅[后续步骤][后续步骤]一节。

## 目录

-   [什么是 Service Bus 队列？][什么是 Service Bus 队列？]
-   [创建服务命名空间][创建服务命名空间]
-   [获得命名空间的默认管理凭据][获得命名空间的默认管理凭据]
-   [如何：创建队列][如何：创建队列]
-   [如何：向队列发送消息][如何：向队列发送消息]
-   [如何：从队列接收消息][如何：从队列接收消息]
-   [如何：处理应用程序崩溃和不可读消息][如何：处理应用程序崩溃和不可读消息]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南][Python 安装指南]。

## <a name="create-queue"> </a>如何创建队列

可以通过 **ServiceBusService** 对象处理队列。将以下代码添加到任何 Python 文件的顶部附近，你希望在其中以编程方式访问 Azure Service Bus：

    from azure.servicebus import *

以下代码将创建 **ServiceBusService** 对象。使用实际命名空间、密钥和颁发者替换“mynamespace”、“mykey”和“myissuer”。

    bus_service = ServiceBusService(service_namespace='mynamespace', account_key='mykey', issuer='myissuer')

    bus_service.create_queue('taskqueue')

**create\_queue** 还支持其他选项，以允许你重写默认队列设置，例如消息生存时间或最大队列大小。下面的示例演示如何将最大队列大小设置为 5 GB，将生存时间设置为 1 分钟：

    queue_options = Queue()
    queue_options.max_size_in_megabytes = '5120'
    queue_options.default_message_time_to_live = 'PT1M'

    bus_service.create_queue('taskqueue', queue_options)

## <a name="send-messages"> </a>如何向队列发送消息

若要向 Service Bus 队列发送消息，你的应用程序需要对**ServiceBusService** 对象调用 **send\_queue\_message** 方法。

下面的示例演示如何使用 **send\_queue\_message** 向名为“taskqueue”的队列发送测试消息：

    msg = Message('Test Message')
    bus_service.send_queue_message('taskqueue', msg)

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## <a name="receive-messages"> </a>如何从队列接收消息

对 **ServiceBusService** 对象使用 **receive\_queue\_message** 方法可从队列接收消息。

    msg = bus_service.receive_queue_message('taskqueue')
    print(msg.body)

在读取消息后将从队列中删除它们；不过，通过将可选参数 **peek\_lock** 设置为 **True**，你可以读取（扫视）并锁定消息，以避免将其从队列中删除。

在接收过程中读取并删除消息的默认行为是最简单的模式，并且最适合在发生故障时应用程序可以容忍不处理消息的情况。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

如果将 **peek\_lock** 参数设置为 **True**，则接收将变成一个两阶段操作，这样就可以支持无法容忍遗漏消息的应用程序。当Service Bus 接收请求时，它会查找要耗用的下一条消息，将其锁定以防止其他使用者接收它，然后将它返回到应用程序。在应用程序完成处理该消息（或将其可靠存储以备将来处理）后，它通过对 **Message** 对象调用 **delete** 方法完成接收过程的第二阶段。**delete** 方法会将消息标记为已使用，并从队列中删除它。

    msg = bus_service.receive_queue_message('taskqueue', peek_lock=True)
    print(msg.body)

    msg.delete()

## <a name="handle-crashes"> </a>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，则它可以对 **Message** 对象调用 **unlock** 方法。这将导致 Service Bus 解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），Service Bus 将自动解锁该消息并使它可再次被接收。

如果应用程序在处理消息之后，但在调用 **delete** 方法之前崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的**MessageId** 属性来实现，该属性在多次传送尝试中保持不变。

## <a name="next-steps"> </a> 后续步骤

现在，你已了解有关 Service Bus 队列的基础知识，单击下面的链接可了解更多信息。

-   查看 MSDN 参考：[队列、主题和订阅。][队列、主题和订阅。]

  [后续步骤]: #next-steps
  [什么是 Service Bus 队列？]: #what-are-service-bus-queues
  [创建服务命名空间]: #create-a-service-namespace
  [获得命名空间的默认管理凭据]: #obtain-default-credentials
  [如何：创建队列]: #create-queue
  [如何：向队列发送消息]: #send-messages
  [如何：从队列接收消息]: #receive-messages
  [如何：处理应用程序崩溃和不可读消息]: #handle-crashes
  [howto-service-bus-queues]: ../includes/howto-service-bus-queues.md
  [Python 安装指南]: ../python-how-to-install/
  [队列、主题和订阅。]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367516.aspx
