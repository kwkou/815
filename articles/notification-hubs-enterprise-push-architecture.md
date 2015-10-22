<properties 
	pageTitle="通知中心 - 企业推送体系结构" 
	description="有关在企业环境中使用 Azure 通知中心的指南" 
	services="notification-hubs" 
	documentationCenter="" 
	authors="piyushjo" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="notification-hubs" 	
	ms.date="01/15/2015" 
	wacn.date="08/29/2015"/>

# 企业推送体系结构指南

现在，企业逐渐趋向于为其终端用户（外部）或员工（内部）创建移动应用程序。它们含有已经就位的现有后端系统，其中大型机或 LoB 应用程序必须要与移动应用程序体系结构进行集成。本指南将介绍如何以最佳方式进行集成，同时推荐适合常见场景的潜在解决方案。

在后端系统中发生相关的事件时，常见的要求是通过用户的移动应用程序向他们发送推送通知。例如，一位银行客户（其 iPhone 上安装了该行的银行应用）希望在其欠款超出特定的帐户金额之后获得通知；或者一种内部网场景：财务部门的员工（其 Windows Phone 上安装了预算审批应用）希望在获得批准请求之后获得通知。
 
银行帐户或批准处理很可能在某个后端系统中完成，而其必须向该用户发出推送。当事件触发通知时，可能有多个此类后端系统必须都要构建相同类型的逻辑来实施推送。此处的复杂性在于将若干个后端与单个推送系统进行集成，其中终端用户可能已订阅了不同的通知，甚至有多个移动应用程序。例如，在内部网移动应用的案例中，一个移动应用程序可能希望从多个此类后端系统中接收通知。后端系统不知道，也无需知道推送语义/技术，因此从传统角度来说，此处的常用解决方案是引入一种组件，这种组件可以调查所有感兴趣事件的后端系统，并负责向客户端发送推送消息。
我们将在此处探讨一种甚至更好的解决方案，即使用 Azure 服务总线 - 主题/订阅模型，这样可以降低复杂性，同时使该解决方案更具伸缩性。

以下是该解决方案的一般体系结构（虽然与多个移动应用通用，但当只有一个移动应用时也同样适用）

## 体系结构

![][1]

此体系结构图示中的主要部分是 Azure 服务总线，其提供主题/订阅编程模型（有关更多内容，请访问[服务总线发布/订阅编程])。在本案例中，接收器是移动后端（通常 [Azure 移动服务]会发起指向移动应用的推送），它不直接从后端系统接收消息；相反，我们有一个 [Azure 服务总线]提供的中间抽象层，通过它可使移动后端从一个或多个后端系统中接收消息。需要为每个后端系统创建一个服务总线主题（例如，帐户、人力资源和财务），这些是基本的相关“主题”，它们会发起作为推送通知而发送的信息。后端系统将会向这些主题发送消息。通过创建服务总线订阅，移动后端可以订阅到一个或更多此类主题。这可以让移动后端从相应的后端系统中接收通知。移动后端继续监听其订阅上的消息，而且一旦消息到达，它会返回并将其作为通知发送到它的通知中心。然后，通知中心会向该移动应用传递消息。那么，若要总结主要组件，我们拥有：

1. 后端系统（LoB/旧版系统）
	- 创建服务总线主题
	- 发送消息
2. 移动后端
	- 创建服务订阅
	- 接收消息（来自后端系统）
	- （通过 Azure 通知中心）向客户端发送通知
3. 移动应用程序
	- 接收并显示通知
		
### 优势：

1. 接收器（通过通知中心的移动应用/服务）和发送器（后端系统）之间的分离可通过最小的更改与其他后端系统进行集成。
2. 这也使得多个移动应用的场景能够从一个或多个后端系统中接收事件。  

## 示例：

### 先决条件
您应该完成以下教程，熟悉各种概念以及常用的创建和配置步骤：

1. [服务总线发布/订阅编程] - 它对使用服务总线主题/订阅、如何创建命名空间以包含主题/订阅以及如何发送和从中接收消息进行了详细阐述。 
2. [通知中心 - Windows 通用教程] - 它阐述了如何设置 Windows 应用商店应用以及如何使用通知中心来注册并接收通知。 

### 代码示例

可在[通知中心示例]中获取完整的实例代码。它可以分为三个组件：

1. **EnterprisePushBackendSystem**
	
	a.此项目使用 *WindowsAzure.ServiceBus* Nuget 包，并以[服务总线发布/订阅编程]为基础。

	b.这是一个简单的 C# 控制台应用，可以模拟 LoB 系统（该系统可以发起传递到移动应用的消息）。
	
		static void Main(string[] args)
        {
            string connectionString =
                CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

            // Create the topic where we will send notifications
            CreateTopic(connectionString);

            // Send message
            SendMessage(connectionString);
        }
	
	c. `CreateTopic` 可用于创建我们将在其中发送消息的服务总线主题。

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

	d. `SendMessage` 可用于向此服务总线主题发送消息。出于演示的目的，此处我们只是定期向该主题发送一组随机消息。通常，当事件发生时，会有一个发送消息的后端系统。

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

	a.此项目使用 *WindowsAzure.ServiceBus* 和 *Microsoft.Web.WebJobs.Publish* Nuget 包，并以[服务总线发布/订阅编程]为基础。

	b.这是另一个我们作为 [Azure WebJob] 运行的 C# 控制台应用，因为它必须连续运行以监听来自 LoB/后端系统的消息。这将是您移动后端的一部分。

	    static void Main(string[] args)
	    {
	        string connectionString =
	                 CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
	
	        // Create the subscription which will receive messages
	        CreateSubscription(connectionString);   
	
	        // Receive message
	        ReceiveMessageAndSendNotification(connectionString);
	    }

	c. `CreateSubscription` 可用于为后端系统在其中发送消息的主题创建服务总线订阅。根据企业方案的不同，此组件将创建一个或多个订阅，以对应于相应主题（例如，一些可能从人力资源系统接收消息，一些可能从财务系统接收消息等）

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

	d.ReceiveMessageAndSendNotification 可通过使用它的订阅从该主题中读取消息，并且如果成功读取，则会使用 Azure 通知中心创建一个通知（即该示例场景中的 Windows 本机 toast 通知）以发送到移动应用程序。

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

	e.为了将其作为 **WebJob** 进行发布，在 Visual Studio 中右键单击该解决方案，并选择**作为 WebJob 发布**

	![][2]

	f.选择您的发布配置文件并新建 Azure WebSite（如果不存在），其将会托管此 WebJob，并在具备 WebSite 后进行**发布**。
	
	![][3]

	g.将该作业配置为“连续运行”，以便于当您登录到 Azure 管理门户时，您应该看到类似于如下内容：

	![][4]


3. **EnterprisePushMobileApp**

	a.这是一个 Windows 应用商店应用程序，它可以从作为您的移动后端一部分运行的 WebJob 中接收 toast 通知，并显示该通知。这以[通知中心 - Windows 通用教程]为基础。
	
	b.确保已启用您的应用程序来接收 toast 通知。

	c.（在替换 *HubName* 和 *DefaultListenSharedAccessSignature*）之后，确保在应用启动时调用以下通知中心注册代码：

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

1. 确保您的 WebJob 成功运行，并且计划为“连续运行”。 
2. 运行将会启动 Windows 应用商店应用的 **EnterprisePushMobileApp**。 
3. 运行将会模拟 LoB 后端并开始发送消息的 **EnterprisePushBackendSystem** 控制台应用程序，并且您应该可以看到类似于以下消息的 toast 通知： 

	![][5]

4. 这些消息最初发送到服务总线主题，而这些主题受到 Web Job 中的服务总线订阅的监视。收到消息之后，系统会创建通知并发送到移动应用。当您转到 Web Job 的 Azure 管理门户中的日志链接时，您可以浏览 WebJob 日志以确认处理情况：

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
[Azure 移动服务]: http://www.windowsazure.cn/home/features/mobile-services/
[Azure 服务总线]: /documentation/articles/fundamentals-service-bus-hybrid-solutions
[服务总线发布/订阅编程]: /documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions
[Azure WebJob]: /documentation/articles/web-sites-create-web-jobs
[通知中心 - Windows 通用教程]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started

<!---HONumber=67-->