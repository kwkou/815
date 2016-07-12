<properties 
	pageTitle="如何通过 Ruby 使用队列存储 | Azure" 
	description="了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。用 Ruby 编写的相关示例。" 
	services="storage" 
	documentationCenter="ruby" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="05/04/2016" 
	wacn.date="06/13/2016"/>


# 如何通过 Ruby 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

本指南演示如何使用 Azure 队列存储服务执行常见方案。相关示例是使用 Ruby Azure API 编写的。介绍的方案包括“插入”、“扫视”、“获取”和“删除”队列消息以及“创建”和“删除”队列。

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅[Azure VM 上的 Ruby on Rails Web 应用程序](/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)。

## 配置应用程序以访问存储

要使用 Azure 存储空间，你需要下载和使用 Ruby azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2. 在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## 设置 Azure 存储连接

Azure 模块将读取环境变量 **AZURE_STORAGE_ACCOUNT** 和 **AZURE_STORAGE_ACCESS_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::QueueService** 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your Azure storage access key>"

获取这些值：

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)。
2. 导航到要使用的存储帐户
3. 单击导航窗格底部的“管理访问密钥”。
4. 在弹出对话框中，将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，您可以使用主访问密钥，也可以使用辅助访问密钥。 
5. 单击复制图标以将键复制到剪贴板。

## 如何：创建队列

以下代码将创建一个 **Azure::QueueService** 对象，您可以通过该对象来操作队列。

	azure_queue_service = Azure::QueueService.new

使用 create_queue() 方法创建具有指定名称的队列。

	begin
	  azure_queue_service.create_queue("test-queue")
	rescue
	  puts $!
	end

## 如何：在队列中插入消息

若要在队列中插入消息，可使用 create_message() 方法创建一条新消息并将其添加到队列中。

	azure_queue_service.create_message("test-queue", "test message")

## 如何：扫视下一条消息

通过调用 **peek_messages()** 方法，可以查看队列前面的消息，而不必从队列中将其删除。默认情况下，**peek_messages()** 扫视单条消息。也可以指定要扫视的消息数。

	result = azure_queue_service.peek_messages("test-queue",
	  {:number_of_messages => 10})

## 如何：取消对下一条消息的排队

可通过两个步骤从队列中删除消息。

1. 在调用 **list_messages()** 时，默认情况下会获取队列中的下一条消息。也可以指定要获取的消息数。从 **list_messages()** 返回的消息变得对从此队列读取消息的任何其他代码不可见。你将传递以秒为单位的可见性超时值作为参数。

2. 若要完成从队列中删除消息，您还必须调用 **delete_message()**。

此删除消息的两步过程可确保当您的代码因硬件或软件故障而无法处理消息时，您的其他代码实例可以获取同一消息并重试。你的代码在处理消息后会立即调用 **delete_message()**。

	messages = azure_queue_service.list_messages("test-queue", 30)
	azure_queue_service.delete_message("test-queue", 
	  messages[0].id, messages[0].pop_receipt)

## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。以下代码使用 **update_message()** 方法来更新消息。该方法将返回一个元组，其中包含队列消息的 pop 接收方，以及一个 UTC 日期时间值，表示消息将在队列中可见的时间。

	message = azure_queue_service.list_messages("test-queue", 30)
	pop_receipt, time_next_visible = azure_queue_service.update_message(
	  "test-queue", message.id, message.pop_receipt, "updated test message", 
	  30)

## 如何：用于对消息取消排队的其他选项

你可以通过两种方式自定义队列中的消息检索。

1. 你可以获取一批消息。

2. 你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少的时间来完全处理每个消息。

以下代码示例使用 **list_messages()** 方法通过一次调用获取 15 条消息。然后，它打印并删除每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	azure_queue_service.list_messages("test-queue", 300
	  {:number_of_messages => 15}).each do |m|
	  puts m.message_text
	  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
	end

## 如何：获取队列长度

你可以获取队列中消息数的估计值。**get_queue_metadata()** 方法要求队列服务返回有关队列的大概消息数和元数据。

	message_count, metadata = azure_queue_service.get_queue_metadata(
	  "test-queue")

## 如何：删除队列

若要删除队列及其包含的所有消息，请对队列对象调用 **delete_queue()** 方法。

	azure_queue_service.delete_queue("test-queue")

## 后续步骤

现在，您已了解有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- 访问 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- 访问 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

有关本文中讨论的 Azure 队列服务与[如何使用服务总线队列](/documentation/articles/service-bus-ruby-how-to-use-queues/)一文中讨论的 Azure 服务总线队列的比较，请参阅 [Azure 队列和服务总线队列 - 比较与对照](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)
 

<!---HONumber=Mooncake_0606_2016-->