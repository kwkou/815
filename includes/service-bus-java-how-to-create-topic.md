## 什么是 Service Bus 主题和订阅？

Service Bus 主题和订阅支持发布/订阅消息通信模型。在使用主题和订阅时，分布式应用程序的组件不会直接相互通信，而是通过充当中介的主题交换消息。

![TopicConcepts](./media/service-bus-java-how-to-create-topic/sb-topics-01.png)

与每条消息由单个使用方处理的 Service Bus 队列相比，主题和订阅通过发布/订阅模式提供“一对多”通信方式。可向一个主题注册多个订阅。当消息发送到主题时，每个订阅会分别对该消息进行处理。

主题订阅类似于接收发送至该主题的消息副本的虚拟队列。你可以选择基于每个订阅注册主题的筛选规则，这样就可以筛选/限制哪些主题订阅接收发送至某个主题的哪些消息。

利用 Service Bus 主题和订阅，你可以进行扩展以处理跨大量用户和应用程序的许多消息。

## 创建服务命名空间

若要开始在 Azure 中使用服务总线主题和订阅，必须先创建一个服务命名空间。命名空间提供了用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

创建命名空间：

1.  登录到 [Azure 管理门户][]。

2.  在门户的左侧导航窗格中，单击“服务总线”。

3.  在门户的下方窗格中，单击“创建”。![][0]

4.  在“添加新命名空间”对话框中，输入命名空间名称。系统会立即检查该名称是否可用。![][2]

5.  在确保命名空间名称可用后，选择应承载你的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。

	重要说明：选取要部署应用程序的**相同区域**。这将为你提供最佳性能。

6. 	将对话框中的其他字段保留其默认值（“消息传递”和“标准层”），然后单击复选标记。系统现已创建你的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

	![][6]


## 获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建主题或订阅），则必须从获取该命名空间的管理凭据。可以从门户中获取这些凭据。

### 从门户中获取管理凭据

1.  在左侧导航窗格中，单击“Service Bus”节点以显示可用命名空间的列表：![][0]

2.  从显示的列表中单击你刚刚创建的命名空间：![][3]

3.  单击“配置”以查看命名空间的共享访问策略。![](./media/service-bus-java-how-to-create-topic/sb-queues-14.png)

4.  记下主密钥，或将其复制到剪贴板。


  [Azure 管理门户]: http://manage.windowsazure.cn
  [0]: ./media/service-bus-java-how-to-create-topic/sb-queues-13.png
  [2]: ./media/service-bus-java-how-to-create-topic/sb-queues-04.png
  [3]: ./media/service-bus-java-how-to-create-topic/sb-queues-09.png
  [4]: ./media/service-bus-java-how-to-create-topic/sb-queues-06.png

  [6]: ./media/service-bus-java-how-to-create-topic/getting-started-multi-tier-27.png
  [34]: ./media/service-bus-java-how-to-create-topic/VSProperties.png

<!---HONumber=Mooncake_0104_2016-->