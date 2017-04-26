<properties
    pageTitle="Azure IoT 中心入门 (Java) | Azure"
    description="如何使用 Azure IoT SDK for Java 将设备到云的消息从设备发送到 Azure IoT 中心。 将创建一个用于发送消息的模拟设备、一个用于在注册表中标识注册设备的服务应用，以及一个用于从 IoT 中心读取设备到云消息的服务应用。"
    services="iot-hub"
    documentationcenter="java"
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="70dae4a8-0e98-4c53-b5a5-9d6963abb245"
    ms.service="iot-hub"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/07/2017"
    wacn.date="04/24/2017"
    ms.author="dobett"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="0fb1239169f40c0f9c0b406dc93128da6bb915ba"
    ms.lasthandoff="04/14/2017" />

# <a name="connect-your-simulated-device-to-your-iot-hub-using-java"></a>使用 Java 将模拟设备连接到 IoT 中心
[AZURE.INCLUDE [iot-hub-selector-get-started](../../includes/iot-hub-selector-get-started.md)]

在本教程结束时，将获得三个 Java 控制台应用：

* **create-device-identity**，创建用于连接模拟设备应用的设备标识和关联的安全密钥。
* **read-d2c-messages**，显示模拟设备应用发送的遥测数据。
* **simulated-device**，使用前面创建的设备标识连接到 IoT 中心，并使用 MQTT 协议每秒发送一次遥测消息。

> [AZURE.NOTE]
> [Azure IoT SDK][lnk-hub-sdks] 文章介绍了一些 Azure IoT SDK，它们可用于构建在设备和解决方案后端运行的应用。
> 
> 

要完成本教程，需要具备以下先决条件：

+ Java SE 8。 <br/> [准备开发环境][lnk-dev-setup] 介绍了如何在 Windows 或 Linux 上安装本教程所用的 Java。

* Maven 3。  <br/> [准备开发环境][lnk-dev-setup]介绍如何在 Windows 或 Linux 上安装本教程所用的 [Maven][lnk-maven]。

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

最后，请记下“主密钥”值。 然后单击“终结点”和“事件”内置终结点。 在“属性”边栏选项卡中，记下“与事件中心兼容的名称”和“与事件中心兼容的终结点”的地址。 创建 **read-d2c-messages** 应用时，将要用到这三个值。

![Azure 门户 IoT 中心消息传递边栏选项卡][6]


现在已创建 IoT 中心。已获取 IoT 中心主机名、IoT 中心连接字符串、IoT 中心主密钥、与事件中心兼容的名称以及与事件中心兼容的终结点，接下来需要完成本教程。

## <a name="create-a-device-identity"></a>创建设备标识
在本部分中，会创建一个 Java 控制台应用，用于在 IoT 中心的标识注册表中创建设备标识。 设备无法连接到 IoT 中心，除非它在标识注册表中具有条目。 有关详细信息，请参阅 **IoT 中心开发人员指南** 的 [标识注册表][lnk-devguide-identity]部分。 当你运行此控制台应用时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以用于标识设备本身。

1. 创建名为 iot-java-get-started 的空文件夹。在 iot-java-get-started 文件夹的命令提示符处，使用以下命令创建名为 **create-device-identity** 的 Maven 项目。请注意，这是一条很长的命令：

    
	    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=create-device-identity -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    

2. 在命令提示符下，浏览到 create-device-identity 文件夹。
3. 使用文本编辑器，打开 create-device-identity 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。 通过此依赖项可在应用中使用 iot-service-client 包：

        <groupId>com.microsoft.azure.sdk.iot</groupId>
          <artifactId>iot-service-client</artifactId>
          <version>1.0.15</version>
        </dependency>

    
    > [AZURE.NOTE]
    > 可以使用 [Maven 搜索][lnk-maven-service-search]检查是否有最新版本的 **iot-service-client**。

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 create-device-identity\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

        import com.microsoft.azure.sdk.iot.service.exceptions.IotHubException;
        import com.microsoft.azure.sdk.iot.service.sdk.Device;
        import com.microsoft.azure.sdk.iot.service.sdk.RegistryManager;

	    import java.io.IOException;
	    import java.net.URISyntaxException;
    

7. 将以下类级变量添加到“应用”类，并将 **{yourhubconnectionstring}** 替换为前面记录的值：

    
	    private static final String connectionString = "{yourhubconnectionstring}";
	    private static final String deviceId = "myFirstJavaDevice";
    
    
    
8. 修改 **main** 方法的签名，包含如下所示的异常：

    
	    public static void main( String[] args ) throws IOException, URISyntaxException, Exception
    
    
9. 添加以下代码作为 **main** 方法的主体。此代码将在 IoT 中心标识注册表中创建名为 *javadevice* 的设备（如果还没有该设备）。随即显示稍后需要用到的设备 ID 和密钥：

    
	    RegistryManager registryManager = RegistryManager.createFromConnectionString(connectionString);

	    Device device = Device.createFromId(deviceId, null, null);
	    try {
	      device = registryManager.addDevice(device);
	    } catch (IotHubException iote) {
	      try {
	        device = registryManager.getDevice(deviceId);
	      } catch (IotHubException iotf) {
	        iotf.printStackTrace();
	      }
	    }
    	    System.out.println("Device ID: " + device.getDeviceId());
	    System.out.println("Device key: " + device.getPrimaryKey());
    

10. 保存并关闭 App.java 文件。

11. 若要使用 Maven 生成 **create-device-identity** 应用，请在命令提示符下的 create-device-identity 文件夹中执行以下命令：

    
	    mvn clean package -DskipTests
    

12. 若要使用 Maven 运行 **create-device-identity** 应用，请在 create-device-identity 文件夹中的命令提示符下执行以下命令：

    
	    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    

13. 记下**设备 ID** 和**设备密钥**。 稍后在创建连接到作为设备的 IoT 中心的应用时需要这些值。

> [AZURE.NOTE]
> IoT 中心标识注册表仅存储用于实现 IoT 中心安全访问的设备标识。 它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志（可用于禁用对单个设备的访问）。 如果应用需要存储设备特定的其他元数据，需使用应用特定的存储。 有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。
> 
> 

## <a name="receive-device-to-cloud-messages"></a>接收设备到云的消息
在本部分中，你将创建一个 Java 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。 IoT 中心公开与 [事件中心][lnk-event-hubs-overview]兼容的终结点，以便你可读取设备到云的消息。 为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。 [Process device-to-cloud messages][lnk-process-d2c-tutorial] （处理设备到云的消息）教程介绍了如何大规模处理设备到云的消息。 [事件中心入门][lnk-eventhubs-tutorial] 教程更详细地介绍了如何处理来自事件中心的消息，此教程也适用于与 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE]
> 与事件中心兼容的终结点始终使用 AMQP 协议读取设备到云的消息。
> 
> 

1. 在 *创建设备标识* 部分中创建的 iot-java-get-started 文件夹中，在命令提示符处使用以下命令创建名为 **read-d2c-messages** 的 Maven 项目。 请注意，这是一条很长的命令：

    
	    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-d2c-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    

2. 在命令提示符下，浏览到 read-d2c-messages 文件夹。
3. 使用文本编辑器，打开 read-d2c-messages 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。 通过此依赖项可在应用中使用 eventhubs-client 包，从与事件中心兼容的终结点读取数据：

        <dependency> 
            <groupId>com.microsoft.azure</groupId> 
            <artifactId>azure-eventhubs</artifactId> 
            <version>0.11.0</version> 
        </dependency>

    > [AZURE.NOTE]
    > 可以使用 [Maven 搜索][lnk-maven-eventhubs-search]检查是否有最新版本的 **azure-eventhubs**。

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 read-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

        import java.io.IOException;
        import com.microsoft.azure.eventhubs.*;
        import com.microsoft.azure.servicebus.*;

        import java.nio.charset.Charset;
        import java.time.*;
        import java.util.function.*;

7. 将下列类级变量添加到 **App** 类。 将 **{youriothubkey}**、**{youreventhubcompatibleendpoint}** 和 **{youreventhubcompatiblename}** 替换为前面记下的值：

    
	    private static String connStr = "Endpoint={youreventhubcompatibleendpoint};EntityPath={youreventhubcompatiblename};SharedAccessKeyName=iothubowner;SharedAccessKey={youriothubkey}";
    

8. 将以下 **receiveMessages** 方法添加到 **App** 类。 此方法创建 **EventHubClient** 实例以连接到与事件中心兼容的终结点，然后以异步方式创建 **PartitionReceiver** 实例，以便从事件中心分区读取。 它将持续循环并输出消息详细信息，直到应用终止。

        private static EventHubClient receiveMessages(final String partitionId)
        {
          EventHubClient client = null;
          try {
            client = EventHubClient.createFromConnectionStringSync(connStr);
          }
          catch(Exception e) {
            System.out.println("Failed to create client: " + e.getMessage());
            System.exit(1);
          }
          try {
            client.createReceiver( 
              EventHubClient.DEFAULT_CONSUMER_GROUP_NAME,  
              partitionId,  
              Instant.now()).thenAccept(new Consumer<PartitionReceiver>()
            {
              public void accept(PartitionReceiver receiver)
              {
                System.out.println("** Created receiver on partition " + partitionId);
                try {
                  while (true) {
                    Iterable<EventData> receivedEvents = receiver.receive(100).get();
                    int batchSize = 0;
                    if (receivedEvents != null)
                    {
                      for(EventData receivedEvent: receivedEvents)
                      {
                        System.out.println(String.format("Offset: %s, SeqNo: %s, EnqueueTime: %s", 
                          receivedEvent.getSystemProperties().getOffset(), 
                          receivedEvent.getSystemProperties().getSequenceNumber(), 
                          receivedEvent.getSystemProperties().getEnqueuedTime()));
                    	System.out.println(String.format("| Device ID: %s", receivedEvent.getSystemProperties().get("iothub-connection-device-id")));
                        System.out.println(String.format("| Message Payload: %s", new String(receivedEvent.getBody(),
                          Charset.defaultCharset())));
                        batchSize++;
                      }
                    }
                    System.out.println(String.format("Partition: %s, ReceivedBatch Size: %s", partitionId,batchSize));
                  }
                }
                catch (Exception e)
                {
                  System.out.println("Failed to receive messages: " + e.getMessage());
                }
              }
            });
          }
          catch (Exception e)
          {
            System.out.println("Failed to create receiver: " + e.getMessage());
          }
          return client;
        }

    > [AZURE.NOTE]
    > 在创建开始运行后只读取发送到 IoT 中心的消息的接收方时，此方法将使用筛选器。 此方法很适合测试环境，因为这样可以看到当前的消息集。 在生产环境中，代码应确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-process-d2c-tutorial]教程。
    > 
    > 
    
9. 修改 **main** 方法的签名，包含如下所示的异常：

    
	    public static void main( String[] args ) throws IOException
    

10. 在 **App** 类的 **main** 方法中添加以下代码。 此代码将创建两个（**EventHubClient** 和 **PartitionReceiver**）实例并使用户可在处理完消息后关闭应用：

    
	    EventHubClient client0 = receiveMessages("0");
	    EventHubClient client1 = receiveMessages("1");
	    System.out.println("Press ENTER to exit.");
	    System.in.read();
	    try
	    {
	      client0.closeSync();
	      client1.closeSync();
	      System.exit(0);
	    }
	    catch (ServiceBusException sbe)
	    {
	      System.exit(1);
	    }
    
    > [AZURE.NOTE]
    > 此代码假设已在 F1（免费）层创建 IoT 中心。免费 IoT 中心有“0”和“1”这两个分区。
    > 
    > 
11. 保存并关闭 App.java 文件。

12. 若要使用 Maven 构建 **read-d2c-messages** 应用，请在 read-d2c-messages 文件夹的命令提示符处执行以下命令：

    
	    mvn clean package -DskipTests
    

## <a name="create-a-simulated-device-app"></a>创建模拟设备应用程序
在本部分中，你将创建一个 Java 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 在*创建设备标识*部分创建的 iot-java-get-started 文件夹中，在命令提示符处创建名为 **simulated-device** 的 Maven 项目。请注意，这是一条很长的命令：

    
	    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    

2. 在命令提示符下，浏览到 simulated-device 文件夹。
3. 使用文本编辑器，打开 simulated-device 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。 通过依赖项可以使用应用中的 iothub-java-client 包来与 IoT 中心通信，并将 Java 对象序列化为 JSON：

        <dependency>
          <groupId>com.microsoft.azure.sdk.iot</groupId>
          <artifactId>iot-device-client</artifactId>
          <version>1.0.21</version>
        </dependency>
        <dependency>
          <groupId>com.google.code.gson</groupId>
          <artifactId>gson</artifactId>
          <version>2.3.1</version>
        </dependency>

    > [AZURE.NOTE]
    > 可以使用 [Maven 搜索][lnk-maven-device-search]检查是否有最新版本的 **iot-device-client**。

4. 保存并关闭 pom.xml 文件。
5. 使用文本编辑器打开 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。
6. 在该文件中添加以下 **import** 语句：

        import com.microsoft.azure.sdk.iot.device.DeviceClient;
        import com.microsoft.azure.sdk.iot.device.IotHubClientProtocol;
        import com.microsoft.azure.sdk.iot.device.Message;
        import com.microsoft.azure.sdk.iot.device.IotHubStatusCode;
        import com.microsoft.azure.sdk.iot.device.IotHubEventCallback;
        import com.microsoft.azure.sdk.iot.device.MessageCallback;
        import com.microsoft.azure.sdk.iot.device.IotHubMessageResult;
        import com.google.gson.Gson;
        import java.io.IOException;
        import java.net.URISyntaxException;
        import java.util.Random;
        import java.util.concurrent.Executors;
        import java.util.concurrent.ExecutorService;

7. 将以下类级变量添加到 **App** 类。 将 **{youriothubname}** 替换为你的 IoT 中心名称，将 **{yourdevicekey}** 替换为在*创建设备标识*部分中生成的设备值：

    
	    private static String connString = "HostName={youriothubname}.azure-devices.cn;DeviceId=myFirstJavaDevice;SharedAccessKey={yourdevicekey}";
	    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
	    private static String deviceId = "myFirstJavaDevice";
	    private static DeviceClient client;
    

    本示例应用在实例化 **DeviceClient** 对象时使用 **protocol** 变量。 可以使用 MQTT、AMQP 或 HTTP 协议与 IoT 中心通信。
8. 在 **App** 类中添加以下嵌套的 **TelemetryDataPoint** 类，以指定设备要发送到 IoT 中心的遥测数据：

        private static class TelemetryDataPoint {
          public String deviceId;
          public double windSpeed;

          public String serialize() {
            Gson gson = new Gson();
            return gson.toJson(this);
          }
        }

9. 在 **App** 类中添加以下嵌套的 **EventCallback** 类，以显示 IoT 中心在处理来自模拟设备应用的消息时返回的确认状态。 处理消息时，此方法还会通知应用中的主线程：

        private static class EventCallback implements IotHubEventCallback
        {
          public void execute(IotHubStatusCode status, Object context) {
            System.out.println("IoT Hub responded to message with status: " + status.name());

            if (context != null) {
              synchronized (context) {
                context.notify();
              }
            }
          }
        }

10. 在 **App** 类中添加以下嵌套的 **MessageSender** 类。 此类中的 **run** 方法将生成要发送到 IoT 中心的示例遥测数据，并在发送下一条消息之前等待确认：

        private static class MessageSender implements Runnable {

          public void run()  {
            try {
              double avgWindSpeed = 10; // m/s
              Random rand = new Random();

              while (true) {
                double currentWindSpeed = avgWindSpeed + rand.nextDouble() * 4 - 2;
                TelemetryDataPoint telemetryDataPoint = new TelemetryDataPoint();
                telemetryDataPoint.deviceId = deviceId;
                telemetryDataPoint.windSpeed = currentWindSpeed;

                String msgStr = telemetryDataPoint.serialize();
                Message msg = new Message(msgStr);
                System.out.println("Sending: " + msgStr);

                Object lockobj = new Object();
                EventCallback callback = new EventCallback();
                client.sendEventAsync(msg, callback, lockobj);

                synchronized (lockobj) {
                  lockobj.wait();
                }
                Thread.sleep(1000);
              }
            } catch (InterruptedException e) {
              System.out.println("Finished.");
            }
          }
        }

    IoT 中心确认前面的消息一秒后，此方法将发送新的设备到云消息。 该消息包含具有 deviceId 的 JSON 序列化对象和一个随机生成的编号，用于模拟风速传感器。
11. 将 **main** 方法替换为以下代码，该代码创建用于向 IoT 中心发送设备到云消息的线程：

        public static void main( String[] args ) throws IOException, URISyntaxException {
          client = new DeviceClient(connString, protocol);
          client.open();

          MessageSender sender = new MessageSender();

          ExecutorService executor = Executors.newFixedThreadPool(1);
          executor.execute(sender);

          System.out.println("Press ENTER to exit.");
          System.in.read();
          executor.shutdownNow();
          client.close();
        }

12. 保存并关闭 App.java 文件。
13. 若要使用 Maven 构建 **simulated-device** 应用，请在 simulated-device 文件夹的命令提示符处执行以下命令：

        mvn clean package -DskipTests

> [AZURE.NOTE]
> 为简单起见，本教程不实现任何重试策略。 在生产代码中，应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。
> 
> 

## <a name="run-the-apps"></a>运行应用
现在，已准备就绪，可以运行应用。

1. 在 read-d2c 文件夹的命令提示符处，运行以下命令监视 IoT 中心的第一个分区：

    
	    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    

    ![用于监视设备到云的消息的 Java IoT 中心服务应用][7]  

2. 在 simulated-device 文件夹的命令提示符处，运行以下命令将遥测数据发送到 IoT 中心：

    
	    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    

    ![用于发送设备到云消息的 Java IoT 中心设备应用][8]

3. [Azure 门户预览][lnk-portal]中的“使用情况”磁贴显示发送到 IoT 中心的消息数：

    ![显示发送到 IoT 中心的消息数的 Azure 门户“使用情况”磁贴][43]

## <a name="next-steps"></a>后续步骤
本教程中，在 Azure 门户预览中配置了新的 IoT 中心，然后在 IoT 中心的标识注册表中创建了设备标识。已使用此设备标识，使模拟设备应用可将设备到云的消息发送至 IoT 中心。还创建了一个应用，用于显示 IoT 中心接收的消息。

若要继续了解 IoT 中心入门知识并浏览其他 IoT 方案，请参阅：

* [连接你的设备][lnk-connect-device]
* [设备管理入门][lnk-device-management]
* [IoT 网关 SDK 入门][lnk-gateway-SDK]

若要了解如何扩展 IoT 解决方案和如何大规模处理设备到云的消息，请参阅 [Process device-to-cloud messages][lnk-process-d2c-tutorial]（处理设备到云的消息）教程。

<!-- Images. -->
[6]: ./media/iot-hub-java-java-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-java-java-getstarted/runapp1.png
[8]: ./media/iot-hub-java-java-getstarted/runapp2.png
[43]: ./media/iot-hub-java-java-getstarted/usage.png

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-dotnet-standard-getstarted-send/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-what-is-event-hubs/

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-java
[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://portal.azure.cn/

[lnk-device-management]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-connect-device]: /develop/iot/
[lnk-maven]: https://maven.apache.org/what-is-maven.html
[lnk-maven-service-search]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22
[lnk-maven-device-search]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22
[lnk-maven-eventhubs-search]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22



<!--Update_Description:update wording and code-->