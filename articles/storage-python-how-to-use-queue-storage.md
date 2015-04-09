<properties linkid="develop-python-queue-service" urlDisplayName="Queue Service" pageTitle="如何使用队列服务 (Python) | Microsoft Azure" metaKeywords="Azure Queue Service messaging Python" description="了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。相关示例是使用 Python 编写的。" metaCanonical="" services="storage" documentationCenter="Python" title="How to Use the Queue Storage Service from Python" authors="" solutions="" manager="" editor="" />



# 如何从 Python 使用队列存储服务
本指南将演示如何使用 Microsoft Azure 队列服务
Azure 队列存储服务。相关示例是使用 Python
API 编写的。涉及的方案包括**插入**、**扫视**、
**获取**和**删除**队列消息，以及**创建和删除队列**。有关队列的详细信息，请参阅[后续步骤][]部分。

## 目录

[什么是队列存储？][]   
 [概念][]   
 [创建 Azure 存储帐户][]   
 [如何：创建队列][]   
 [如何：在队列中插入消息][]   
 [如何：扫视下一条消息][]   
 [如何：取消对下一条消息的排队][]   
 [如何：更改已排队消息的内容][]   
 [如何：用于对消息取消排队的其他方法][]   
 [如何：获取队列长度][]   
 [如何：删除队列][]   
 [后续步骤][]

[WACOM.INCLUDE [howto-queue-storage](../includes/howto-queue-storage.md)]

## <a name="create-account"> </a>创建 Azure 存储帐户
[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]


**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南](../python-how-to-install/)。

## <a name="create-queue"></a>如何：创建队列

可以通过 QueueService 对象来处理队列。以下代码创建 QueueService 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

	from azure.storage import QueueService

以下代码使用存储帐户名称和帐户密钥创建 QueueService 对象。使用实际帐户和密钥替换  'myaccount' 和  'mykey'。

	queue_service = QueueService(account_name='myaccount', account_key='mykey')

	queue_service.create_queue('taskqueue')


## <a name="insert-message"></a>如何：在队列中插入消息

若要在队列中插入消息，可使用 **put\_message** 方法创建一条新消息并将其添加到队列中。

	queue_service.put_message('taskqueue', 'Hello World')


## <a name="peek-message"></a>如何：扫视下一条消息

通过调用 **peek\_messages** 方法，可以查看队列前面的消息，而不必从队列中将其删除。默认情况下，
**peek\_messages** 扫视单个消息。

	messages = queue_service.peek_messages('taskqueue')
	for message in messages:
		print(message.message_text)


## <a name="get-message"></a>如何：取消对下一条消息的排队

你的代码分两步从队列中删除消息。当你调用
**get\_messages** 时，默认情况下你将获得队列中的下一条消息。从 **get\_messages** 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要从队列中删除消息，你还必须调用 **delete\_message**。此删除消息的两步过程可确保当您的代码因硬件或软件故障而无法处理消息时，您的其他代码实例可以获取同一消息并重试。你的代码在处理消息后会立即调用 **delete\_message**。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)


## <a name="change-contents"></a>如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息表示工作任务，则可以使用此功能来更新该工作任务的状态。以下代码使用 **update\_message** 方法来更新消息。

	messages = queue_service.get_messages('taskqueue')
	for message in messages:
		queue_service.update_message('taskqueue', message.message_id, 'Hello World Again', message.pop_receipt, 0)

## <a name="advanced-get"></a>如何：用于对消息取消排队的其他方法

你可以通过两种方式自定义队列中的消息检索。
首先，你可以获取一批消息（最多 32 个）。其次，您可以设置更长或更短的不可见超时时间，从而允许您的代码使用更多或更少时间来完全处理每个消息。下面的代码示例使用
**get\_messages** 方法在一次调用中获取 16 条消息。然后，使用 for 循环来处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	messages = queue_service.get_messages('taskqueue', numofmessages=16, visibilitytimeout=5*60)
	for message in messages:
		print(message.message_text)
		queue_service.delete_message('taskqueue', message.message_id, message.pop_receipt)

## <a name="get-queue-length"></a>如何：获取队列长度

你可以获取队列中消息的估计数。将显示
**get\_queue\_metadata** 方法要求队列服务返回有关队列的元数据，并且 **x-ms-approximate-messages-count** 应用作返回的字典中的索引以查找计数。
结果仅是近似值，因为在队列服务响应您的请求之后，可能添加或删除了消息。

	queue_metadata = queue_service.get_queue_metadata('taskqueue')
	count = queue_metadata['x-ms-approximate-messages-count']

## <a name="delete-queue"></a>如何：删除队列

若要删除队列及其中包含的所有消息，请针对队列对象调用
**delete\_queue** 方法。

	queue_service.delete_queue('taskqueue')

## <a name="next-steps"> </a>后续步骤

现在，你已了解有关队列存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]

  [后续步骤]: #next-steps
  [什么是队列存储？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [如何：创建队列]: #create-queue
  [如何：在队列中插入消息]: #insert-message
  [如何：扫视下一条消息]: #peek-message
  [如何：取消对下一条消息的排队]: #get-message
  [如何：更改已排队消息的内容]: #change-contents
  [如何：用于对消息取消排队的其他方法]: #advanced-get
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
<!--HONumber=41-->
