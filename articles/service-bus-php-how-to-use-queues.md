<properties linkid="develop-php-how-to-guides-service-bus-queues" urlDisplayName="Service Bus Queues" pageTitle="如何使用 Service Bus 队列 (PHP) - Azure" metaKeywords="Azure Service Bus queues, Azure queues, Azure messaging, Azure queues PHP" description="了解如何在 Azure 中使用 Service Bus 队列。采用 PHP 编写的代码示例。" metaCanonical="" services="service-bus" documentationCenter="PHP" title="如何使用 Service Bus 队列" authors="" solutions="" manager="" editor="" />

# 如何使用 Service Bus 队列

本指南将演示如何将 Service Bus 队列与 PHP 一起使用。示例是
采用 PHP 编写的并使用了 [Azure SDK for PHP][Azure SDK for PHP]。所涉及
的任务包括**创建队列**、**发送和接收
消息**和**删除队列**。

## 目录

-   [什么是 Service Bus 队列？][什么是 Service Bus 队列？]
-   [创建服务命名空间][创建服务命名空间]
-   [获取命名空间的默认管理凭据][获取命名空间的默认管理凭据]
-   [创建 PHP 应用程序][创建 PHP 应用程序]
-   [获取 Azure 客户端库][获取 Azure 客户端库]
-   [将应用程序配置为使用 Service Bus][将应用程序配置为使用 Service Bus]
-   [如何：创建队列][如何：创建队列]
-   [如何：向队列发送消息][如何：向队列发送消息]
-   [如何：从队列接收消息][如何：从队列接收消息]
-   [如何：处理应用程序崩溃和不可读消息][如何：处理应用程序崩溃和不可读消息]
-   [后续步骤][后续步骤]

[WACOM.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## <span id="CreateApplication"></span></a>创建 PHP 应用程序

创建访问 Azure Blob 服务的 PHP 应用程序的唯一要求是从代码中引用 [Azure SDK for PHP][Azure SDK for PHP] 中的类。您可以使用任何开发工具（包括“记事本”）创建应用程序。

> [WACOM.NOTE]
> 您的 PHP 安装还必须已安装并启用 [OpenSSL 扩展][OpenSSL 扩展]。

在本指南中，您将使用服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## <span id="GetClientLibrary"></span></a>获取 Azure 客户端库

[WACOM.INCLUDE [get-client-libraries](../includes/get-client-libraries.md)]

## <span id="ConfigureApp"></span></a>配置应用程序以使用 Service Bus

若要使用 Azure Servise Bus 队列 API，您需要：

1.  使用 [require\_once][require\_once] 语句引用 autoloader 文件，并
2.  引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [WACOM.NOTE]
> 本示例（以及本文中的其他示例）假定您已通过 Composer 安装了适用于 Azure 的 PHP 客户端库。如果您已手动安装这些库或将其作为 PEAR 包安装，则需要引用 `WindowsAzure.php` autoloader 文件。

    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;

在下面的示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## <span id="ConnectionString"></span></a>设置 Azure Service Bus 连接

若要实例化 Azure Service Bus 客户端，您必须先拥有采用此格式的有效连接字符串：

    Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]

其中，终结点的格式通常为 `https://[yourNamespace].servicebus.windows.net`。

若要创建任何 Azure 服务客户端，您将需要使用 **ServicesBuilder** 类。您可以：

-   将连接字符串直接传递给此类或
-   使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：

    -   默认情况下，它附带了对一个外部源的支持 - 环境变量
    -   您可通过扩展 **ConnectionStringSource** 类来添加新源

在此处列出的示例中，将直接传递连接字符串。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $connectionString = "Endpoint=[yourEndpoint];SharedSecretIssuer=[Default Issuer];SharedSecretValue=[Default Key]";

    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

## <span id="CreateQueue"></span></a>如何：创建队列

Service Bus 队列的管理操作可通过
**ServiceBusRestProxy** 类执行。**ServiceBusRestProxy** 对象是
通过 **ServicesBuilder::createServiceBusService** 工厂方法与一个适当的连接字符串（该字符串封装了令牌权限以进行管理）构造的。

下面的示例演示了如何实例化 **ServiceBusRestProxy** 并调用 **ServiceBusRestProxy-\>createQueue**，以在 `MySBNamespace` 服务命名空间中创建名为 `myqueue` 的队列：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\Models\QueueInfo;

    // Create Service Bus REST proxy.
        $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

    try {
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

> [WACOM.NOTE]
> 您可以对 **ServiceBusRestProxy** 对象使用 **listQueues** 方法来检查具有指定名称的队列是否已存在于某个服务命名空间中。

## <span id="SendMessages"></span></a>如何：向队列发送消息

若要向 Service Bus 队列发送消息，您的应用程序将调用 **ServiceBusRestProxy-\>sendQueueMessage** 方法。下面的代码演示了如何将消息发送到我们之前在
`MySBNamespace` 服务命名空间中创建的 `myqueue` 队列。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\models\BrokeredMessage;

    // Create Service Bus REST proxy.
    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
        
    try {
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

发送到（和接收自）Service Bus 队列的消息
是 **BrokeredMessage** 类的实例。**BrokeredMessage** 对象包含一组
标准方法（如 **getLabel**、**getTimeToLive**、
**setLabel** 和 **setTimeToLive**）、用来保留
特定于应用程序的自定义属性的属性以及大量任意的
应用程序数据。

Service Bus 队列支持最大为 256 KB 的消息（标头
最大为 64 KB，其中包括标准和自定义
应用程序属性）。一个队列可包含的消息数不受
限制，但消息的总
大小受限。队列大小的上限为 5 GB。

## <span id="ReceiveMessages"></span></a>如何：从队列接收消息

从队列接收消息的主要方法是使用 **ServiceBusRestProxy-\>receiveQueueMessage** 方法。可在两种不同的模式中接收消息：**ReceiveAndDelete**（默认值）和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一项单步操作，即当 Service Bus 接收队列中某条消息的读取请求时，它会将该消息标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式，最适合应用程序
允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“将使用”，因此当应用程序重启并重新开始使用消息时，它会丢失在发生崩溃前使用的消息。

在 **PeekLock** 模式下，接收消息会变成一个两阶段操作，这将能够支持不能允许丢失消息的应用程序。当 Service Bus 收到请求时，它会找到要使用的下一个消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。在应用程序处理完消息（或以可靠方式存储消息以供将来处理）后，它通过将收到的消息传送到 **ServiceBusRestProxy-\>deleteMessage** 来完成接收过程的第二个阶段。当 Service Bus 发现 **deleteMessage** 调用时，它会将消息标记为“将使用”并将其从队列中删除。

下面的示例演示了如何使用 **PeekLock** 模式（非默认模式）接收和处理消息。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\ServiceBus\models\ReceiveMessageOptions;

    // Create Service Bus REST proxy.
    $serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
        
    try {
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

## <span id="HandleCrashes"></span></a>如何：处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助您轻松地从
应用程序错误或消息处理问题中恢复。如果
接收方应用程序出于某种原因无法处理消息，
它可以对收到的消息调用 **unlockMessage** 方法（而不
是 **deleteMessage** 方法）。这将导致 Service Bus
解锁队列中的消息并使其能够
重新被同一个正在使用的应用程序
或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的
超时，并且如果应用程序无法在锁定超时
到期之前处理消息（例如，如果应用程序崩溃）
，Service Bus 将自动解锁该消息
并使它可再次被接收。

如果在处理消息之后但在发出 **deleteMessage** 请求
之前应用程序发生崩溃，该消息将在
应用程序重新启动时重新传送给它。此情况通常
称作**至少处理一次**，即每条消息将
至少被处理一次，但在某些情况下，同一消息可能会被
重新传送。如果方案不允许重复处理，
则建议向应用程序添加其他逻辑来处理重复消息传送。通常可使用消息
的 **getMessageId** 方法实现此操作，这在
多个传送尝试中保持不变。

## <span id="NextSteps"></span></a>后续步骤

现在，您已了解 Service Bus 队列的基础知识，请参阅 MSDN
主题[队列、主题和订阅][队列、主题和订阅]以获取更多信息。

  [Azure SDK for PHP]: http://go.microsoft.com/fwlink/?LinkId=252473
  [什么是 Service Bus 队列？]: #what-are-service-bus-queues
  [创建服务命名空间]: #create-a-service-namespace
  [获取命名空间的默认管理凭据]: #obtain-default-credentials
  [创建 PHP 应用程序]: #CreateApplication
  [获取 Azure 客户端库]: #GetClientLibrary
  [将应用程序配置为使用 Service Bus]: #ConfigureApp
  [如何：创建队列]: #CreateQueue
  [如何：向队列发送消息]: #SendMessages
  [如何：从队列接收消息]: #ReceiveMessages
  [如何：处理应用程序崩溃和不可读消息]: #HandleCrashes
  [后续步骤]: #NextSteps
  [OpenSSL 扩展]: http://php.net/openssl
  [require\_once]: http://php.net/require_once
  [队列、主题和订阅]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367516.aspx
