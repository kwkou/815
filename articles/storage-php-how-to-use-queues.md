<properties title="How to use the queue service (PHP) - Azure feature guide" pageTitle="How to use the queue service (PHP) | Microsoft Azure" metaKeywords="Azure Queue Service messaging PHP" description="Learn how to use the Azure Queue service to create and delete queues, and insert, get, and delete messages. Samples written in PHP." documentationCenter="PHP" services="storage" authors="" />

# 如何通过 PHP 使用队列服务

本指南将演示如何使用 Azure 队列服务执行常见方案。这些示例通过使用 Windows SDK for PHP 中的类编写。涉及的方案包括**插入**、**查看**、**获取**和**删除**队列消息以及**创建和删除队列**。有关队列的更多信息，请参见[后续步骤][]一节。

## 目录

-   [什么是队列存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 PHP 应用程序][]
-   [针对队列服务配置应用程序][]
-   [设置 Azure 存储连接][]
-   [如何：创建队列][]
-   [如何：向队列添加消息][]
-   [如何：查看下一条消息][]
-   [如何：取消对下一条消息的排队][]
-   [如何：更改已排队消息的内容][]
-   [用于取消对消息进行排队的其他选项][]
-   [如何：获取队列长度][]
-   [如何：删除队列][]
-   [后续步骤][1]

[WACOM.INCLUDE [howto-queue-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 PHP 应用程序

创建访问 Azure 队列服务的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。你可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，你将使用队列服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## 获取 Azure 客户端库

[WACOM.INCLUDE [get-client-libraries][]]

## 配置你的应用程序以访问队列服务

若要使用 Azure 队列服务 API，你需要：

1.  使用 [require\_once][] 语句引用 autoloader 文件，并
2.  引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [WACOM.NOTE]
> 本示例（以及本文中的其他示例）假定你已通过 Composer 安装了用于 Azure 的 PHP 客户端库。如果你已手动安装这些库或将其作为 PEAR 包安装，则需要引用 `WindowsAzure.php` autoloader 文件。

    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;

在下面的示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## 设置 Azure 存储连接

若要实例化 Azure 队列服务客户端，你必须首先具有有效的连接字符串。队列服务连接字符串的格式为：

对于访问实时服务：

    DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

对于访问模拟器存储：

    UseDevelopmentStorage=true

若要创建任何 Azure 服务客户端，你将需要使用 **ServicesBuilder** 类。你可以：

-   将连接字符串直接传递给此类或
-   使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：

    -   默认情况下，它附带了对一个外部源的支持 - 环境变量
    -   你可通过扩展 **ConnectionStringSource** 类来添加新源

在此处列出的示例中，将直接传递连接字符串。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

## 如何：创建队列

利用 **QueueRestProxy** 对象，可以使用 **createQueue** 方法创建队列。创建队列时，可以在该队列上设置选项，但此操作不是必需的。（下面的示例演示了如何在队列上设置元数据。）

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\CreateQueueOptions;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // 可选：设置队列元数据。
    $createQueueOptions = new CreateQueueOptions();
    $createQueueOptions->addMetaData("key1", "value1");
    $createQueueOptions->addMetaData("key2", "value2");

    try {
    // 创建队列。
    $queueRestProxy->createQueue("myqueue", $createQueueOptions);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

> [WACOM.NOTE]
> 你不应期望元数据密钥区分大小写。所有密钥都是采用小写形式从服务中读取的。

## 如何：向队列添加消息

若要将消息添加到队列，请使用 **QueueRestProxy-\>createMessage**。此方法接受队列名称、消息文本和消息选项（这些都是可选的）。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\CreateMessageOptions;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try {
    // 创建消息。
    $builder = new ServicesBuilder();
    $queueRestProxy->createMessage("myqueue", "Hello World!");
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 如何：扫视下一条消息

通过调用 **QueueRestProxy-\>peekMessages**，可以扫视队列前面的消息，而不会从队列中将其删除。默认情况下，**peekMessage** 方法将返回单个消息，但你可以使用 **PeekMessagesOptions-\>setNumberOfMessages** 方法更改该值。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\PeekMessagesOptions;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // 可选：设置扫视消息选项。
    $message_options = new PeekMessagesOptions();
    $message_options->setNumberOfMessages(1); // 默认值为 1。

    try {
    $peekMessagesResult = $queueRestProxy->peekMessages("myqueue", $message_options);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    $messages = $peekMessagesResult->getQueueMessages();

    // 查看消息。
    $messageCount = count($messages);
    if($messageCount <= 0){
    echo "There are no messages.<br />";
    }
    else{
    foreach($messages as $message)  {
    echo "Peeked message:<br />";
    echo "Message Id:".$message->getMessageId()."<br />";
    echo "Date:".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
    echo "Message text:".$message->getMessageText()."<br /><br />";
        }
    }

## 如何：取消对下一条消息的排队

你的代码分两步从队列中删除消息。首先，你将调用 **QueueRestProxy-\>listMessages**，这将使消息对从该队列读取的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见（如果在此时段内未删除该消息，则它将变得在队列上再次可见）。若要从队列中删除消息，你必须调用 **QueueRestProxy-\>deleteMessage**。此删除消息的两步过程确保在你的代码因硬件或软件故障而无法处理消息时，代码的另一个实例可以获取相同消息并重试。你的代码在处理消息后会立即调用 **deleteMessage**。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // 获取消息。
    $listMessagesResult = $queueRestProxy->listMessages("myqueue");
    $messages = $listMessagesResult->getQueueMessages();
    $message = $messages[0];

    /* ---------------------
    处理消息。
       --------------------- */

    // 获取消息 Id 和 pop 接收方。
    $messageId = $message->getMessageId();
    $popReceipt = $message->getPopReceipt();

    try {
    // 删除消息。
    $queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 如何：更改已排队消息的内容

可以通过调用 **QueueRestProxy-\>updateMessage** 来更改队列中已就位消息的内容。如果消息表示工作任务，则可以使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常，你还可以保留重试计数，如果某条消息的重试次数超过 n 次，你将删除此消息。这可避免每次处理某条消息时都触发应用程序错误。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;   

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // 获取消息。
    $listMessagesResult = $queueRestProxy->listMessages("myqueue");
    $messages = $listMessagesResult->getQueueMessages();
    $message = $messages[0];

    // 定义新消息属性。
    $new_message_text = "New message text.";
    $new_visibility_timeout = 5; // 以秒为度量单位。 

    // 获取消息 Id 和 pop 接收方。
    $messageId = $message->getMessageId();
    $popReceipt = $message->getPopReceipt();

    try {
    // 更新消息。
    $queueRestProxy->updateMessage("myqueue", 
    $messageId, 
    $popReceipt, 
    $new_message_text, 
    $new_visibility_timeout);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 用于取消对消息进行排队的其他选项

你可以通过两种方式自定义队列中的消息检索。首先，你可以获取一批消息（最多 32 个）。其次，你可以设置更长或更短的可见超时，从而允许你的代码使用更多或更少的时间来彻底处理每条消息。以下代码示例使用 **getMessages** 方法通过一次调用获取 16 条消息。然后，它使用 **for** 循环来处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Queue\Models\ListMessagesOptions;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    // 设置列表消息选项。 
    $message_options = new ListMessagesOptions();
    $message_options->setVisibilityTimeoutInSeconds(300); 
    $message_options->setNumberOfMessages(16);

    // 获取消息。
    try {
    $listMessagesResult = $queueRestProxy->listMessages("myqueue", 
    $message_options); 
    $messages = $listMessagesResult->getQueueMessages(); 

    foreach($messages as $message){
            
            /* ---------------------
    处理消息。
            --------------------- */
        
    // 获取消息 Id 和 pop 接收方。
    $messageId = $message->getMessageId();
    $popReceipt = $message->getPopReceipt();
            
    // 删除消息。
    $queueRestProxy->deleteMessage("myqueue", $messageId, $popReceipt);   
        }
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 如何：获取队列长度

你可以获取队列中消息的估计数。**QueueRestProxy-\>getQueueMetadata** 方法要求队列服务返回有关队列的元数据。对返回的对象调用 **getApproximateMessageCount** 方法将提供队列中消息的计数。此计数只是一个近似值，因为只能在队列服务响应你的请求后添加或删除消息。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try {
    // 获取队列元数据。
    $queue_metadata = $queueRestProxy->getQueueMetadata("myqueue");
    $approx_msg_count = $queue_metadata->getApproximateMessageCount();
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    echo $approx_msg_count;

## 如何：删除队列

若要删除队列及其包含的所有消息，请调用 **QueueRestProxy-\>deleteQueue** 方法。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建队列 REST 代理。
    $queueRestProxy = ServicesBuilder::getInstance()->createQueueService($connectionString);

    try {
    // 删除队列。
    $queueRestProxy->deleteQueue("myqueue");
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 后续步骤

现在，你已了解了 Azure 队列服务的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>

  [后续步骤]: #NextSteps
  [什么是队列存储]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [创建 PHP 应用程序]: #create-app
  [针对队列服务配置应用程序]: #configure-app
  [设置 Azure 存储连接]: #connection-string
  [如何：创建队列]: #create-queue
  [如何：向队列添加消息]: #add-message
  [如何：查看下一条消息]: #peek-message
  [如何：取消对下一条消息的排队]: #dequeue-message
  [如何：更改已排队消息的内容]: #change-message
  [用于取消对消息进行排队的其他选项]: #additional-options
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [1]: #next-steps
  [howto-queue-storage]: ../includes/howto-queue-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [get-client-libraries]: ../includes/get-client-libraries.md
  [require\_once]: http://www.php.net/manual/en/function.require-once.php
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
