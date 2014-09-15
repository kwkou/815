<a id="what-are-service-bus-queues"></a>
##什么是 Service Bus 队列？

Service Bus 队列支持中转消息传递通信模型。在使用队列时，分布式应用程序的组件
不会直接相互通信，而是通过
充当中介的队列交换消息。消息创建方（发送方）
将消息传送到队列，然后继续对其进行处理。
消息使用方（接收方）以异步方式从队列中提取
消息并处理它。创建方不必等待
使用方的答复即可继续处理并发送更多
消息。队列为一个或多个竞争使用方提供“先入先出 (FIFO)”
消息传递方式。也就是说，接收方通常
会按照消息添加到队列中的顺序来接收并处理
消息，并且每条消息仅由一个消息使用方接收
并处理。

![QueueConcepts](./media/howto-service-bus-queues/sb-queues-08.png)

Service Bus 队列是一种可用于各种应用场景
的通用技术：

-   多层 Windows Azure 应用程序中 Web 角色和辅助角色之间的通信
-   混合解决方案中本地应用程序和 Windows Azure 托管应用程序之间的通信
-   在不同组织或组织的各部门中本地运行的分布式应用程序组件之间
    的通信

利用队列，您可以更好地向外扩展应用程序，并
增强您的体系结构的恢复能力。

<a id="create-a-service-namespace"></a>
<h2>创建服务命名空间</h2>

若要开始在 Windows Azure 中使用 Service Bus 队列，必须先
创建一个服务命名空间。服务命名空间提供了
用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

创建服务命名空间：

1. 登录到 [Windows Azure 管理门户][]。

2. 在该管理门户的左侧导航窗格中，单击 Service Bus。

3. 在管理门户的下方窗格中，单击“创建”。
	![](./media/howto-service-bus-queues/sb-queues-03.png)

4. 在“添加新命名空间”对话框中，输入命名空间名称。
    系统会立即检查该名称是否可用。
	![](./media/howto-service-bus-queues/sb-queues-04.png)

5. 在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区
（确保使用在其中部署计算资源的同一国家/地区）。

	重要说明：选取要部署应用程序的相同区域。这将为您提供最佳性能。

6.	单击复选标记。系统现已创建您的服务命名空间并
   已将其启用。您可能需要等待几分钟，因为系统将为
   您的帐户配置资源。

	![](./media/howto-service-bus-queues/getting-started-multi-tier-27.png)

您创建的命名空间随后将显示在管理门户中，然后要
花费一段时间来激活。请等到状态变为“活动”后再继续。

<a id="obtain-default-credentials"></a>
<h2>获取命名空间的默认管理凭据</h2>

若要在新命名空间上执行管理操作（如创建队列），
则必须获取该命名空间的管理
凭据。

1. 在左侧导航窗格中，单击 Service Bus 节点以显示
    可用命名空间的列表：
	![](./media/howto-service-bus-queues/sb-queues-13.png)

2. 从显示的列表中选择刚刚创建的命名空间：
	![](./media/howto-service-bus-queues/sb-queues-09.png)

3. 单击“连接信息”。
	![](./media/howto-service-bus-queues/sb-queues-06.png)

4. 在“访问连接信息”对话框中，找到“默认颁发者”和“默认密钥”条目。记下这些值，因为您将在下面使用此信息来对命名空间执行操作。

  [Windows Azure 管理门户]: http://manage.windowsazure.com
  [Windows Azure 管理门户]: http://manage.windowsazure.com



