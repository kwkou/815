﻿## 将消息发送到事件中心
在本节中，您将编写用于将事件发送到事件中心的 Windows 控制台应用。

1. 在 Visual Studio 中，使用"控制台应用程序"项目模板创建一个新的 Visual C# 桌面应用项目。将该项目命名为 **Sender**。

   	![][7]

2. 在"解决方案资源管理器"中，右键单击该解决方案，然后单击"为解决方案管理 NuGet 程序包..."。 

	此时将显示"管理 NuGet 程序包"对话框。

3. 搜索  `Azure Service Bus`、单击"安装"，并接受使用条款。 

	![][8]

	这样便会下载、安装 <a href="https://www.nuget.org/packages/WindowsAzure.ServiceBus/">Azure 服务总线库 NuGet 程序包</a>并添加对它的引用。

4. 在 **Program.cs** 文件顶部添加以下  `using` 语句：

		using Microsoft.ServiceBus.Messaging;

5. 将以下  `static` 字段添加到 **Program** 类，从而将值分别替换为您在上一节中创建的事件中心的名称和具有 **send** 权限的连接字符串：

		static string eventHubName = "{event hub name}";
        static string connectionString = "{send connection string}";

6. 将以下方法添加到 **Program** 类：

		static async Task SendingRandomMessages()
        {
            var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
            while (true)
            {
                try
                {
                    var message = Guid.NewGuid().ToString();
                    Console.WriteLine("{0} > Sending message: {1}", DateTime.Now.ToString(), message);
                    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                }
                catch (Exception exception)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine("{0} > Exception: {1}", DateTime.Now.ToString(), exception.Message);
                    Console.ResetColor();
                }

                await Task.Delay(200);
            }
        }

	此方法会不断地将事件发送到事件中心，迟延为 200 毫秒。

7. 最后，在 **Main** 方法中添加以下行：

		Console.WriteLine("Press Ctrl-C to stop the sender process");
        Console.WriteLine("Press Enter to start now");
        Console.ReadLine();
        SendingRandomMessages().Wait();


<!-- Images -->
[7]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp1.png
[8]: ./media/service-bus-event-hubs-getstarted/create-sender-csharp2.png
<!--HONumber=41-->
