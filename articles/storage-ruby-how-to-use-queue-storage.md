<properties linkid="dev-ruby-how-to-service-bus-queues" urlDisplayName="Queue Service" pageTitle="How to use the queue service (Ruby) | Microsoft Azure" metaKeywords="Azure Queue Service get messages Ruby" description="Learn how to use the Azure Queue service to create and delete queues, and insert, get, and delete messages. Samples written in Ruby." metaCanonical="" services="storage" documentationCenter="Ruby" title="How to Use the Queue Storage Service from Ruby" authors="guayan" solutions="" manager="" editor="" />

# 如何通过 Ruby 使用队列存储服务

本指南将演示如何使用 Windows Azure 队列存储服务执行常见方案。
示例是使用 Ruby Azure API 编写的。
涉及的方案包括**插入**、**扫视**、
**获取**和**删除**队列消息以及
**创建和删除队列**。有关队列的详细信息，请参阅
[后续步骤][]部分。

## 目录

-   [什么是队列存储？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Ruby 应用程序][]
-   [配置应用程序以访问存储][]
-   [设置 Azure 存储连接][]
-   [如何：创建队列][]
-   [如何：在队列中插入消息][]
-   [如何：扫视下一条消息][]
-   [如何：取消对下一条消息的排队][]
-   [如何：更改已排队消息的内容][]
-   [如何：用于对消息取消排队的其他方法][]
-   [如何：获取队列长度][]
-   [如何：删除队列][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-queue-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅
[在 Azure 上创建 Ruby 应用程序][]。

## 配置应用程序以访问存储

要使用 Azure 存储空间，你需要下载和使用 Ruby azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1.  使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2.  在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

    require "azure"

## 设置 Azure 存储连接

azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY** 以获取
连接到你的 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在
使用 **Azure::QueueService** 之前必须通过以下代码指定帐户信息：

    Azure.config.account_name = "<your azure storage account>"
    Azure.config.access_key = "<your Azure storage access key>"

获取这些值：

1.  登录到 [Azure 管理门户][]。
2.  导航到要使用的存储帐户
3.  单击导航窗格底部的**“管理密钥”**。
4.  在弹出对话框中，你将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，你可以选择主访问密钥，也可以选择辅助访问密钥。

## 如何：创建队列

以下代码将创建一个 **Azure::QueueService** 对象，你可以通过该对象来操作队列。

    azure_queue_service = Azure::QueueService.new

使用 **create\_queue()** 方法创建具有指定名称的队列。

    begin
    azure_queue_service.create_queue("test-queue")
    rescue
    puts $!
    end

## 如何：在队列中插入消息

若要在队列中插入消息，可使用 **create\_message()** 方法创建一条新消息并将其添加到队列中。

    azure_queue_service.create_message("test-queue", "test message")

## 如何：扫视下一条消息

通过调用 **peek\_messages()** 方法，你可以扫视队列前面的消息，而不会从队列中删除它。默认情况下，**peek\_messages()** 扫视单个消息。也可以指定要扫视的消息数。

    result = azure_queue_service.peek_messages("test-queue",
    {:number_of_messages => 10})

## 如何：取消对下一条消息的排队

可通过两个步骤从队列中删除消息。

1.  在调用 **list\_messages()** 时，默认情况下会获取队列中的下一条消息。也可以指定要获取的消息数。对于从此队列读取消息的任何其他代码，从 **list\_messages()** 返回的消息将变得不可见。你将传递以秒为单位的可见性超时值作为参数。

2.  若要从队列中删除消息，你还必须调用 **delete\_message()**。

此删除消息的两步过程确保在你的代码因硬件或软件故障而无法处理消息时，代码的另一个实例可以获取相同消息并重试。你的代码在处理消息后会立即调用 **delete\_message()**。

    messages = azure_queue_service.list_messages("test-queue", 30)
    azure_queue_service.delete_message("test-queue", 
    messages[0].id, messages[0].pop_receipt)

## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。以下代码使用 **update\_message()** 方法来更新消息。该方法将返回一个元组，其中包含队列消息的 pop 接收方，以及一个 UTC 日期时间值，表示消息将在队列中可见的时间。

    message = azure_queue_service.list_messages("test-queue", 30)
    pop_receipt, time_next_visible = azure_queue_service.update_message(
    "test-queue", message.id, message.pop_receipt, "updated test message", 
      30)

## 如何：用于对消息取消排队的其他方法

你可以通过两种方式自定义队列中的消息检索。

1.  你可以获取一批消息。

2.  你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少的时间来完全处理每个消息。

以下代码示例使用 **list\_messages()** 方法在一次调用中获取 15 条消息。然后，它打印并删除每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

    azure_queue_service.list_messages("test-queue", 300
    {:number_of_messages => 15}).each do |m|
    puts m.message_text
    azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
    end

## 如何：获取队列长度

你可以获取队列中消息数的估计值。**get\_queue\_metadata()** 方法要求队列服务返回有关队列的大概消息数和元数据。

    message_count, metadata = azure_queue_service.get_queue_metadata(
    "test-queue")

## 如何：删除队列

若要删除队列及其包含的所有消息，请对队列对象调用 **delete\_queue()** 方法。

    azure_queue_service.delete_queue("test-queue")

## 后续步骤

现在，你已了解有关队列存储的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]
-   访问 GitHub 上的 [Azure SDK for Ruby][] 存储库

有关本文中讨论的 Azure 队列服务与[如何使用 Service Bus 队列][]一文中讨论的 Azure Service Bus 队列的比较，请参阅 [Azure 队列和 Azure Service Bus 队列 - 比较与对照][]

  [后续步骤]: #next-steps
  [什么是队列存储？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 Ruby 应用程序]: #create-a-ruby-application
  [配置应用程序以访问存储]: #configure-your-application-to-access-storage
  [设置 Azure 存储连接]: #setup-a-windows-azure-storage-connection
  [如何：创建队列]: #how-to-create-a-queue
  [如何：在队列中插入消息]: #how-to-insert-a-message-into-a-queue
  [如何：扫视下一条消息]: #how-to-peek-at-the-next-message
  [如何：取消对下一条消息的排队]: #how-to-dequeue-the-next-message
  [如何：更改已排队消息的内容]: #how-to-change-the-contents-of-a-queued-message
  [如何：用于对消息取消排队的其他方法]: #how-to-additional-options-for-dequeuing-messages
  [如何：获取队列长度]: #how-to-get-the-queue-length
  [如何：删除队列]: #how-to-delete-a-queue
  [howto-queue-storage]: ../includes/howto-queue-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [在 Azure 上创建 Ruby 应用程序]: /en-us/develop/ruby/tutorials/web-app-with-linux-vm/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure SDK for Ruby]: https://github.com/WindowsAzure/azure-sdk-for-ruby
  [如何使用 Service Bus 队列]: /en-us/develop/ruby/how-to-guides/service-bus-queues/
  [Azure 队列和 Azure Service Bus 队列 - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/azure/hh767287.aspx
