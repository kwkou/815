<h2><a name="what-are-service-bus-topics"></a>什么是 Service Bus 主题和订阅</h2>

Service Bus 主题和订阅支持发布/订阅消息通信模型。在使用主题和订阅时，分布式应用
程序的组件不会直接相互通信，而是通过充当
中介的主题交换消息。

![TopicConcepts](./media/howto-service-bus-topics/sb-topics-01.png)

与每条消息由单个使用方处理的 Service Bus 队列相比，主题和订阅
通过发布/订阅模式提供一对多通信方式。可向一个
主题注册多个订阅。当消息发送到主题时，每个订阅
会分别对该消息进行处理。

主题订阅类似于接收发送至该主题的消息副本的虚拟队列。您可以选择基于每个订阅
注册主题的筛选规则，这样就可以筛选/限制哪些主题
订阅接收发送至某个主题的哪些消息。

利用 Service Bus 主题和订阅，您可以进行扩展以处理跨大量
用户和应用程序的许多消息。

<h2><a name="create-a-service-namespace"></a>创建服务命名空间</h2>

若要开始在 Windows Azure 中使用 Service Bus 主题和订阅，
必须先创建一个服务命名空间。服务命名空间提供了
用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

创建服务命名空间：

1. 登录到 [Windows Azure 管理门户][]。

2. 在该管理门户的左侧导航窗格中，单击 Service Bus。

3. 在管理门户的下方窗格中，单击“创建”。
    ![][0]

4. 在“添加新命名空间”对话框中，输入命名空间名称。
    系统会立即检查该名称是否可用。
    ![][2]

5. 在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区
（确保使用在其中部署计算资源的同一国家/地区）。

	重要说明：选取要部署应用程序的相同区域。这将为您提供最佳性能。

6.	单击复选标记。系统现已创建您的服务命名空间并
   已将其启用。您可能需要等待几分钟，因为系统将为
   您的帐户配置资源。

	![][6]


<h2><a name="obtain-default-credentials"></a>获取命名空间的默认管理凭据</h2>

若要在新命名空间上执行管理操作（如创建主题或订阅），则需要
获取该命名空间的管理凭据。

1. 在左侧导航窗格中，单击 Service Bus 节点以显示
   可用命名空间的列表：  
    ![][0]

2. 从显示的列表中选择刚刚创建的命名空间：
    ![][3]

3. 单击“连接信息”。
    ![][4]

4. 在“访问连接信息”对话框中，找到“默认颁发者”和“默认密钥”条目。记下这些值，因为您将在下面使用此信息来对命名空间执行操作。

 
  [Windows Azure 管理门户]: http://manage.windowsazure.com
  [0]: ./media/howto-service-bus-topics/sb-queues-13.png
  [2]: ./media/howto-service-bus-topics/sb-queues-04.png
  [3]: ./media/howto-service-bus-topics/sb-queues-09.png
  [4]: ./media/howto-service-bus-topics/sb-queues-06.png
  
  [6]: ./media/howto-service-bus-topics/getting-started-multi-tier-27.png

