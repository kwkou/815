<properties
    pageTitle="使用 .NET Standard 将事件发送到 Azure 事件中心 | Azure"
    description="在 .NET Standard 中将事件发送到事件中心入门"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/30/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub" />  


# 在 .NET Standard 中将消息发送到事件中心入门

> [AZURE.NOTE]
[GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender) 上提供了此示例。

## 将要完成的任务

本教程演示如何创建现有解决方案 **SampleSender**（在此文件夹内）。可以按原样运行该解决方案，并将 `EhConnectionString`、`EhEntityPath` 和 `StorageAccount` 字符串替换为事件中心的值，也可以按照本教程中的步骤创建自己的解决方案。

在本教程中，我们会编写一个 .NET Core 控制台应用程序，以将消息发送到事件中心。

## 先决条件

1. [Visual Studio 2015](http://www.visualstudio.com)。

2. [.NET Core Visual Studio 2015 工具](http://www.microsoft.com/net/core)。

3. Azure 订阅。

4. 事件中心命名空间。

## 将消息发送到事件中心

为了将消息发送到事件中心，我们会使用 Visual Studio 编写一个 C# 控制台应用程序。

### 创建控制台应用程序

* 启动 Visual Studio 并创建新的 .NET Core 控制台应用程序。

### 添加事件中心 NuGet 程序包

1. 右键单击新创建的项目，然后选择“管理 NuGet 程序包”。

2. 单击“浏览”选项卡，然后搜索“Azure 事件中心”并选择“Azure 事件中心”项。单击“安装”以完成安装，然后关闭此对话框。

### 编写一些代码将消息发送到事件中心

1. 在 Program.cs 文件顶部添加以下 `using` 语句。

        using Microsoft.Azure.EventHubs;

2. 向 `Program` 类添加常量作为事件中心连接字符串和实体路径（单个事件中心名称）。将括号中的占位符替换为在创建事件中心时获得的相应值。

        private static EventHubClient eventHubClient;
        private const string EhConnectionString = "{Event Hubs connection string}";
        private const string EhEntityPath = "{Event Hub path/name}";

3. 向 `Program` 类添加名为 `MainAsync` 的新方法，如下所示：

        private static async Task MainAsync(string[] args)
        {
            // Creates an EventHubsConnectionStringBuilder object from a the connection string, and sets the EntityPath.
            // Typically the connection string should have the Entity Path in it, but for the sake of this simple scenario
            // we are using the connection string from the namespace.
            var connectionStringBuilder = new EventHubsConnectionStringBuilder(EhConnectionString)
            {
                EntityPath = EhEntityPath
            };

            eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

            await SendMessagesToEventHub(100);

            await eventHubClient.CloseAsync();

            Console.WriteLine("Press any key to exit.");
            Console.ReadLine();
        }

4. 向 `Program` 类添加名为 `SendMessagesToEventHub` 的新方法，如下所示：

        // Creates an Event Hub client and sends 100 messages to the event hub.
        private static async Task SendMessagesToEventHub(int numMessagesToSend)
        {
            for (var i = 0; i < numMessagesToSend; i++)
            {
                try
                {
                    var message = $"Message {i}";
                    Console.WriteLine($"Sending message: {message}");
                    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                }
                catch (Exception exception)
                {
                    Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
                }

                await Task.Delay(10);
            }

            Console.WriteLine($"{numMessagesToSend} messages sent.");
        }

5. 向 `Program` 类中的 `Main` 方法添加以下代码。

        MainAsync(args).GetAwaiter().GetResult();

    Program.cs 文件的内容如下所示。

        namespace SampleSender
        {
            using System;
            using System.Text;
            using System.Threading.Tasks;
            using Microsoft.Azure.EventHubs;
    
            public class Program
            {
                private static EventHubClient eventHubClient;
                private const string EhConnectionString = "{Event Hubs connection string}";
                private const string EhEntityPath = "{Event Hub path/name}";
    
                public static void Main(string[] args)
                {
                    MainAsync(args).GetAwaiter().GetResult();
                }
    
                private static async Task MainAsync(string[] args)
                {
                    // Creates an EventHubsConnectionStringBuilder object from a the connection string, and sets the EntityPath.
                    // Typically the connection string should have the Entity Path in it, but for the sake of this simple scenario
                    // we are using the connection string from the namespace.
                    var connectionStringBuilder = new EventHubsConnectionStringBuilder(EhConnectionString)
                    {
                        EntityPath = EhEntityPath
                    };
    
                    eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
    
                    await SendMessagesToEventHub(100);
    
                    await eventHubClient.CloseAsync();
    
                    Console.WriteLine("Press any key to exit.");
                    Console.ReadLine();
                }
    
                // Creates an Event Hub client and sends 100 messages to the event hub.
                private static async Task SendMessagesToEventHub(int numMessagesToSend)
                {
                    for (var i = 0; i < numMessagesToSend; i++)
                    {
                        try
                        {
                            var message = $"Message {i}";
                            Console.WriteLine($"Sending message: {message}");
                            await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                        }
                        catch (Exception exception)
                        {
                            Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
                        }
    
                        await Task.Delay(10);
                    }
    
                    Console.WriteLine($"{numMessagesToSend} messages sent.");
                }
            }
        }

6. 运行程序，并确保不引发任何错误。
  
祝贺你！ 现在已将消息发送到事件中心。

## 后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [从事件中心接收事件](/documentation/articles/event-hubs-dotnet-standard-getstarted-receive-eph/)
* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:new article about how to send event to Azure event hubs with dotnet standard -->