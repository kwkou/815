## 将消息发送到事件中心
在本部分中，我们将编写用于将事件发送到事件中心的 Java 控制台应用。我们将从 [Apache Qpid 项目](http://qpid.apache.org/)利用 JMS AMQP 提供者。这类似于通过 Java 将服务总线队列和主题与 AMQP 配合使用，如[此处](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)所示。有关详细信息，请参阅 [Qpid JMS 文档](http://qpid.apache.org/releases/qpid-0.30/programming/book/QpidJMS.html)和 [Java 消息服务](http://www.oracle.com/technetwork/java/jms/index.html)。

1. 在 Eclipse 中，安装 [Azure Toolkit for Eclipse](/documentation/articles/azure-toolkit-for-eclipse-installation/)。这包括 Qpid JMS AMQP 客户端库。

2. 在 Eclipse 中，创建一个名为 **Sender** 的新 Java 项目。

3. 在 Eclipse 包资源管理器中，右键单击该 **Sender** 项目并选择“属性”。在对话框的左窗格中，单击“Java 生成路径”，然后单击“库”选项卡，然后单击“添加库”按钮。选择“Apache Qpid JMS 客户端包(由 MS Open Tech 开发)”，单击“下一步”，然后单击“完成”。

	![][8]

4. 在 **Sender** 项目的根目录中，创建一个名为 **servicebus.properties** 的文件，其内容如下。请记得替换以下值：
	- 事件中心名称。
	- 命名空间名称（后者通常是 `{event hub name}-ns`）。
	- URL 编码的 **SendRule** 键（你在创建事件中心时已经记下了此键）。可以在[此处](http://www.w3schools.com/tags/ref_urlencode.asp)对它进行 URL 编码。

    		# servicebus.properties - sample JNDI configuration
    
    		# Register a ConnectionFactory in JNDI using the form:
    		# connectionfactory.[jndi_name] = [ConnectionURL]
    		connectionfactory.SBCF = amqps://SendRule:{Send Rule key}@{namespace name}.servicebus.chinacloudapi.cn/?sync-publish=false
    
    		# Register some queues in JNDI using the form
    		# queue.[jndi_name] = [physical_name]
    		# topic.[jndi_name] = [physical_name]
    		queue.EventHub = {event hub name}

5. 创建名为 **Sender** 的新类。添加以下  `import` 语句：

		import java.io.BufferedReader;
		import java.io.IOException;
		import java.io.InputStreamReader;
		import java.io.UnsupportedEncodingException;
		import java.util.Hashtable;
		
		import javax.jms.BytesMessage;
		import javax.jms.Connection;
		import javax.jms.ConnectionFactory;
		import javax.jms.Destination;
		import javax.jms.JMSException;
		import javax.jms.MessageProducer;
		import javax.jms.Session;
		import javax.naming.Context;
		import javax.naming.InitialContext;
		import javax.naming.NamingException; 

6. 然后，给它添加以下代码：

		public static void main(String[] args) throws NamingException,
				JMSException, IOException, InterruptedException {
			// Configure JNDI environment
			Hashtable<String, String> env = new Hashtable<String, String>();
			env.put(Context.INITIAL_CONTEXT_FACTORY,
					"org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
			env.put(Context.PROVIDER_URL, "servicebus.properties");
			Context context = new InitialContext(env);
	
			ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
	
			Destination queue = (Destination) context.lookup("EventHub");
	
			// Create Connection
			Connection connection = cf.createConnection();
	
			// Create sender-side Session and MessageProducer
			Session sendSession = connection.createSession(false,
					Session.AUTO_ACKNOWLEDGE);
			MessageProducer sender = sendSession.createProducer(queue);
	
			System.out.println("Press Ctrl-C to stop the sender process");
			System.out.println("Press Enter to start now");
			BufferedReader commandLine = new java.io.BufferedReader(
					new InputStreamReader(System.in));
			commandLine.readLine();
	
			while (true) {
				sendBytesMessage(sendSession, sender);
				Thread.sleep(200);
			}
		}
		
		private static void sendBytesMessage(Session sendSession, MessageProducer sender) throws JMSException, UnsupportedEncodingException {
	        BytesMessage message = sendSession.createBytesMessage();
	        message.writeBytes("Test AMQP message from JMS".getBytes("UTF-8"));
	        sender.send(message);
	        System.out.println("Sent message");
	    }



<!-- Images -->
[8]: ./media/service-bus-event-hubs-getstarted/create-sender-java1.png

<!---HONumber=Mooncake_1207_2015-->