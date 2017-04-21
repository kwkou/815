<properties
    pageTitle="使用 .NET Standard 将事件发送到 Azure 事件中心 | Azure"
    description="在 .NET Standard 中将事件发送到事件中心入门"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/01/2017"
    wacn.date="04/17/2017"
    ms.author="jotaub"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="5772c3723cfb9ba827d9ccd2ad4c05fe4ea854d2"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-sending-messages-to-event-hubs-in-net-standard"></a>在 .NET Standard 中将消息发送到事件中心入门

> [AZURE.NOTE]
> [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender) 上提供了此示例。

本教程演示如何编写 .NET Core 控制台应用程序，以将一组消息发送到事件中心。 可以按原样运行 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender) 解决方案，并将 `EhConnectionString` 和 `EhEntityPath` 字符串替换为事件中心的值，也可以按照本教程中的步骤创建自己的解决方案。

## <a name="prerequisites"></a>先决条件

1. [Microsoft Visual Studio 2015 或 2017](http://www.visualstudio.com)。 本教程中的示例使用 Visual Studio 2015，但也支持 Visual Studio 2017。

2. [.NET Core Visual Studio 2015 或 2017 工具](http://www.microsoft.com/net/core)。

3. Azure 订阅。

4. 事件中心命名空间。

为将消息发送到事件中心，我们将使用 Visual Studio 编写 C# 控制台应用程序。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>创建事件中心命名空间和事件中心

第一步是使用 [Azure 门户](https://portal.azure.cn)创建事件中心类型的命名空间，并获取应用程序与事件中心进行通信所需的管理凭据。 若要创建命名空间和事件中心，请按照[本文](/documentation/articles/event-hubs-create/)中的步骤进行操作，然后继续执行以下步骤。

## <a name="create-a-console-application"></a>创建控制台应用程序

启动 Visual Studio。 在“文件”菜单中，单击“新建”，然后单击“项目”。 创建 .NET Core 控制台应用程序。

![][1]

## <a name="add-the-event-hubs-nuget-package"></a>添加事件中心 NuGet 程序包

将 [`Microsoft.Azure.EventHubs`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) NuGet 包添加到项目。

## <a name="write-some-code-to-send-messages-to-the-event-hub"></a>编写一些代码将消息发送到事件中心

1. 在 Program.cs 文件顶部添加以下 `using` 语句。

        using Microsoft.Azure.EventHubs;
        using System.Text;

2. 向 `Program` 类添加常量作为事件中心连接字符串和实体路径（单个事件中心名称）。将括号中的占位符替换为在创建事件中心时获得的相应值。

        private static EventHubClient eventHubClient;
        private const string EhConnectionString = "{Event Hubs connection string}";
        private const string EhEntityPath = "{Event Hub path/name}";

3. 将名为 `MainAsync` 的新方法添加到 `Program` 类，如下所示：

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

            Console.WriteLine("Press ENTER to exit.");
            Console.ReadLine();
        }

4. 将名为 `SendMessagesToEventHub` 的新方法添加到 `Program` 类，如下所示：

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

                    Console.WriteLine("Press ENTER to exit.");
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

6. 运行程序，并确保没有任何错误。

祝贺你！ 现在已将消息发送到事件中心。

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [从事件中心接收事件](/documentation/articles/event-hubs-dotnet-standard-getstarted-receive-eph/)
* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)
<!-- Image Reference-->
[1]： ./media/event-hubs-dotnet-standard-getstarted-send/netcore.png
    
<!--Update_Description:update meta properties;wording update;add content about hot to create a console application-->