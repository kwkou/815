<properties 
	pageTitle="如何通过 Python 使用队列存储 | Windows Azure" 
	description="了解如何通过 Python 使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。" 
	services="storage" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="03/11/2015" 
	wacn.date="09/18/2015"/>

# 如何通过 Python 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

本指南演示如何使用 Azure 队列存储服务执行常见方案。相关示例是使用 Python 编写的，并使用 [Python Azure 包][]。介绍的方案包括“插入”、“查看”、“获取”和“删除”队列消息以及“创建和删除队列”。有关队列的详细信息，请参阅 [后续步骤](#Next-Steps) 部分。

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]


> [AZURE.NOTE] 如果您需要安装 Python 或 [Python Azure 包][]，请参阅 [Python 安装指南](/documentation/articles/python-how-to-install)。

## 如何：创建队列

可以通过 **QueueService** 对象来处理队列。以下代码创建 **QueueService** 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

	from azure.storage import QueueService

以下代码使用存储帐户名称和帐户密钥创建 **QueueService** 对象。使用实际帐户和密钥替换“myaccount”和“mykey”。

	queue_service = QueueService(account_name='myaccount', account_key='mykey')

	queue_service.create_queue('taskqueue')


## 如何：在队列中插入消息

若要在队列中插入消息，可使用 **put_message** 方法创建一条新消息并将其添加到队列中。

	queue_service.put_message('taskqueue', 'Hello World')


## 如何：扫视下一条消息

通过调用 **peek_messages** 方法，可以查看队列前面的消息，而不必从队列中将其删除。默认情况下，**peek_messages** 扫视单条消息。

	messages = queue_service.peek_messages('taskqueue')
	for message in messages:
		print(message.message_text)


## 如何：取消对下一条消息的排队

你的代码分两步从队列中删除消息。当你调用
**get_messages** 时，默认情况下你将获得队列中的下一条消息。从 **get_messages** 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要从队列中删除消息，你还必须调用 **delete_message**。此删除消息的两步过程可确保当您的代码因硬件或软件故障而无法处理消息时，您的其他代码实例可以获取同一消息并重试。你的代码在处理消息后会立即调用 **delete_message**。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)


## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息表示工作任务，则你可以使用此功能来更新该工作任务的状态。以下代码使用 **update_message** 方法来更新消息。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		queue_service.update_message('taskqueue', message.message_id, 'Hello World Again', message.pop_receipt, 0)

## 如何：用于对消息取消排队的其他选项

你可以通过两种方式自定义队列中的消息检索。
首先，你可以获取一批消息（最多 32 个）。其次，您可以设置更长或更短的不可见超时时间，从而允许您的代码使用更多或更少时间来完全处理每个消息。下面的代码示例使用
**get_messages** 方法在一次调用中获取 16 条消息。然后，使用 for 循环来处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	messages = queue_service.get_messages('taskqueue', numofmessages=16, visibilitytimeout=5*60)
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)

## 如何：获取队列长度

你可以获取队列中消息的估计数。将显示
**get_queue_metadata** 方法要求队列服务返回有关队列的元数据，并且 **x-ms-approximate-messages-count** 应用作返回的字典中的索引以查找计数。
结果仅是近似值，因为在队列服务响应您的请求之后，可能添加或删除了消息。

	queue_metadata = queue_service.get_queue_metadata('taskqueue')
	count = queue_metadata['x-ms-approximate-messages-count']

## 如何：删除队列

若要删除队列及其中包含的所有消息，请针对队列对象调用
**delete_queue** 方法。

	queue_service.delete_queue('taskqueue')

##<a id="Next-Steps"></a> 后续步骤

现在，您已了解有关队列存储的基础知识，请按照下面的链接了解更复杂的存储任务。

-   请参阅 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]

  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure 包]: https://pypi.python.org/pypi/azure
<!---HONumber=70-->