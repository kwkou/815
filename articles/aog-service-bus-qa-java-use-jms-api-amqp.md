
<properties
                pageTitle="使用 JAVA AMQP 协议如何订阅启用分区功能的 Azure 服务总线主题的消息"
                description="借助 Java JMS API 使用 AMQP 协议订阅启用分区的 Azure 服务总线主题的消息"
                services="service-bus"
                documentationCenter=""
                authors=""
                manager=""
                editor=""
                tags="service bus topic,partition,subscription,AMQP"/>

<tags
                ms.service="service-bus-aog"
                ms.date="12/15/2016"
                wacn.date="12/15/2016"/>

# 使用 JAVA AMQP 协议如何订阅启用分区功能的 Azure 服务总线主题的消息  

## 问题描述：  

通常借助 Java JMS API 使用 AMQP 协议订阅启用分区的 Azure 服务总线主题会报错, 而订阅未启用分区功能的  Azure 服务总线主题则可正常运行。  

**报错信息为：**  

    JMS Exception: Cannot open a Topic client for entity type Subscriber

## 解决方法：  

使用 Azure 服务总线队列订阅的方式订阅主题消息，主题订阅者对应队列的 `entity path` 为  `[Topic Name]/Subscriptions/[Subscription Name]`。

**代码如下：**  



	Context context = new InitialContext();
	ConnectionFactory factory = (ConnectionFactory) context.lookup("myFactoryLookup");
	Connection connection = factory.createConnection(USER, PASSWORD);
	connection.setExceptionListener(new ExceptionListener());
	connection.start();

	TopicSession session = (TopicSession) connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	Topic topic = session.createTopic("test");

	// 对未启用 partition 的 topic 可以用 TopicSubscriber 订阅消息
	// TopicSubscriber subscriber = session.createDurableSubscriber(topic, "subscription1");
	// subscriber.setMessageListener(new MessageListener());

	// 对启用 partition的topic 只能用 MessageConsumer 来订阅消息
	MessageConsumer messageConsumer = session.createConsumer(session.createQueue("test/Subscriptions/sub1"));
	messageConsumer.setMessageListener(new MessageListener());

	MessageProducer messageProducer = session.createProducer(topic);
	Message message = session.createTextMessage("Hello world1213!");
	messageProducer.send(message);



