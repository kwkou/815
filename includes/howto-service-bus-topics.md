## 什么是服务总线主题和订阅？

Service Bus 主题和订阅支持*发布/订阅*消息通信模型。在使用主题和订阅时，分布式应用程序的组件不会直接相互通信，而是通过充当中介的主题交换消息。

![TopicConcepts](./media/howto-service-bus-topics/sb-topics-01.png)

与每条消息由单个使用方处理的 Service Bus 队列相比，主题和订阅通过发布/订阅模式提供“一对多”通信方式。可向一个主题注册多个订阅。当消息发送到主题时，每个订阅会分别对该消息进行处理。

主题订阅类似于接收发送至该主题的消息副本的虚拟队列。您可以选择基于每个订阅注册主题的筛选规则，这样就可以筛选/限制哪些主题订阅接收发送至某个主题的哪些消息。

利用 Service Bus 主题和订阅，您可以进行扩展以处理跨大量用户和应用程序的许多消息。

## 创建服务命名空间

若要开始在 Azure 中使用服务总线主题和订阅，必须先创建一个服务命名空间。服务命名空间提供了用于对应用程序中的服务总线资源进行寻址的范围容器。

创建服务命名空间：

1.  登录到 [Azure 管理门户][]。

2.  在管理门户的左侧导航窗格中，单击“Service Bus”。

3.  在管理门户的下方窗格中，单击“创建”。

	![][0]

4.  在“添加新命名空间”对话框中，输入命名空间名称。系统会立即检查该名称是否可用。

    ![][2]

5.  在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。

	重要说明：选取要部署应用程序的**相同区域**。这将为您提供最佳性能。

6. 	将对话框中的其他字段保留其默认值（“消息传递”和“标准层”），然后单击复选标记。系统现已创建您的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

	![][6]

## 获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建主题或订阅），则必须从获取该命名空间的管理凭据。你可以从 Azure 管理门户或从 Visual Studio 服务器资源管理器获取这些凭据。

###从门户中获取管理凭据

1.  在左侧导航窗格中，单击“Service Bus”节点以显示可用命名空间的列表：
	![][0]

2.  从显示的列表中选择刚刚创建的命名空间：

	![][3]

3.  单击“连接信息”。

	![][4]

4.  在“访问连接信息”对话框中，找到包含 SAS 密钥和密钥名称的连接字符串。记下这些值，因为你稍后将使用此信息来对命名空间执行操作。

###从服务器资源管理器中获取管理凭据

若要使用 Visual Studio 而非管理门户来获取连接信息，请按[此处](http://http://msdn.microsoft.com/zh-cn/library/windowsazure/ff687127.aspx)所述过程进行操作，详见**从 Visual Studio 连接到 Azure** 部分。当你登录到 Azure 时，服务器资源管理器中“Microsoft Azure”树下的“服务总线”节点中会自动填充你所创建的任何命名空间。右键单击任意命名空间，然后单击“属性”，此时就会看到在 Visual Studio 的“属性”窗格中显示与该命名空间关联的连接字符串和其他元数据。

记下 **SharedAccessKey** 值，或将其复制到剪贴板：

![][34]

 
  [Azure 管理门户]: http://manage.windowsazure.cn
  [0]: ./media/howto-service-bus-topics/sb-queues-13.png
  [2]: ./media/howto-service-bus-topics/sb-queues-04.png
  [3]: ./media/howto-service-bus-topics/sb-queues-09.png
  [4]: ./media/howto-service-bus-topics/sb-queues-06.png
  
  [6]: ./media/howto-service-bus-topics/getting-started-multi-tier-27.png
  [34]: ./media/howto-service-bus-topics/VSProperties.png

<!---HONumber=74-->