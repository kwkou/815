<properties linkid="dev-ruby-how-to-service-bus-queues" urlDisplayName="Service Bus Queues" pageTitle="How to use Service Bus queues (Ruby) - Azure" metaKeywords="Azure Service Bus queues, Azure queues, Azure messaging, Azure queues Ruby" description="Learn how to use Service Bus queues in Azure. Code samples written in Ruby." metaCanonical="" services="service-bus" documentationCenter="Ruby" title="How to Use Service Bus Queues" authors="guayan" solutions="" manager="" editor="" />

# 如何使用 Service Bus 队列

本指南演示如何使用 Service Bus 队列。相关示例通过 Ruby 编写并使用 Azure gem。所涉及的任务包括**创建队列、发送和接收消息**和**删除队列**。有关队列的详细信息，请参阅[后续步骤][后续步骤]一节。

## 目录

-   [什么是 Service Bus 队列？][什么是 Service Bus 队列？]
-   [创建服务命名空间][创建服务命名空间]
-   [获得命名空间的默认管理凭据][获得命名空间的默认管理凭据]
-   [创建 Ruby 应用程序][创建 Ruby 应用程序]
-   [配置应用程序以使用 Service Bus][配置应用程序以使用 Service Bus]
-   [设置 Azure Service Bus 连接][设置 Azure Service Bus 连接]
-   [如何创建队列][如何创建队列]
-   [如何向队列发送消息][如何向队列发送消息]
-   [如何从队列接收消息][如何从队列接收消息]
-   [如何处理应用程序崩溃和不可读消息][如何处理应用程序崩溃和不可读消息]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## <span id="create-a-ruby-application"></span></a>创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅[在 Azure 上创建 Ruby 应用程序][在 Azure 上创建 Ruby 应用程序]。

## <span id="configure-your-application-to-use-service-bus"></span></a>配置应用程序以使用 Service Bus

要使用 Azure Service Bus，你需要下载和使用 Ruby azure 程序包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1.  使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2.  在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

    require "azure"

## <span id="setup-a-windows-azure-service-bus-connection"></span></a>设置 Azure Service Bus 连接

azure 模块将读取环境变量 **AZURE\_SERVICEBUS\_NAMESPACE** 和 **AZURE\_SERVICEBUS\_ACCESS\_KEY** 以获取连接到你的 Azure Service Bus 命名空间所需的信息。如果未设置这些环境变量，则在使用 **Azure::ServiceBusService** 之前必须通过以下代码指定命名空间信息：

    Azure.config.sb_namespace = "<your azure service bus namespace>"
    Azure.config.sb_access_key = "<your azure service bus access key>"

## <span id="how-to-create-a-queue"></span></a>如何创建队列

可以通过 **Azure::ServiceBusService** 对象处理队列。若要创建队列，请使用 **create\_queue()** 方法。以下示例创建一个队列或输出存在的错误。

    azure_service_bus_service = Azure::ServiceBusService.new
    begin
      queue = azure_service_bus_service.create_queue("test-queue")
    rescue
      puts $!
    end

还可以通过其他选项传入 **Azure::ServiceBus::Queue** 对象，这些选项允许你重写默认队列设置，如消息保存时间或最大队列大小。下面的示例演示如何将最大队列大小设置为 5 GB，将保存时间设置为 1 分钟：

    queue = Azure::ServiceBus::Queue.new("test-queue")
    queue.max_size_in_megabytes = 5120
    queue.default_message_time_to_live = "PT1M"

    queue = azure_service_bus_service.create_queue(queue)

## <span id="how-to-send-messages-to-a-queue"></span></a>如何向队列发送消息

若要向 Service Bus 队列发送消息，你的应用程序需要对 **Azure::ServiceBusService** 对象调用 **send\_queue\_message()** 方法。发往 Service Bus 队列的消息以及从 Service Bus 队列接收的消息是 **Azure::ServiceBus::BrokeredMessage** 对象，它们具有一组标准属性（如 **label** 和 **time\_to\_live**）、一个用于保存自定义应用程序特定属性的字典和一段任意应用程序数据正文。应用程序可以通过将字符串值作为消息传送来设置消息正文，任何必需的标准属性将用默认值来填充。

下面的示例演示如何使用 **send\_queue\_message()** 向名为“test-queue”的队列发送测试消息：

    message = Azure::ServiceBus::BrokeredMessage.new("test queue message")
    message.correlation_id = "test-correlation-id"
    azure_service_bus_service.send_queue_message("test-queue", message)

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## <span id="how-to-receive-messages-from-a-queue"></span></a>如何从队列接收消息

消息通过对 **Azure::ServiceBusService** 对象使用 **receive\_queue\_message()** 方法从队列接收。默认情况下，消息在被读取的同时会被锁定，从而无法从队列中删除。但是，你可以通过将 **:peek\_lock** 选项设置为 **false**，在读取消息时将其从队列中删除。

默认行为使读取和删除变成一个两阶段操作，从而有可能支持不允许遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。应用程序处理完该消息（或将其可靠地存储起来留待将来处理）后，会通过调用 **delete\_queue\_message()** 方法并提供要删除的消息作为参数来完成接收过程的第二阶段。**delete\_queue\_message()** 方法将该消息标记为“已使用”并将其从队列中删除。

如果 **:peek\_lock** 参数设置为 **false**，读取并删除消息将是最简单的模式，并且最适合在发生故障时应用程序允许不处理消息的情况。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

下面的示例演示了如何使用 **receive\_queue\_message()** 接收和处理消息。该示例先通过将 **:peek\_lock** 设置为 **false** 接收并删除一条消息，然后再接收另一条消息，最后使用 **delete\_queue\_message()** 删除该消息：

    message = azure_service_bus_service.receive_queue_message("test-queue", 
      { :peek_lock => false })
    message = azure_service_bus_service.receive_queue_message("test-queue")
    azure_service_bus_service.delete_queue_message("test-queue",
      message.sequence_number, message.lock_token)

## <span id="how-to-handle-application-crashes-and-unreadable-messages"></span></a>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，则它可以对 **Azure::ServiceBusService** 对象调用 **unlock\_queue\_message()** 方法。这将导致 Service Bus 解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），Service Bus 将自动解锁该消息并使它可再次被接收。

如果应用程序在处理消息之后，但在调用 **delete\_queue\_message()** 方法之前崩溃，则在应用程序重新启动时，该消息将重新传送给应用程序。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的 **message\_id** 属性来实现，该属性在多次传送尝试中保持不变。

## <span id="next-steps"></span></a> 后续步骤

现在，你已了解有关 Service Bus 队列的基础知识，单击下面的链接可了解更多信息。

-   查看 MSDN 参考：[队列、主题和订阅][队列、主题和订阅]
-   访问 GitHub 上的 [Azure SDK for Ruby][Azure SDK for Ruby] 存储库

有关本文中讨论的 Azure Service Bus 队列与[如何使用 Azure 队列服务][如何使用 Azure 队列服务]一文中讨论的 Azure 队列的比较，请参阅 [Azure 队列和 Azure Service Bus 队列 - 比较与对照][Azure 队列和 Azure Service Bus 队列 - 比较与对照]

  [后续步骤]: #next-steps
  [什么是 Service Bus 队列？]: #what-are-service-bus-queues
  [创建服务命名空间]: #create-a-service-namespace
  [获得命名空间的默认管理凭据]: #obtain-default-credentials
  [创建 Ruby 应用程序]: #create-a-ruby-application
  [配置应用程序以使用 Service Bus]: #configure-your-application-to-use-service-bus
  [设置 Azure Service Bus 连接]: #setup-a-windows-azure-service-bus-connection
  [如何创建队列]: #how-to-create-a-queue
  [如何向队列发送消息]: #how-to-send-messages-to-a-queue
  [如何从队列接收消息]: #how-to-receive-messages-from-a-queue
  [如何处理应用程序崩溃和不可读消息]: #how-to-handle-application-crashes-and-unreadable-messages
  [howto-service-bus-queues]: ../includes/howto-service-bus-queues.md
  [在 Azure 上创建 Ruby 应用程序]: /zh-cn/develop/ruby/tutorials/web-app-with-linux-vm/
  [队列、主题和订阅]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367516.aspx
  [Azure SDK for Ruby]: https://github.com/WindowsAzure/azure-sdk-for-ruby
  [如何使用 Azure 队列服务]: /zh-cn/develop/ruby/how-to-guides/queue-service/
  [Azure 队列和 Azure Service Bus 队列 - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh767287.aspx
