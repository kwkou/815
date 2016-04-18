<properties 
	pageTitle="如何通过 Python 使用队列存储 | Azure" 
	description="了解如何通过 Python 使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。" 
	services="storage" 
	documentationCenter="python" 
	authors="emgerner-msft" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="02/11/2016"
	wacn.date="04/18/2016"/>

# 如何通过 Python 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

本指南演示如何使用 Azure 队列存储服务执行常见方案。这些示例通过 Python 编写并使用 [Azure Storage SDK for Python]。介绍的方案包括“插入”、“查看”、“获取”和“删除”队列消息以及“创建和删除队列”。有关队列的详细信息，请参阅 [后续步骤] 部分。

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 如何：创建队列

可以通过 **QueueService** 对象来处理队列。以下代码创建 **QueueService** 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

	from azure.storage.queue import QueueService

以下代码使用存储帐户名称和帐户密钥创建 **QueueService** 对象。使用你的帐户名称和密钥替换“myaccount”和“mykey”。

	queue_service = QueueService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

	queue_service.create_queue('taskqueue')


## 如何：在队列中插入消息

若要在队列中插入消息，可使用 **put_message** 方法创建一条新消息并将其添加到队列中。

	queue_service.put_message('taskqueue', u'Hello World')


## 如何：扫视下一条消息

通过调用 **peek_messages** 方法，可以查看队列前面的消息，而不必从队列中将其删除。默认情况下，**peek_messages** 扫视单条消息。

	messages = queue_service.peek_messages('taskqueue')
	for message in messages:
		print(message.message_text)


## 如何：取消消息的排队

你的代码分两步从队列中删除消息。在调用 **get_messages** 时，默认情况下你会获得队列中的下一条消息。从 **get_messages** 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要完成从队列中删除消息，你还必须调用 **delete_message**。此删除消息的两步过程可确保当您的代码因硬件或软件故障而无法处理消息时，您的其他代码实例可以获取同一消息并重试。你的代码在处理消息后会立即调用 **delete_message**。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		print(message.content)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)

你可以通过两种方式自定义队列中的消息检索。首先，你可以获取一批消息（最多 32 个）。其次，你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少时间来完全处理每个消息。以下代码示例使用 **get_messages** 方法来在一次调用中获取 16 条消息。然后，它会使用 for 循环处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	messages = queue_service.get_messages('taskqueue', numofmessages=16, visibilitytimeout=5*60)
	for message in messages:
		print(message.content)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)		


## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息表示工作任务，则你可以使用此功能来更新该工作任务的状态。以下代码使用 **update_message** 方法来更新消息。可见性超时设为 0，这意味着消息会立刻出现且内容将更新。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		queue_service.update_message('taskqueue', message.message_id, message.pop_receipt, 0, u'Hello World Again')

## 如何：获取队列长度

你可以获取队列中消息的估计数。**get_queue_metadata** 方法要求队列服务返回有关队列的元数据和 **approximate_message_count**。结果仅是近似值，因为在队列服务响应您的请求之后，可能添加或删除了消息。

	metadata = queue_service.get_queue_metadata('taskqueue')
	count = metadata.approximate_message_count

## 如何：删除队列

若要删除队列及其中包含的所有消息，请调用 **delete_queue** 方法。

	queue_service.delete_queue('taskqueue')

##<a id="Next-Steps"></a> 后续步骤

现在，你已了解队列存储的基础知识，可单击下面的链接了解详细信息。

- [Python 开发人员中心](/develop/python/)
- [Azure 存储空间服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- [Azure 存储团队博客]
- [Azure Storage SDK for Python]

[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python
[Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python

<!---HONumber=Mooncake_0411_2016-->