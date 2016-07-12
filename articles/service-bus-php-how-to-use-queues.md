<properties 
	pageTitle="如何通过 PHP 使用服务总线队列 | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Service Bus 队列。采用 PHP 编写的代码示例。" 
	services="service-bus" 
	documentationCenter="php" 
	authors="sethmanheim" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.date="05/06/2016" 
	wacn.date="06/21/2016"/>

# 如何使用 Service Bus 队列

[AZURE.INCLUDE [service-bus-selector-queues](../includes/service-bus-selector-queues.md)]

本指南说明如何使用服务总线队列。示例是用 PHP 编写的并使用了 [Azure SDK for PHP](/documentation/articles/php-download-sdk/)。涉及的任务包括**创建队列**、**发送和接收消息**以及**删除队列**。

[AZURE.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## 创建 PHP 应用程序

创建访问 Azure Blob 服务的 PHP 应用程序的唯一要求是从代码中引用 [Azure SDK for PHP](/documentation/articles/php-download-sdk/) 中的类。你可以使用任何开发工具或记事本创建应用程序。

> [AZURE.NOTE]你的 PHP 安装还必须已安装并启用 [OpenSSL 扩展](http://php.net/openssl)。

在本指南中，你将使用服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## 获取 Azure 客户端库

[AZURE.INCLUDE [get-client-libraries](../includes/get-client-libraries.md)]

## 配置应用程序以使用 Service Bus

若要使用服务总线队列 API，请执行以下操作：

1. 使用 [require_once][require_once] 语句引用 autoloader 文件。
2. 引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [AZURE.NOTE]本示例（以及本文中的其他示例）假定你已通过 Composer 安装用于 Azure 的 PHP 客户端库。如果你已手动安装这些库或将其作为 PEAR 包安装，则必须引用 **WindowsAzure.php** autoloader 文件。

```
require_once 'vendor\autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

在以下示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## 设置服务总线连接

若要实例化服务总线客户端，必须先设置采用以下格式的有效连接字符串：

```
Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]
```

其中，**Endpoint** 的格式通常为 `https://[yourNamespace].servicebus.chinacloudapi.cn`。

若要创建任何 Azure 服务客户端，必须使用 **ServicesBuilder** 类。你可以：

* 将连接字符串直接传递给它。
* 使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：
	* 默认情况下，它附带了对一个外部源的支持 - 环境变量
	* 你可以通过扩展 **ConnectionStringSource** 类添加新的源

在此处列出的示例中，将直接传递连接字符串。

```
require_once 'vendor\autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## 如何：创建队列

可通过 **ServiceBusRestProxy** 类对服务总线队列执行管理操作。**ServiceBusRestProxy** 对象是通过 **ServicesBuilder::createServiceBusService** 工厂方法与一个适当的连接字符串（该字符串封装了令牌权限以进行管理）构造的。

下面的示例演示了如何实例化 **ServiceBusRestProxy** 并调用 **servicebusrestproxy->createqueue** 以创建 `MySBNamespace` 服务命名空间中名为 `myqueue` 的队列：

```
require_once 'vendor\autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\QueueInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
	
try	{
	$queueInfo = new QueueInfo("myqueue");
		
	// Create queue.
	$serviceBusRestProxy->createQueue($queueInfo);
}
catch(ServiceException $e){
	// Handle exception based on error codes and messages.
	// Error codes and messages are here: 
	// http://msdn.microsoft.com/zh-cn/library/windowsazure/dd179357
	$code = $e->getCode();
	$error_message = $e->getMessage();
	echo $code.": ".$error_message."<br />";
}
```

> [AZURE.NOTE] 你可以对 `ServiceBusRestProxy` 对象使用 `listQueues` 方法，以检查具有指定名称的队列是否已位于命名空间中。

## 如何：向队列发送消息

若要将消息发送到服务总线队列，应用程序应调用 **servicebusrestproxy->sendqueuemessage** 方法。下面的代码演示了如何将消息发送到在 `MySBNamespace` 服务命名空间先前创建的 `myqueue` 队列。

```
require_once 'vendor\autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
		
try	{
	// Create message.
	$message = new BrokeredMessage();
	$message->setBody("my message");
	
	// Send message.
	$serviceBusRestProxy->sendQueueMessage("myqueue", $message);
}
catch(ServiceException $e){
	// Handle exception based on error codes and messages.
	// Error codes and messages are here: 
	// http://msdn.microsoft.com/zh-cn/library/windowsazure/hh780775
	$code = $e->getCode();
	$error_message = $e->getMessage();
	echo $code.": ".$error_message."<br />";
}
```

发送至服务总线队列（和接收自服务总线队列）的消息是 **BrokeredMessage** 类实例。**BrokeredMessage** 对象具有一组标准方法（例如 **getLabel**、**getTimeToLive**、**setLabel** 和 **setTimeToLive**）和用来保存自定义的特定于应用程序的属性和任意应用程序数据正文的属性。

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。队列大小的上限为 5 GB。

## 如何从队列接收消息

从队列接收消息的最佳方法是使用 **servicebusrestproxy->receivequeuemessage** 方法。可以采用两种不同的模式接收消息：**ReceiveAndDelete**（默认）和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一项单次操作，即，当服务总线接收到队列中某条消息的读取请求时，它会将该消息标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

在 **PeekLock** 模式下，接收消息会变成一个两阶段操作，这将能够支持不能允许丢失消息的应用程序。当 Service Bus 收到请求时，它会找到要使用的下一个消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。在应用程序处理完消息（或以可靠方式存储消息以供将来处理）后，它通过将收到的消息传送到 **ServiceBusRestProxy->deleteMessage** 来完成接收过程的第二个阶段。当服务总线发现 **deleteMessage** 调用时，它会将消息标记为“正在使用”并将其从队列中删除。

以下示例演示了如何使用 **PeekLock** 模式（非默认模式）接收和处理消息。

```
require_once 'vendor\autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
		
try	{
	// Set the receive mode to PeekLock (default is ReceiveAndDelete).
	$options = new ReceiveMessageOptions();
	$options->setPeekLock();
		
	// Receive message.
	$message = $serviceBusRestProxy->receiveQueueMessage("myqueue", $options);
	echo "Body: ".$message->getBody()."<br />";
	echo "MessageID: ".$message->getMessageId()."<br />";
		
	/*---------------------------
		Process message here.
	----------------------------*/
		
	// Delete message. Not necessary if peek lock is not set.
	echo "Message deleted.<br />";
	$serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
	// Handle exception based on error codes and messages.
	// Error codes and messages are here:
	// http://msdn.microsoft.com/zh-cn/library/windowsazure/hh780735
	$code = $e->getCode();
	$error_message = $e->getMessage();
	echo $code.": ".$error_message."<br />";
}
```

## 如何：处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序出于某种原因无法处理消息，它可以对收到的消息调用 **unlockMessage** 方法（而不是 **deleteMessage** 方法）。这将导致 Service Bus 解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将自动解锁该消息并使它可再次被接收。

如果在处理消息之后但在发出 **deleteMessage** 请求之前应用程序发生崩溃，该消息将在应用程序重新启动时重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案不允许重复处理，则建议向应用程序添加其他逻辑来处理重复消息传送。通常可使用消息的 **getMessageId** 方法实现此操作，这在多个传送尝试中保持不变。

## 后续步骤

现在，你已了解服务总线队列的基础知识，请参阅[队列、主题和订阅][]以获取更多信息。

有关详细信息，请参阅 [PHP 开发人员中心](/develop/php/)。

[队列、主题和订阅]: /documentation/articles/service-bus-queues-topics-subscriptions/
[require_once]: http://php.net/require_once

<!---HONumber=82-->