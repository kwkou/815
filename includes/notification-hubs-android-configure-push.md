

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)，然后单击屏幕底部的"+新建"。

2. 依次单击"应用服务"、**Service Bus**、"通知中心"和"快速创建"。

3. 键入通知中心的名称，选择所需的区域，然后单击"创建新的通知中心"。

   	![Set notification hub properties](./media/notification-hubs-android-configure-push/notification-hub-create-from-portal2.png)

4. 单击刚刚创建的命名空间（通常为***通知中心名称*-ns**），然后单击顶部的"配置"。

5. 单击顶部的"通知中心"选项卡，然后单击你刚刚创建的通知中心。

6. 单击顶部的"配置"选项卡，输入在上一部分中获得的"API 密钥"值，然后单击"保存"。

   	![Set the GCM API key in the Configure tab](./media/notification-hubs-android-configure-push/notification-hub-configure-android.png)

7. 选择顶部的"仪表板"选项卡，然后单击"查看连接字符串"。记下两个连接字符串。

你的通知中心现在已配置为使用 GCM，并且你有连接字符串，既可用于注册你的应用以接收通知，又可用于发送推送通知。

<!--HONumber=51-->