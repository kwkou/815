<properties
    pageTitle="如何通过 PHP 使用队列存储 | Azure"
    description="了解如何使用 Azure 队列存储服务创建和删除队列，以及插入、获取和删除消息。示例用 PHP 编写。"
    documentationcenter="php"
    services="storage"
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="7582b208-4851-4489-a74a-bb952569f55b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="PHP"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="robinsh" />

# 如何通过 PHP 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]

## 概述

本指南将演示如何使用 Azure 队列存储服务执行常见方案。这些示例使用来自 Windows SDK for PHP 中的类编写。介绍的方案包括插入、扫视、获取和删除队列消息以及创建和删除队列。

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 创建 PHP 应用程序

创建访问 Azure 队列存储的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。可使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，你将使用队列存储功能，可在 PHP 应用程序中本地调用，或在 Azure Web 角色、辅助角色或网站内运行的代码中调用这些功能。

## 获取 Azure 客户端库

[AZURE.INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

## 配置应用程序以访问队列存储

若要使用 Azure 队列存储 API，需执行以下操作：

1. 通过使用 [require\_once] 语句引用 autoloader 文件。
2. 引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [AZURE.NOTE]
本示例（以及本文中的其他示例）假定你已通过 Composer 安装用于 Azure 的 PHP 客户端库。如果已手动安装这些库，则将需要引用 `WindowsAzure.php` autoloader 文件。

	require_once 'vendor/autoload.php';
	use WindowsAzure\Common\ServicesBuilder;


在下面的示例中，将始终显示 `require_once` 语句，但只会引用执行该示例所需的类。

## 设置 Azure 存储连接

若要实例化 Azure 队列存储客户端，首先必须拥有有效的连接字符串。队列服务连接字符串的格式如下。

若要访问实时服务：

	DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey];EndpointSuffix=core.chinacloudapi.cn

若要访问模拟器存储：

	UseDevelopmentStorage=true


若要创建任何 Azure 服务客户端，需要使用 **ServicesBuilder** 类。可使用以下方法之一：

* 将连接字符串直接传递给它。
* 使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：
	* 默认情况下，它附带了对一个外部源的支持 - 环境变量
	* 可通过扩展 **ConnectionStringSource** 类来添加新源。

在此处列出的示例中，将直接传递连接字符串。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;

	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);


## 创建队列

**QueueRestProxy** 对象允许使用 **createQueue** 方法创建队列。创建队列时，可以在该队列上设置选项，但此操作不是必需的。（下面的示例演示了如何在队列上设置元数据。）

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;
	use MicrosoftAzure\Storage\Queue\Models\CreateQueueOptions;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	// OPTIONAL: Set queue metadata.
	$createQueueOptions = new CreateQueueOptions();
	$createQueueOptions->addMetaData("key1", "value1");
	$createQueueOptions->addMetaData("key2", "value2");

	try	{
		// Create queue.
		$queueRestProxy->createQueue("myqueue", $createQueueOptions);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

> [AZURE.NOTE] 不应依赖元数据密钥区分大小写的性质。以小写形式从服务中读取所有密钥。


## 向队列添加消息

若要将消息添加到队列，请使用 **QueueRestProxy->createMessage**。此方法接受队列名称、消息文本和消息选项（可选）。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;
	use MicrosoftAzure\Storage\Queue\Models\CreateMessageOptions;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	try	{
		// Create message.
		$builder = new ServicesBuilder();
		$queueRestProxy->createMessage("myqueue", "Hello World!");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 扫视下一条消息

通过调用 **QueueRestProxy->peekMessages**，可以扫视队列前面的消息，而不会从队列中将其删除。默认情况下，**peekMessage** 方法将返回单个消息，但可使用 **PeekMessagesOptions->setNumberOfMessages** 方法更改该值。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;
	use MicrosoftAzure\Storage\Queue\Models\PeekMessagesOptions;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	// OPTIONAL: Set peek message options.
	$message_options = new PeekMessagesOptions();
	$message_options->setNumberOfMessages(1); // Default value is 1.

	try	{
		$peekMessagesResult = $queueRestProxy->peekMessages("myqueue", $message_options);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	$messages = $peekMessagesResult->getQueueMessages();

	// View messages.
	$messageCount = count($messages);
	if($messageCount <= 0){
		echo "There are no messages.<br />";
	}
	else{
		foreach($messages as $message)	{
			echo "Peeked message:<br />";
			echo "Message Id: ".$message->getMessageId()."<br />";
			echo "Date: ".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
			echo "Message text: ".$message->getMessageText()."<br /><br />";
		}
	}

## 从队列中删除下一条消息

代码分两步从队列中删除消息。首先，调用 **queuerestproxy->listmessages**，这将使消息对从队列中读取的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。（如果在此时段内未删除该消息，则它将变得在队列上再次可见）。 若要从队列中删除消息，你必须调用 **QueueRestProxy->deleteMessage**。此删除消息的两步过程可确保当代码因硬件或软件故障而无法处理消息时，其他代码实例可以获取同一消息并重试。代码处理消息后会立即调用 **deleteMessage**。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	// Get message.
	$listMessagesResult = $queueRestProxy->listMessages("myqueue");
	$messages = $listMessagesResult->getQueueMessages();
	$message = $messages[0];

	/* ---------------------
		Process message.
	   --------------------- */

	// Get message ID and pop receipt.
	$messageId = $message->getMessageId();
	$popReceipt = $message->getPopReceipt();

	try	{
		// Delete message.
		$queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 更改已排队消息的内容

可以通过调用 **QueueRestProxy->updateMessage** 来更改队列中已就位消息的内容。如果消息表示工作任务，可使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常同时保留重试计数，当消息重试次数超过 *n* 时再删除该消息。这可避免每次处理某条消息时都触发应用程序错误。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	// Get message.
	$listMessagesResult = $queueRestProxy->listMessages("myqueue");
	$messages = $listMessagesResult->getQueueMessages();
	$message = $messages[0];

	// Define new message properties.
	$new_message_text = "New message text.";
	$new_visibility_timeout = 5; // Measured in seconds.

	// Get message ID and pop receipt.
	$messageId = $message->getMessageId();
	$popReceipt = $message->getPopReceipt();

	try	{
		// Update message.
		$queueRestProxy->updateMessage("myqueue",
									$messageId,
									$popReceipt,
									$new_message_text,
									$new_visibility_timeout);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 用于从队列中删除消息的其他选项

可通过两种方式自定义队列中的消息检索。首先，可获取一批消息（最多 32 条）。其次，可设置更长或更短的可见超时时间，允许代码使用更长或更短的时间来彻底处理每条消息。以下代码示例使用 **getMessages** 方法在一次调用中获取 16 条消息。然后，它会使用 **for** 循环处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;
	use MicrosoftAzure\Storage\Queue\Models\ListMessagesOptions;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	// Set list message options.
	$message_options = new ListMessagesOptions();
	$message_options->setVisibilityTimeoutInSeconds(300);
	$message_options->setNumberOfMessages(16);

	// Get messages.
	try{
		$listMessagesResult = $queueRestProxy->listMessages("myqueue",
														 $message_options);
		$messages = $listMessagesResult->getQueueMessages();

		foreach($messages as $message){

			/* ---------------------
				Process message.
			--------------------- */

			// Get message Id and pop receipt.
			$messageId = $message->getMessageId();
			$popReceipt = $message->getPopReceipt();

			// Delete message.
			$queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);
		}
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 获取队列长度

可获取队列中消息数量的估计值。**QueueRestProxy->getQueueMetadata** 方法要求队列服务返回有关队列的元数据。对返回的对象调用 **getApproximateMessageCount** 方法将提供队列中消息的计数。此计数仅为近似值，因为只能在队列服务响应的请求后添加或删除消息。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	try	{
		// Get queue metadata.
		$queue_metadata = $queueRestProxy->getQueueMetadata("myqueue");
		$approx_msg_count = $queue_metadata->getApproximateMessageCount();
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	echo $approx_msg_count;

## 删除队列

若要删除队列及其包含的所有消息，请调用 **QueueRestProxy->deleteQueue** 方法。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create queue REST proxy.
	$queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

	try	{
		// Delete queue.
		$queueRestProxy->deleteQueue("myqueue");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}


## 后续步骤

既已了解有关 Azure 队列存储的基础知识，可单击以下链接了解更复杂的存储任务。

- 访问 [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)。

有关详细信息，另请参阅 [PHP 开发人员中心](/develop/php/)。

[download]: /documentation/articles/php-download-sdk/
[require\_once]: http://www.php.net/manual/en/function.require-once.php
[Azure Portal]: https://portal.azure.cn

<!---HONumber=Mooncake_0103_2017-->