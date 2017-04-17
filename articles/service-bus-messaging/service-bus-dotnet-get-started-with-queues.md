<properties
    pageTitle="编写使用 Azure 服务总线队列的程序 | Azure"
    description="如何编写用于服务总线消息传送的 C# 控制台应用程序"
    services="service-bus"
    documentationCenter=".net"
    authors="jtaubensee"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="hero-article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="11/30/2016"
    ms.author="jotaub;sethm"
    wacn.date="04/17/2017"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="34911e5ffcf8392691c0a895c24138c4587e683a"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-with-service-bus-queues"></a>服务总线队列入门

[AZURE.INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

## <a name="what-will-be-accomplished"></a>将要完成的任务

在本教程中我们将完成以下任务：

1. 使用 Azure 门户创建服务总线命名空间。

2. 使用 Azure 门户创建服务总线消息传递队列。

3. 编写一个控制台应用程序用于发送消息。

4. 编写一个控制台应用程序用于接收消息。

## <a name="prerequisites"></a>先决条件
1. [Visual Studio 2015 或更高版本](http://www.visualstudio.com)。 本教程中的示例使用 Visual Studio 2015。
2. Azure 订阅。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## <a name="1-create-a-namespace-using-the-azure-portal"></a>1.使用 Azure 门户创建命名空间

如果你已创建服务总线命名空间，请跳转到 [使用 Azure 门户创建队列](#2-create-a-queue-using-the-azure-portal) 部分。

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="2-create-a-queue-using-the-azure-portal"></a>2.使用 Azure 门户创建队列

如果你已创建服务总线队列，请跳转到 [将消息发送到队列](#3-send-messages-to-the-queue) 部分。

[AZURE.INCLUDE [service-bus-create-queue-portal](../../includes/service-bus-create-queue-portal.md)]

## <a name="3-send-messages-to-the-queue"></a>3.将消息发送到队列

为了将消息发送到队列中，我们将使用 Visual Studio 编写一个 C# 控制台应用程序。

### <a name="create-a-console-application"></a>创建控制台应用程序

1. 启动 Visual Studio 并创建新的控制台应用程序。

### <a name="add-the-service-bus-nuget-package"></a>添加服务总线 NuGet 包

1. 右键单击新创建的项目，然后选择“管理 NuGet 包” 。

2. 单击“浏览”选项卡，然后搜索“Microsoft Azure 服务总线”，并选择“Microsoft Azure 服务总线”项。 单击“安装”以完成安装，然后关闭此对话框。

    ![选择 NuGet 包][nuget-pkg]

### <a name="write-some-code-to-send-a-message-to-the-queue"></a>编写一些代码以向队列发送消息

1. 在 Program.cs 文件的顶部添加以下 using 语句。

        using Microsoft.ServiceBus.Messaging;

2. 将以下代码添加到 `Main` 方法，并将 **connectionString** 变量设置为创建命名空间时获取的连接字符串，以及将 **queueName** 设置为创建队列时使用的队列名称。

        var connectionString = "<Your connection string>";
        var queueName = "<Your queue name>";

        var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
        var message = new BrokeredMessage("This is a test message!");
        client.Send(message);

    Program.cs 文件的内容如下所示。

        using System;
        using Microsoft.ServiceBus.Messaging;

        namespace GettingStartedWithQueues
        {
            class Program
            {
                static void Main(string[] args)
                {
                    var connectionString = "<Your connection string>";
                    var queueName = "<Your queue name>";

                    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
                    var message = new BrokeredMessage("This is a test message!");

                    client.Send(message);
                }
            }
        }

3. 运行该程序，并检查 Azure 门户。 单击命名空间“概述”  边栏选项卡中的队列名称。 请注意，“活动消息计数”  值现在应为 1。

      ![消息计数][queue-message]

## <a name="4-receive-messages-from-the-queue"></a>4.从队列接收消息

1. 创建新的控制台应用程序并添加对服务总线 NuGet 程序包的引用，类似于前面的发送应用程序。

2. 在 Program.cs 文件顶部添加以下 `using` 语句。

        using Microsoft.ServiceBus.Messaging;

3. 将以下代码添加到 `Main` 方法，并将 **connectionString** 变量设置为创建命名空间时获取的连接字符串，以及将 **queueName** 设置为创建队列时使用的队列名称。

        var connectionString = "";
        var queueName = "samplequeue";

        var client = QueueClient.CreateFromConnectionString(connectionString, queueName);

        client.OnMessage(message =>
        {
          Console.WriteLine(String.Format("Message body: {0}", message.GetBody<String>()));
          Console.WriteLine(String.Format("Message id: {0}", message.MessageId));
        });

        Console.ReadLine();

    Program.cs 文件的内容如下所示：

        using System;
        using Microsoft.ServiceBus.Messaging;

        namespace GettingStartedWithQueues
        {
          class Program
          {
            static void Main(string[] args)
            {
              var connectionString = "";
              var queueName = "samplequeue";

              var client = QueueClient.CreateFromConnectionString(connectionString, queueName);

              client.OnMessage(message =>
              {
                Console.WriteLine(String.Format("Message body: {0}", message.GetBody<String>()));
                Console.WriteLine(String.Format("Message id: {0}", message.MessageId));
              });

              Console.ReadLine();
            }
          }
        }

4. 运行该程序，并检查门户。 请注意，“队列长度”  值现在应为 0。

    ![队列长度][queue-message-receive]

祝贺你！ 你已创建队列、发送和接收消息。

## <a name="next-steps"></a>后续步骤

查看 [GitHub 存储库](https://github.com/Azure-Samples/azure-servicebus-messaging-samples) 中的示例，了解 Azure 服务总线消息传送的一些更高级的功能。

<!--Image references-->

[nuget-pkg]: ./media/service-bus-dotnet-get-started-with-queues/nuget-package.png
[queue-message]: ./media/service-bus-dotnet-get-started-with-queues/queue-message.png
[queue-message-receive]: ./media/service-bus-dotnet-get-started-with-queues/queue-message-receive.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples


<!--Update_Description:update wording and code-->