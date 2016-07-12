<properties 
	pageTitle="通知中心 - 企业推送架构" 
	description="有关在企业环境中使用 Azure 通知中心的指南" 
	services="notification-hubs" 
	documentationCenter="" 
	authors="wesmc7777"
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="notification-hubs"
	ms.date="02/29/2016" 
	wacn.date="05/31/2016"/>

# 企业推送架构指南

当今企业正在逐渐趋向为其最终用户（外部）或员工（内部）创建移动应用程序。它们已经拥有现成的后端系统，无论是大型机还是一些 LoB 应用程序都必须集成到移动应用程序体系结构中。本指南将介绍如何最好地实现此集成，并针对常见场景建议可能的解决方案。

一个常见需求是在后端系统中发生了用户感兴趣的事件时，通过其移动应用程序向用户发送推送通知。例如，某个在 iPhone 上安装了银行应用的客户想要在记入她帐户的借方金额超过特定值时收到通知，或者在 Intranet 方案中，某个在 Windows Phone 上安装了预算审批应用的财务部门员工，希望在收到审批请求时获得通知。

银行帐户或审批处理很可能要在某个后端系统中完成，该系统必须启动到用户的推送。可能有多个此类后端系统，所有这些系统在事件触发通知时都必须生成同样的逻辑来实现推送。此处的复杂性在于将多个后端系统与单个推送系统集成在一起，在此系统中，最终用户可能已订阅不同通知，甚至可能存在多个移动应用程序，例如，对于 Intranet 移动应用来说，一个移动应用程序可能需要从多个此类后端系统接收通知。后端系统不知道或不需要知道推送语义/技术，因此在这里常见的解决方案通常是引入一个组件，该组件轮询后端系统是否有任何感兴趣的事件并负责将推送消息发送到客户端。
在这里我们将讨论一个更好的解决方案，该解决方案使用 Azure Service Bus - 主题/订阅模型，在使解决方案可缩放的同时降低了复杂性。

下面是该解决方案的一般体系结构（普遍用于多个移动应用，但在只有一个移动应用时也同样适用）

## 体系结构

![][1]

此体系结构图中的关键部分是 Azure 服务总线，它提供了主题/订阅编程模型（可在[服务总线 Pub/Sub 编程]中找到有关它的更多内容）。接收方，在本示例中是移动后端（通常是 [Azure 移动服务]，它将启动到移动应用的推送）不直接从后端系统接收消息，我们改用 [Azure 服务总线]提供的中间抽象层，移动后端通过它可从一个或多个后端系统接收消息。需要为每个后端系统（例如，帐户、人力资源、财务，这些基本上是感兴趣的“主题”，将启动要作为推送通知发送的消息）创建一个服务总线主题。后端系统会将消息发送到这些主题。移动后端可以通过创建服务总线订阅来订阅一个或多个此类主题。这将授权移动后端从相应的后端系统接收通知。移动后端将继续在其订阅上侦听消息，一条消息到达后，它将立即返回并将该消息作为通知发送到通知中心。然后，通知中心最终将该消息传送到移动应用。因此，若要汇总关键组件，我们使用了：

1. 后端系统（LoB/旧系统）
	- 创建服务总线主题
	- 发送消息
2. 移动后端
	- 创建服务订阅
	- 接收消息（来自后端系统）
	- 将通知发送到客户端（通过 Azure 通知中心）
3. 移动 应用程序
	- 接收并显示通知
		
###好处：

1. 接收方（通过通知中心的移动应用/服务）和发送方（后端系统）之间的这种解耦使得只需要最小的更改即可集成其他后端系统。
2. 这还使得采用多个移动应用的方案能够从一个或多个后端系统接收事件。  

## 示例：

###先决条件
你应完成以下教程以熟悉相关概念以及常见的创建和配置步骤：

1. [服务总线 Pub/Sub 编程] - 此教程说明了使用服务总线主题/订阅的详细信息、如何创建命名空间以包含主题/订阅、如何通过它们发送和接收消息。 
2. [通知中心 - Windows 通用教程] - 此教程说明了如何设置 Windows 应用商店应用以及如何使用通知中心注册，然后接收通知。 

###代码示例

完整的示例代码可在[通知中心示例]中找到。它分为三个组件：

1. **EnterprisePushBackendSystem**
	
	a.此项目使用 *WindowsAzure.ServiceBus* Nuget 包，并基于[服务总线 Pub/Sub 编程]构建。

	b.这是一个简单 C# 控制台应用，模拟启动要传送到移动应用的消息的 LoB 系统。
	
		static void Main(string[] args)
        {
            string connectionString =
                CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

            // Create the topic where we will send notifications
            CreateTopic(connectionString);

            // Send message
            SendMessage(connectionString);
        }
	
	c. `CreateTopic` 用于创建我们将消息发送到的服务总线主题。

        public static void CreateTopic(string connectionString)
        {
            // Create the topic if it does not exist already

            var namespaceManager =
                NamespaceManager.CreateFromConnectionString(connectionString);

            if (!namespaceManager.TopicExists(sampleTopic))
            {
                namespaceManager.CreateTopic(sampleTopic);
            }
        }

	d. `SendMessage` 用于将消息发送到此服务总线主题。出于本示例的目的，此处我们只定期向此主题发送一组随机消息。通常将有在事件发生时发送消息的后端系统。

        public static void SendMessage(string connectionString)
        {
            TopicClient client =
                TopicClient.CreateFromConnectionString(connectionString, sampleTopic);

            // Sends random messages every 10 seconds to the topic
            string[] messages = 
            {
                "Employee Id '{0}' has joined.",
                "Employee Id '{0}' has left.",
                "Employee Id '{0}' has switched to a different team."
            };

            while (true)
            {
                Random rnd = new Random();
                string employeeId = rnd.Next(10000, 99999).ToString();
                string notification = String.Format(messages[rnd.Next(0,messages.Length)], employeeId);

                // Send Notification
                BrokeredMessage message = new BrokeredMessage(notification);
                client.Send(message);

                Console.WriteLine("{0} Message sent - '{1}'", DateTime.Now, notification);

                System.Threading.Thread.Sleep(new TimeSpan(0, 0, 10));
            }
        }

2. **ReceiveAndSendNotification**

	a.此项目使用 *WindowsAzure.ServiceBus* 和 *Microsoft.Web.WebJobs.Publish* Nuget 包，并基于[服务总线 Pub/Sub 编程]构建。

	b.这是另一个 C# 控制台应用，我们将它作为 [Azure WebJob] 运行，因为它必须连续运行以侦听来自 LoB/后端系统的消息。它将是移动后端的一部分。

	    static void Main(string[] args)
	    {
	        string connectionString =
	                 CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
	
	        // Create the subscription which will receive messages
	        CreateSubscription(connectionString);   
	
	        // Receive message
	        ReceiveMessageAndSendNotification(connectionString);
	    }

	c. `CreateSubscription` 用于为后端系统将消息发送到的主题创建服务总线订阅。此组件将根据业务方案，为相应主题创建一个或多个订阅（例如，一些订阅可能从人力资源系统接收消息，一些订阅从财务系统接收消息，等等）

	    static void CreateSubscription(string connectionString)
        {
            // Create the subscription if it does not exist already
            var namespaceManager =
                NamespaceManager.CreateFromConnectionString(connectionString);

            if (!namespaceManager.SubscriptionExists(sampleTopic, sampleSubscription))
            {
                namespaceManager.CreateSubscription(sampleTopic, sampleSubscription);
            }
        }

	d.ReceiveMessageAndSendNotification 用于从使用其订阅的主题读取消息，如果读取成功，则构建要使用 Azure 通知中心发送到移动应用程序的通知（在此示例方案中为 Windows 本机 toast 通知）。

		static void ReceiveMessageAndSendNotification(string connectionString)
        {
            // Initialize the Notification Hub
            string hubConnectionString = CloudConfigurationManager.GetSetting
                    ("Microsoft.NotificationHub.ConnectionString");
            hub = NotificationHubClient.CreateClientFromConnectionString
                    (hubConnectionString, "enterprisepushservicehub");
            
            SubscriptionClient Client =
                SubscriptionClient.CreateFromConnectionString
                        (connectionString, sampleTopic, sampleSubscription);

            Client.Receive();

            // Continuously process messages received from the subscription 
            while (true)
            {
                BrokeredMessage message = Client.Receive();
                var toastMessage = @"<toast><visual><binding template=""ToastText01""><text id=""1"">{messagepayload}</text></binding></visual></toast>";

                if (message != null)
                {
                    try
                    {
                        Console.WriteLine(message.MessageId);
                        Console.WriteLine(message.SequenceNumber);
                        string messageBody = message.GetBody<string>();
                        Console.WriteLine("Body: " + messageBody + "\n");

                        toastMessage = toastMessage.Replace("{messagepayload}", messageBody);
                        SendNotificationAsync(toastMessage);

                        // Remove message from subscription
                        message.Complete();
                    }
                    catch (Exception)
                    {
                        // Indicate a problem, unlock message in subscription
                        message.Abandon();
                    }
                }
            } 
        }
        static async void SendNotificationAsync(string message)
        {
            await hub.SendWindowsNativeNotificationAsync(message);
        }

	e.若要将此项发布为 **WebJob**，请右键单击 Visual Studio 中的解决方案，然后选择“发布为 WebJob”

	![][2]

	f.选择发布配置文件并创建新的 Azure 网站（如果它尚未存在），该网站将托管此 WebJob，拥有该网站后，单击“发布”。
	
	![][3]

	g.将该作业配置为“连续运行”，以便在你登录到 [Azure 经典管理门户]时，应看到如下内容：

	![][4]


3. **EnterprisePushMobileApp**

	a.这是一个 Windows 应用商店应用程序，它将从作为移动后端的一部分运行的 WebJob 接收 toast 通知并显示它。这基于[通知中心 - Windows 通用教程]构建。
	
	b.确保应用程序已启用接收 toast 通知。

	c.确保在应用启动时（替换 *HubName* 和 *DefaultListenSharedAccessSignature* 后）调用以下通知中心注册代码：

        private async void InitNotificationsAsync()
        {
            var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

            var hub = new NotificationHub("[HubName]", "[DefaultListenSharedAccessSignature]");
            var result = await hub.RegisterNativeAsync(channel.Uri);

            // Displays the registration ID so you know it was successful
            if (result.RegistrationId != null)
            {
                var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }

### 运行示例：

1. 确保你的 WebJob 成功运行并且计划为“连续运行”。
2. 运行 **EnterprisePushMobileApp**，这将启动 Windows 应用商店应用。
3. 运行 **EnterprisePushBackendSystem** 控制台应用程序，这将模拟 LoB 后端并将开始发送消息，你应该会看到如下所示的 toast 通知：

	![][5]

4. 消息最初发送到正被 WebJob 中的订阅监视的线主题。收到消息后，将创建通知并将其发送到移动应用。当你转到 [Azure 经典管理门户]中 Web 作业的“日志”链接时，可以仔细查看 Web 作业日志来确认处理：

	![][6]

<!-- Images -->
[1]: ./media/notification-hubs-enterprise-push-architecture/architecture.png
[2]: ./media/notification-hubs-enterprise-push-architecture/WebJobsContextMenu.png
[3]: ./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png
[4]: ./media/notification-hubs-enterprise-push-architecture/WebJob.png
[5]: ./media/notification-hubs-enterprise-push-architecture/Notifications.png
[6]: ./media/notification-hubs-enterprise-push-architecture/WebJobsLog.png

<!-- Links -->
[通知中心示例]: https://github.com/Azure/azure-notificationhubs-samples
[Azure 移动服务]: /home/features/mobile-services/
[Azure 服务总线]: /documentation/articles/service-bus-fundamentals-hybrid-solutions/ 
[服务总线 Pub/Sub 编程]: /documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/
[Azure WebJob]: /documentation/articles/web-sites-create-web-jobs/
[通知中心 - Windows 通用教程]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[Azure 经典管理门户]: https://manage.windowsazure.cn/

<!---HONumber=Mooncake_0523_2016-->