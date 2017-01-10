<properties
    pageTitle="如何通过 Ruby 使用队列存储 | Azure"
    description="了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。用 Ruby 编写的相关示例。"
    services="storage"
    documentationcenter="ruby"
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="59c2d81b-db9c-46ee-ade2-2f0caae6b1e6"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="robinsh" />  


# 如何通过 Ruby 使用队列存储
[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]

## 概述
本指南演示如何使用 Azure 队列存储服务执行常见方案。相关示例是使用 Ruby Azure API 编写的。介绍的方案包括“插入”、“扫视”、“获取”和“删除”队列消息以及“创建”和“删除”队列。

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 创建 Ruby 应用程序
创建 Ruby 应用程序。有关说明，请参阅 [Azure VM 上的 Ruby on Rails Web 应用程序](/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)。

## 配置应用程序以访问存储
要使用 Azure 存储，需下载和使用 Ruby Azure 包，其中包括与存储 REST 服务进行通信的一组方便的库。

### 使用 RubyGems 获取该程序包
1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。
2. 在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包
使用常用的文本编辑器将以下内容添加到要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## 设置 Azure 存储连接
Azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::QueueService** 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your Azure storage access key>"

 
在 Azure 门户中，从经典或 Resource Manager 存储帐户获取这些值：

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 导航到要使用的存储帐户。
3. 在右侧的“设置”边栏选项卡中，单击“访问密钥”。
4. 在显示的“访问密钥”边栏选项卡中，可看到访问密钥 1 和访问密钥 2。可以使用其中任意一个密钥。
5. 单击复制图标以将密钥复制到剪贴板。

从 Azure 经典管理门户中的经典存储帐户中获取这些值：

1. 登录到[ Azure 经典管理门户](https://manage.windowsazure.cn/)。
2. 导航到要使用的存储帐户。
3. 单击导航窗格底部的“管理访问密钥”。
4. 在弹出对话框中，可看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，可以使用主访问密钥，也可以使用辅助访问密钥。
5. 单击复制图标以将密钥复制到剪贴板。

## 如何：创建队列
以下代码将创建 **Azure::QueueService** 对象，可用于队列。

	azure_queue_service = Azure::QueueService.new

使用 **create\_queue()** 方法创建具有指定名称的队列。

	begin
	  azure_queue_service.create_queue("test-queue")
	rescue
	  puts $!
	end

## 如何：在队列中插入消息
若要在队列中插入消息，可使用 **create\_message()** 方法创建新消息并将其添加到队列。

	azure_queue_service.create_message("test-queue", "test message")

## 如何：扫视下一条消息
通过调用 **peek\_messages()** 方法，可以查看队列前面的消息，而不必从队列中将其删除。默认情况下，**peek\_messages()** 扫视单条消息。也可以指定要扫视的消息数。

	result = azure_queue_service.peek_messages("test-queue",
	  {:number_of_messages => 10})

## 如何：取消对下一条消息的排队
可通过两个步骤从队列中删除消息。

1. 在调用 **list\_messages()** 时，默认情况下会获取队列中的下一条消息。也可以指定要获取的消息数。从 **list\_messages()** 返回的消息变得对从此队列读取消息的任何其他代码不可见。以参数形式传入可见性超时秒数。
2. 若要从队列中删除消息，还必须调用 **delete\_message()**。

此删除消息的两步过程可确保当代码因硬件或软件故障而无法处理消息时，其他代码实例可以获取同一消息并重试。代码处理消息后会立即调用 **delete\_message()**。

	messages = azure_queue_service.list_messages("test-queue", 30)
	azure_queue_service.delete_message("test-queue", 
	  messages[0].id, messages[0].pop_receipt)

## 如何：更改已排队消息的内容
可更改队列中现有消息的内容。以下代码使用 **update\_message()** 方法来更新消息。该方法将返回一个元组，其中包含队列消息的 POP 接收方，以及一个 UTC 日期时间值，表示队列中显示消息的时间。

	message = azure_queue_service.list_messages("test-queue", 30)
	pop_receipt, time_next_visible = azure_queue_service.update_message(
	  "test-queue", message.id, message.pop_receipt, "updated test message", 
	  30)

## 如何：用于对消息取消排队的其他选项
可通过两种方式自定义队列中消息的检索。

1. 可获取一批消息。
2. 可设置更长或更短的不可见超时时间，从而允许代码使用更长或更短时间来完全处理每个消息。

以下代码示例使用 **list\_messages()** 方法通过一次调用获取 15 条消息。然后，它打印并删除每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	azure_queue_service.list_messages("test-queue", 300
	  {:number_of_messages => 15}).each do |m|
	  puts m.message_text
	  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
	end

## 如何：获取队列长度
可获取队列中消息数的估计值。**get\_queue\_metadata()** 方法要求队列服务返回有关队列的大概消息数和元数据。

	message_count, metadata = azure_queue_service.get_queue_metadata(
	  "test-queue")

## 如何：删除队列
若要删除队列及其包含的所有消息，请对队列对象调用 **delete\_queue()** 方法。

	azure_queue_service.delete_queue("test-queue")

## 后续步骤
既已了解有关队列存储的基础知识，可单击以下链接以了解更复杂的存储任务。

- 访问 [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- 访问 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

有关本文中讨论的 Azure 队列服务与[如何使用服务总线队列](/documentation/articles/service-bus-ruby-how-to-use-queues/)一文中讨论的 Azure 服务总线队列的比较，请参阅 [Azure 队列和服务总线队列 - 比较与对照](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)
 

<!---HONumber=Mooncake_0103_2017-->