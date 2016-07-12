<properties
	pageTitle="如何通过 Ruby 使用服务总线队列 | Microsoft Azure"
	description="了解如何在 Azure 中使用 Service Bus 队列。用 Ruby 编写的代码示例。"
	services="service-bus"
	documentationCenter="ruby"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.date="03/09/2016"
	wacn.date="01/14/2016"/>




# 如何使用 Service Bus 队列

[AZURE.INCLUDE [service-bus-selector-queues](../includes/service-bus-selector-queues.md)]

本指南介绍如何使用服务总线队列。相关示例通过 Ruby 编写并使用 Azure gem。涉及的任务包括**创建队列、发送和接收消息**以及**删除队列**。有关队列的详细信息，请参阅[后续步骤](#next-steps)部分。

## 什么是 Service Bus 队列？

Service Bus 队列支持*中转消息*通信模型。在使用队列时，分布式应用程序的组件不会直接相互通信，而是通过充当中介的队列交换消息。消息创建方（发送方）将消息传送到队列，然后继续对其进行处理。消息使用方（接收方）以异步方式从队列中提取消息并处理它。创建方不必等待使用方的答复即可继续处理并发送更多消息。队列为一个或多个竞争使用方提供**先入先出 (FIFO)** 消息传递方式。也就是说，接收方通常会按照消息添加到队列中的顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。

![QueueConcepts](./media/service-bus-ruby-how-to-use-queues/sb-queues-08.png)

Service Bus 队列是一种可用于各种应用场景的通用技术：

-   [多层 Azure 应用程序](/documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/)中 Web 角色和辅助角色之间的通信。
-   [混合解决方案](/documentation/articles/service-bus-dotnet-hybrid-app-using-service-bus-relay/)中本地应用程序和 Azure 托管应用程序之间的通信。
-   在不同组织或组织的各部门中本地运行的分布式应用程序组件之间的通信

利用队列，您可以更好地向外扩展应用程序，并增强您的体系结构的恢复能力。

## 创建服务命名空间
若要开始在 Azure 中使用服务总线队列，必须先创建一个服务命名空间。服务命名空间提供了用于对应用程序中的服务总线资源进行寻址的范围容器。必须通过命令行界面创建命名空间，因为 Azure 经典门户不会使用 ACS 连接创建命名空间。

创建服务命名空间：

1. 打开 Azure PowerShell 控制台。

2. 键入以下命令以创建服务总线命名空间。提供你自己的命名空间值，并指定与应用程序相同的区域。

    New-AzureSBNamespace -Name 'yourexamplenamespace' -Location 'China East' -NamespaceType 'Messaging' -CreateACSNamespace $true

    ![创建命名空间](./media/service-bus-ruby-how-to-use-queues/showcmdcreate.png)

## 获取命名空间的管理凭据
若要在新命名空间上执行管理操作（如创建队列），则必须获取该命名空间的管理凭据。

你运行的用于创建 Azure 服务总线命名空间的 PowerShell cmdlet 将显示可用于管理命名空间的密钥。复制 **DefaultKey** 值。你将本教程稍后的代码中使用此值。

![Copy key](./media/service-bus-ruby-how-to-use-queues/defaultkey.png)

> [AZURE.NOTE]登录到 [Azure 经典门户](http://manage.windowsazure.cn/)并导航到服务总线命名空间的连接信息后，也可以看到此密钥。

## 创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅[在 Azure 上创建 Ruby 应用程序](/zh-cn/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)。

## 配置应用程序以使用 Service Bus

要使用 Azure 服务总线，你需要使用 Ruby Azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2. 在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

    require "azure"

## 设置 Azure 服务总线连接

Azure 模块将读取环境变量 **AZURE_SERVICEBUS_NAMESPACE** 和 **AZURE_SERVICEBUS_ACCESS_KEY** 以获取连接到你的 Azure 服务总线命名空间所需的信息。如果未设置这些环境变量，则在使用 **Azure::ServiceBusService** 之前必须通过以下代码指定命名空间信息：

    Azure.config.sb_namespace = "<your azure service bus namespace>"
    Azure.config.sb_access_key = "<your azure service bus access key>"

将服务总线命名空间值设置为你创建的值，而不是整个 URL 的值。例如，使用 **"yourexamplenamespace"**，而不是 "yourexamplenamespace.servicebus.chinacloudapi.cn"。

## 如何创建队列

可以通过 **Azure::ServiceBusService** 对象处理队列。若要创建队列，请使用 **create_queue()** 方法。以下示例将创建一个队列或输出任何错误。

    azure_service_bus_service = Azure::ServiceBusService.new
    begin
      queue = azure_service_bus_service.create_queue("test-queue")
    rescue
      puts $!
    end

还可以通过其他选项传递 **Azure::ServiceBus::Queue** 对象，这些选项可让你重写默认队列设置，如消息保存时间或最大队列大小。以下示例演示如何将最大队列大小设置为 5GB，将生存时间设置为 1 分钟：

    queue = Azure::ServiceBus::Queue.new("test-queue")
    queue.max_size_in_megabytes = 5120
    queue.default_message_time_to_live = "PT1M"

    queue = azure_service_bus_service.create_queue(queue)

## 如何向队列发送消息

若要向服务总线队列发送消息，你的应用程序需要对 **Azure::ServiceBusService** 对象调用 **send_queue_message()** 方法。发往服务总线队列的消息以及从服务总线队列接收的消息是 **Azure::ServiceBus::BrokeredMessage** 对象，它们具有一组标准属性（如 **label** 和 **time_to_live**）、一个用于保存自定义应用程序特定属性的字典和一段任意应用程序数据正文。应用程序可以通过将字符串值作为消息传送来设置消息正文，任何必需的标准属性将用默认值来填充。

以下示例演示了如何使用 **send_queue_message()** 向名为“test-queue”的队列发送测试消息：

    message = Azure::ServiceBus::BrokeredMessage.new("test queue message")
    message.correlation_id = "test-correlation-id"
    azure_service_bus_service.send_queue_message("test-queue", message)

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## 如何从队列接收消息

可通过对 **Azure::ServiceBusService** 对象使用 **receive_queue_message()** 方法从队列接收消息。默认情况下，消息在被读取的同时会被锁定，从而无法从队列中删除。但是，你可以通过将 **:peek_lock** 选项设置为 **false**，在读取消息时将其从队列中删除。

默认行为使读取和删除变成一个两阶段操作，从而也有可能支持不允许遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。应用程序处理完该消息（或将其可靠地存储起来留待将来处理）后，会通过调用 **delete_queue_message()** 方法并提供要删除的消息作为参数来完成接收过程的第二阶段。**delete_queue_message()** 方法将该消息标记为“已使用”并将其从队列中删除。

如果 **:peek_lock** 参数设置为 **false**，读取并删除消息将是最简单的模式，并且最适合在发生故障时应用程序允许不处理消息的情况。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。因为服务总线会将消息标记为“已使用”，所以在应用程序重新启动并开始再次使用消息时，它会遗漏在崩溃之前使用过的消息。

以下示例演示了如何使用 **receive_queue_message()** 接收和处理消息。该示例先通过将 **:peek_lock** 设置为 **false** 接收并删除一条消息，然后再接收另一条消息，最后使用 **delete_queue_message()** 删除该消息：

    message = azure_service_bus_service.receive_queue_message("test-queue", 
	  { :peek_lock => false })
    message = azure_service_bus_service.receive_queue_message("test-queue")
    azure_service_bus_service.delete_queue_message(message)

## 如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，则它可以对 **Azure::ServiceBusService** 对象调用 **unlock\_queue\_message()** 方法。这将导致服务总线解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将自动解锁该消息并使它可再次被接收。

如果应用程序在处理消息之后，但在调用 **delete_queue_message()** 方法之前崩溃，则在应用程序重新启动时，该消息将重新传送给应用程序。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的 **message_id** 属性来实现，该属性在多次传送尝试中保持不变。

## 后续步骤

现在，你已了解有关 Service Bus 队列的基础知识，单击下面的链接可了解更多信息。

-   [队列、主题和订阅](/documentation/articles/service-bus-queues-topics-subscriptions/)的概述
-   访问 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

有关本文中讨论的 Azure 服务总线队列与[如何使用 Azure 队列服务](/develop/ruby/)一文中讨论的 Azure 队列的比较，请参阅 [Azure 队列和 Azure 服务总线队列 - 比较与对照](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)

<!---HONumber=Mooncake_0104_2016-->