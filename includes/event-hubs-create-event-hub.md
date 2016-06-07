## 创建事件中心

1. 登录到 [Azure 管理门户][]，然后单击屏幕底部的“新建”。

2. 依次单击“App Service”、“服务总线”、“事件中心”、“快速创建”。

	![][1]

3. 为你的事件中心键入名称，选择所需区域，然后单击“创建新事件中心”。

	![][2]

4. 如果你未显式选择给定区域中现有的命名空间，门户会为你创建命名空间（通常为 **事件中心名称-ns**）。单击该命名空间（在此示例中为 **eventhub-ns**）。

	![][3]

5. 在页面底部，单击“连接信息”。单击复制按钮（显示在下图中），将 **RootManageSharedAccessKey** 连接字符串复制到剪贴板。保存该连接字符串，以便本教程以后使用。

	![][4]

现在，你的事件中心就创建好了，你已经有了收发事件所需的连接字符串。

[1]: ./media/event-hubs-create-event-hub/create-event-hub1.png
[2]: ./media/event-hubs-create-event-hub/create-event-hub2.png
[3]: ./media/event-hubs-create-event-hub/create-event-hub3.png
[4]: ./media/event-hubs-create-event-hub/create-conn-str1.png
<!---HONumber=Mooncake_0523_2016-->