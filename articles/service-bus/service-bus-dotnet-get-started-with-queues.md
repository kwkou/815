<properties
    pageTitle="服务总线队列入门 | Azure"
    description="如何编写用于服务总线消息传送的 C# 控制台应用程序"
    services="service-bus"
    documentationCenter=".net"
    authors="jtaubensee"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="08/23/2016"
    wacn.date="10/24/2016"/>

# 服务总线队列入门

[AZURE.INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

## 将要完成的任务

在本教程中我们将完成以下任务：

1. 使用 Azure 门户创建服务总线命名空间。

2. 使用 Azure 门户创建服务总线消息传递队列。

3. 编写一个控制台应用程序用于发送消息。

4. 编写一个控制台应用程序用于接收消息。

## 先决条件

1. [Visual Studio 2013 或 Visual Studio 2015](http://www.visualstudio.com)。本教程中的示例使用 Visual Studio 2015。

2. Azure 订阅。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## 使用 Azure 门户创建命名空间

如果你已创建服务总线命名空间，请跳转到[使用 Azure 门户创建队列](#2-create-a-queue-using-the-azure-portal)部分。


## <a name="2-create-a-queue-using-the-azure-portal"></a> 使用 Azure 门户创建队列

如果你已创建服务总线队列，请跳转到[将消息发送到队列](#3-send-messages-to-the-queue)部分。


## <a name="3-send-messages-to-the-queue"></a> 将消息发送到队列

为了将消息发送到队列中，我们将使用 Visual Studio 编写一个 C# 控制台应用程序。

### 创建控制台应用程序

1. 启动 Visual Studio 并创建新的控制台应用程序。

### 添加服务总线 NuGet 包

1. 右键单击新创建的项目，然后选择“管理 NuGet 包”。

2. 单击“浏览”选项卡，然后搜索“Microsoft Azure 服务总线”，并选择“Microsoft Azure 服务总线”项。单击“安装”以完成安装，然后关闭此对话框。

    ![选择 NuGet 包][nuget-pkg]

### 编写一些代码以向队列发送消息

1. 在 Program.cs 文件的顶部添加以下 using 语句。

    ```
    using Microsoft.ServiceBus.Messaging;
    ```
    
2. 将以下代码添加到 `Main` 方法，并将 **connectionString** 变量设置为创建命名空间时获取的连接字符串，以及将 **queueName** 设置为创建队列时使用的队列名称。

    ```
    var connectionString = "<Your connection string>";
    var queueName = "<Your queue name>";
  
    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
    var message = new BrokeredMessage("This is a test message!");
    client.Send(message);
    ```

    Program.cs 文件的内容如下所示。

    ```
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
    ```
  
3. 运行该程序，并检查 Azure 经典管理门户。请注意，**队列长度**值现在应为 1。
    
      ![队列长度][queue-length-send]
    
## 从队列接收消息

1. 创建新的控制台应用程序并添加对服务总线 NuGet 包的引用，类似于上面的发送应用程序。

2. 在 Program.cs 文件顶部添加以下 `using` 语句。
  
    ```
    using Microsoft.ServiceBus.Messaging;
    ```
  
3. 将以下代码添加到 `Main` 方法，并将 **connectionString** 变量设置为创建命名空间时获取的连接字符串，以及将 **queueName** 设置为创建队列时使用的队列名称。

    ```
    var connectionString = "";
    var queueName = "samplequeue";
  
    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);
  
    client.OnMessage(message =>
    {
      Console.WriteLine(String.Format("Message body: {0}", message.GetBody<String>()));
      Console.WriteLine(String.Format("Message id: {0}", message.MessageId));
    });
  
    Console.ReadLine();
    ```

	Program.cs 文件的内容如下所示：

    ```
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
    ```
  
4. 运行该程序，并检查门户。请注意，**队列长度**值现在应为 0。

    ![队列长度][queue-length-receive]
  
祝贺你！ 你已创建队列、发送和接收消息。

## 后续步骤

签出 [GitHub 存储库](https://github.com/Azure-Samples/azure-servicebus-messaging-samples)，该存储库中包含演示 Azure 服务总线消息传送的一些更高级功能的示例。

<!--Image references-->

[nuget-pkg]: ./media/service-bus-dotnet-get-started-with-queues/nuget-package.png
[queue-length-send]: ./media/service-bus-dotnet-get-started-with-queues/queue-length-send.png
[queue-length-receive]: ./media/service-bus-dotnet-get-started-with-queues/queue-length-receive.png


<!--Reference style links - using these makes the source content way more readable than using inline links-->

[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples


<!---HONumber=Mooncake_0718_2016-->