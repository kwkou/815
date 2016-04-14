

1. 登录到 [Azure 门户](https://portal.azure.cn)，然后单击屏幕底部的“+新建”。

2. 依次单击“新建”、“Web + 移动”、“通知中心”、“快速创建”。

   	![Azure 门户 - 创建通知中心](./media/notification-hubs-portal-create-new-hub/notification-hubs-azure-portal-create.png)

3. 确保在“通知中心”字段指定一个唯一的名称。选择所需的“区域”、“订阅”和“资源组”（如果已经有一个）。
 
	如果你已经有想在其中创建中心的服务总线命名空间，则通过“命名空间”字段中的“选择现有”选项来选择它。否则，可以使用根据中心名称创建的默认名称，前提是该命名空间名称可用。

	一旦准备就绪后，单击“创建”。

   	![Azure 门户 - 设置通知中心属性](./media/notification-hubs-portal-create-new-hub/notification-hubs-azure-portal-settings.png)

4. 创建命名空间和通知中心后，就将进入相应的门户页。

   	![Azure 门户 - 通知中心门户页面](./media/notification-hubs-portal-create-new-hub/notification-hubs-azure-portal-page.png)
       
5. 单击“设置”和“访问策略”，记下都提供给你的两个连接字符串，随后将需要它们来处理推送通知。

   	![Azure 门户 - 通知中心连接字符串](./media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png)

<!---HONumber=Mooncake_0405_2016-->