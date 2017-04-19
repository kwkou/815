<properties
    pageTitle="使用 .NET Standard 从 Azure 事件中心接收事件 | Azure"
    description="使用 .NET Standard 中的 EventProcessorHost 接收消息入门"
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
    ms.date="03/03/2017"
    wacn.date="04/17/2017"
    ms.author="jotaub;sethm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="e57cf8307c55b7ddc4ab6a509500a292f9a80dc6"
    ms.lasthandoff="04/07/2017" />

# <a name="get-started-receiving-messages-with-the-event-processor-host-in-net-standard"></a>使用 .NET Standard 中的事件处理程序主机接收消息入门

> [AZURE.NOTE]
> [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleEphReceiver) 上提供了此示例。

本教程显示如何编写 .NET Core 控制台应用程序，以使用 **EventProcessorHost** 从事件中心接收消息。 可以按原样运行 [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleEphReceiver) 解决方案，并将字符串替换为事件中心和存储帐户的值，也可以按照本教程中的步骤创建自己的解决方案。 

## <a name="prerequisites"></a>先决条件

1. [Microsoft Visual Studio 2015 或 2017](http://www.visualstudio.com)。 本教程中的示例使用 Visual Studio 2015，但也支持 Visual Studio 2017。

2. [.NET Core Visual Studio 2015 或 2017 工具](http://www.microsoft.com/net/core)。

3. Azure 订阅。

4. 事件中心命名空间。

5. 一个 Azure 存储帐户。

## <a name="create-an-event-hubs-namespace-and-an-event-hub"></a>创建事件中心命名空间和事件中心  

第一步是使用 [Azure 门户](https://portal.azure.cn)创建事件中心类型的命名空间，并获取应用程序与事件中心进行通信所需的管理凭据。 若要创建命名空间和事件中心，请按照[本文](/documentation/articles/event-hubs-create/)中的步骤进行操作，然后继续执行以下步骤。  

## <a name="create-an-azure-storage-account"></a>创建 Azure 存储帐户  

1. 登录到 [Azure 门户](https://portal.azure.cn)。  
2. 在门户的左侧导航窗格中，依次单击**新建**、**存储**和**存储帐户**。  
3. 完成“存储帐户”边栏选项卡中的字段，然后单击**创建**。

    ![][1]

4. 看到**部署成功**消息后，单击新存储帐户名，然后在**概要**边栏选项卡中单击**Blob**。 **Blob 服务**边栏选项卡打开时，单击顶部的**+ 容器**。 为容器指定名称，然后关闭**Blob 服务**边栏选项卡。  
5. 单击左侧边栏选项卡中的**访问密钥**，复制存储容器、存储帐户的名称和 **key1** 的值。 将这些值保存到记事本或其他临时位置。  

## <a name="create-a-console-application"></a>创建控制台应用程序

1. 启动 Visual Studio。 在“文件”菜单中，单击“新建”，然后单击“项目”。 创建 .NET Core 控制台应用程序。

    ![][2]

2. 在解决方案资源管理器中，双击 **project.json** 文件以在 Visual Studio 编辑器中将其打开。

3. 在 `"frameworks` 节中将字符串 `"portable-net45+win8"` 添加到 `"imports"` 声明。 该节现在应如下所示。 由于 Azure 存储依赖于 OData，因此该字符串是必需的：

        "frameworks": {
          "netcoreapp1.0": {
            "imports": [
              "dnxcore50",
              "portable-net45+win8"
            ]
          }
        }

4. 在“文件”菜单中，单击“全部保存”。

请注意，本教程演示如何编写 .NET Core 应用程序，但如果要针对的是完整 .NET Framework，请将以下代码行添加到 project.json 文件中的 `"frameworks"` 节：

    "net451": {
    },

## <a name="add-the-event-hubs-nuget-package"></a>添加事件中心 NuGet 包

* 将以下 NuGet 包添加到项目：
  * [`Microsoft.Azure.EventHubs`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/)
  * [`Microsoft.Azure.EventHubs.Processor`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/)

## <a name="implement-the-ieventprocessor-interface"></a>实现 IEventProcessor 接口

1. 在“解决方案资源管理器”中，右键单击该项目，单击“添加”，然后单击“类”。 将新类命名为 **SimpleEventProcessor**。

2. 打开 SimpleEventProcessor.cs 文件，并将以下 `using` 语句添加到文件顶部。

        using System.Text;
        using Microsoft.Azure.EventHubs;
        using Microsoft.Azure.EventHubs.Processor;

3. 实现 `IEventProcessor` 接口。 将 `SimpleEventProcessor` 类的全部内容替换为以下代码：

        public class SimpleEventProcessor : IEventProcessor
        {
            public Task CloseAsync(PartitionContext context, CloseReason reason)
            {
                Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
                return Task.CompletedTask;
            }

            public Task OpenAsync(PartitionContext context)
            {
                Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
                return Task.CompletedTask;
            }

            public Task ProcessErrorAsync(PartitionContext context, Exception error)
            {
                Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
                return Task.CompletedTask;
            }

            public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
            {
                foreach (var eventData in messages)
                {
                    var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
                    Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
                }

                return context.CheckpointAsync();
            }
        }

## <a name="write-a-main-console-method-that-uses-the-simpleeventprocessor-class-to-receive-messages"></a>编写使用 SimpleEventProcessor 类接收消息的主控制台方法

1. 在 Program.cs 文件顶部添加以下 `using` 语句。

        using Microsoft.Azure.EventHubs;
        using Microsoft.Azure.EventHubs.Processor;

2. 向 `Program` 类添加常量作为事件中心连接字符串、事件中心名称、存储容器名称、存储帐户名称和存储帐户密钥。 添加以下代码，并将占位符替换为其对应的值。

        private const string EhConnectionString = "{Event Hubs connection string}";
        private const string EhEntityPath = "{Event Hub path/name}";
        private const string StorageContainerName = "{Storage account container name}";
        private const string StorageAccountName = "{Storage account name}";
        private const string StorageAccountKey = "{Storage account key}";

        private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

3. 将名为 `MainAsync` 的新方法添加到 `Program` 类，如下所示：

        private static async Task MainAsync(string[] args)
        {
            Console.WriteLine("Registering EventProcessor...");

            var eventProcessorHost = new EventProcessorHost(
                EhEntityPath,
                PartitionReceiver.DefaultConsumerGroupName,
                EhConnectionString,
                StorageConnectionString,
                StorageContainerName);

            // Registers the Event Processor Host and starts receiving messages
            await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

            Console.WriteLine("Receiving. Press ENTER to stop worker.");
            Console.ReadLine();

            // Disposes of the Event Processor Host
            await eventProcessorHost.UnregisterEventProcessorAsync();
        }

3. 在 `Main` 方法中添加以下代码行：

        MainAsync(args).GetAwaiter().GetResult();

    Program.cs 文件的内容如下所示：

        namespace SampleEphReceiver
        {

            public class Program
            {
                private const string EhConnectionString = "{Event Hubs connection string}";
                private const string EhEntityPath = "{Event Hub path/name}";
                private const string StorageContainerName = "{Storage account container name}";
                private const string StorageAccountName = "{Storage account name}";
                private const string StorageAccountKey = "{Storage account key}";

                private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

                public static void Main(string[] args)
                {
                    MainAsync(args).GetAwaiter().GetResult();
                }

                private static async Task MainAsync(string[] args)
                {
                    Console.WriteLine("Registering EventProcessor...");

                    var eventProcessorHost = new EventProcessorHost(
                        EhEntityPath,
                        PartitionReceiver.DefaultConsumerGroupName,
                        EhConnectionString,
                        StorageConnectionString,
                        StorageContainerName);

                    // Registers the Event Processor Host and starts receiving messages
                    await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

                    Console.WriteLine("Receiving. Press ENTER to stop worker.");
                    Console.ReadLine();

                    // Disposes of the Event Processor Host
                    await eventProcessorHost.UnregisterEventProcessorAsync();
                }
            }
        }

4. 运行程序，并确保没有任何错误。

祝贺你！ 现在你已使用事件处理程序主机从事件中心接收消息。

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

[1]: ./media/event-hubs-dotnet-standard-getstarted-receive-eph/event-hubs-python1.png
[2]: ./media/event-hubs-dotnet-standard-getstarted-receive-eph/netcore.png


<!--Update_Description:update meta properties;wording update;update content about how to create a console application-->