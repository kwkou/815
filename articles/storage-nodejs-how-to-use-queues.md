<properties linkid="dev-nodejs-how-to-service-bus-queues" urlDisplayName="Queue Service" pageTitle="How to use the queue service (Node.js) | Microsoft Azure" metaKeywords="Azure Queue Service get messages Node.js" description="Learn how to use the Azure Queue service to create and delete queues, and insert, get, and delete messages. Samples written in Node.js." metaCanonical="" services="storage" documentationCenter="Node.js" title="How to Use the Queue Service from Node.js" authors="larryfr" solutions="" manager="" editor="" />

# 如何从 Node.js 使用队列服务

本指南将演示如何使用 Windows Azure 队列服务执行常见方案。
示例是用 Node.js API 编写的。
涉及的方案包括**插入**、**扫视**、**获取**
和**删除**队列消息以及
**创建和删除队列**。有关队列的详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是队列服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Node.js 应用程序][]
-   [配置应用程序以访问存储][]
-   [设置 Azure 存储连接字符串][]
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

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][]、[Node.js 云服务][]（使用 Windows PowerShell）或[使用 WebMatrix 构建网站][]。

## 配置应用程序以访问存储

若要使用 Azure 存储空间，你需要下载并使用 Node.js azure 包，
其中包括一组便于与存储 REST 服务
进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 **npm install azure**，这应该产生
    以下输出：

        azure@0.7.5 node_modules\azure
        |-- dateformat@1.0.2-1.2.3
        |-- xmlbuilder@0.4.2
        |-- node-uuid@1.2.0
        |-- mime@1.2.9
        |-- underscore@1.4.4
        |-- validator@1.1.1
        |-- tunnel@0.0.2
        |-- wns@0.5.3
        |-- xml2js@0.2.7 (sax@0.5.2)
        |-- request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)

3.  可以手动运行 **ls** 命令来验证是否创建了
    **node\_modules** 文件夹。在该文件夹中，你将
    找到 **azure** 包，其中包含你访问存储
    所需的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到你要在其中使用存储的
应用程序的 **server.js** 文件的顶部：

    var azure = require('azure');

## 设置 Azure 存储连接

azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY 以获取连接到你的 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 **createQueueService** 时必须指定帐户信息。

有关在 Azure 云服务的配置文件中设置环境变量的示例，请参阅[使用存储构建 Node.js 云服务][]。

有关在管理门户中为 Azure 网站设置环境变量的示例，请参阅[使用存储构建 Node.js Web 应用程序][]。

## 如何：创建队列

以下代码将创建一个 **QueueService** 对象，你可通过该对象来
操作队列。

    var queueService = azure.createQueueService();

使用 **createQueueIfNotExists** 方法，该方法将返回指定队列
（如果它存在），或创建具有指定名称的新队列
（如果它尚不存在）。

    queueService.createQueueIfNotExists(queueName, function(error){
    if(!error){
    // 队列退出
        }
    });

### 筛选器

可以向使用 **QueueService** 执行的操作应用可选的筛选操作。筛选操作可以包括日志记录、自动重试等。筛选器是实现了具有签名的方法的对象：

        function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用“next”并且传递具有以下签名的回调：

        function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 **ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。下面的代码将创建一个 **QueueService** 对象，该对象使用 **ExponentialRetryPolicyFilter**：

    var retryOperations = new azure.ExponentialRetryPolicyFilter();
    var queueService = azure.createQueueService().withFilter(retryOperations);

## 如何：在队列中插入消息

若要在队列中插入消息，可使用 **createMessage** 方法创建一条
新消息并将其添加到队列中。

    queueService.createMessage(queueName, "Hello world!", function(error){
    if(!error){
    // 消息已插入
        }
    });

## 如何：扫视下一条消息

通过调用 **peekMessages** 方法，你可以扫视队列前面的
消息，而不会从队列中删除它。默认情况下，
**peekMessages** 扫视单条消息。

    queueService.peekMessages(queueName, function(error, messages){
    if(!error){
    // 消息已扫视
    // 文本位于 messages[0].messagetext 中
        }
    });

> [WACOM.NOTE]
> 当队列中没有消息时使用 **peekMessage** 不会返回错误，但也不会返回消息。

## 如何：取消对下一条消息的排队

你的代码分两步从队列中删除消息。在调用
**getMessages** 时，默认情况下你会获得队列中的下一条消息。对于
从该队列读取消息的任何其他代码，从 **getMessages** 返回的
消息将变得不可见。默认情况下，此消息将持续 30 秒
不可见。若要从队列中删除消息，你还必须
调用 **deleteMessage**。此删除消息的两步过程可确保
当你的代码因硬件或软件故障而无法处理消息时，
你的代码的另一实例可以获取同一消息
并重试。你的代码在处理消息后立即
调用 **deleteMessage**。

    queueService.getMessages(queueName, function(error, messages){
    if(!error){
    // 在不到 30 秒的时间内处理消息，消息
    // 文本位于 messages[0].messagetext 中 
    var message = messages[0]
    queueService.deleteMessage(queueName
    , message.messageid
    , message.popreceipt
    , function(error){
    if(!error){
    // 消息已删除
                    }
                });
        }
    });

> [WACOM.NOTE]
> 当队列中没有消息时使用 **getMessages** 不会返回错误，但也不会返回消息。

## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息
表示工作任务，则可以使用此功能来更新该工作任务的状态。
以下代码使用 **updateMessage** 方法来
更新消息。

    queueService.getMessages(queueName, function(error, messages){
    if(!error){
    // 获取消息
    var message = messages[0];
    queueService.updateMessage(queueName
    , message.messageid
    , message.popreceipt
                , 10
    , { messagetext:'in your message, doing stuff.' }
    , function(error){
    if(!error){
    // 消息已成功更新
                    }
                });
        }
    });

## 如何：用于对消息取消排队的其他方法

你可以通过两种方式自定义队列的消息检索。
首先，你可以获取一批消息（最多 32 条）。其次，你可以
设置更长或更短的不可见超时，从而允许你的代码使用更多或
更少的时间来彻底处理每条消息。以下代码示例使用
**getMessages** 方法通过一次调用获取 15 条消息。然后，它使用 for 循环
来处理每条消息。它还将每条消息的不可见超时时间设置为
 5 分钟。

    queueService.getMessages(queueName
    , {numofmessages:15, visibilitytimeout: 5 * 60}
    , function(error, messages){
    if(!error){
    // 消息已检索
    for(var index in messages){
    // 文本位于 messages[index].messagetext 中
    var message = messages[index];
    queueService.deleteMessage(queueName
    , message.messageid
    , message.popreceipt
    , function(error){
    if(!error){
    // 消息已删除
                        }
                    });
            }
        }
    });

## 如何：获取队列长度

你可以获取队列中消息的估计数。
**getQueueMetadata** 方法可要求队列服务返回有关队列的元数据，
而响应的 **approximatemessagecount** 属性
包含队列中的消息计数。此计数只是
一个近似值，因为只能在队列服务响应你的请求后
添加或删除消息。

    queueService.getQueueMetadata(queueName, function(error, queueInfo){
    if(!error){
    // 队列长度位于 queueInfo.approximatemessagecount 中
        }
    });

## 如何：删除队列

若要删除队列及其包含的所有消息，请对队列对象
调用 **deleteQueue** 方法。

    queueService.deleteQueue(queueName, function(error){
    if(!error){
    // 队列已删除
        }
    });

## 后续步骤

现在，你已了解有关队列存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]。
-   访问 [Azure 存储空间团队博客][]。
-   访问 GitHub 上的 [Azure SDK for Node][] 存储库。

  [后续步骤]: #next-steps
  [什么是队列服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [创建 Node.js 应用程序]: #create-app
  [配置应用程序以访问存储]: #configure-access
  [设置 Azure 存储连接字符串]: #setup-connection-string
  [如何：创建队列]: #create-queue
  [如何：在队列中插入消息]: #insert-message
  [如何：扫视下一条消息]: #peek-message
  [如何：取消对下一条消息的排队]: #get-message
  [如何：更改已排队消息的内容]: #change-contents
  [如何：用于对消息取消排队的其他方法]: #advanced-get
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [howto-queue-storage]: ../includes/howto-queue-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /en-us/documentation/articles/web-sites-nodejs-develop-deploy-mac/
  [Node.js 云服务]: /en-us/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [使用 WebMatrix 构建网站]: /en-us/documentation/articles/web-sites-nodejs-use-webmatrix/
  [使用存储构建 Node.js 云服务]: /en-us/documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
  [使用存储构建 Node.js Web 应用程序]: /en-us/documentation/articles/storage-nodejs-use-table-storage-web-site/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure SDK for Node]: https://github.com/WindowsAzure/azure-sdk-for-node
