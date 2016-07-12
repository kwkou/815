<properties
	pageTitle="适用于 Java 的 Azure IoT 中心入门 | Azure"
	description="适用于 Java 的 Azure IoT 中心入门教程。配合 Azure IoT SDK 使用 Azure IoT 中心和 Java 来实施物联网解决方案。"
	services="iot-hub"
	documentationCenter="java"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="06/16/2016"
     wacn.date="07/04/2016"/>

# 适用于 Java 的 Azure IoT 中心入门

[AZURE.INCLUDE [iot-hub-selector-get-started](../includes/iot-hub-selector-get-started.md)]

在本教程结束时，你将有三个 Java 控制台应用程序：

* **create-device-identity**，用于创建设备标识和关联的安全密钥以连接模拟设备。
* **read-d2c-messages**，显示模拟设备发送的遥测数据。
* **simulated-device**，它使用前面创建的设备标识连接到 IoT 中心，并使用 AMQPS 协议每秒发送遥测消息一次。

> [AZURE.NOTE] [IoT 中心 SDK][lnk-hub-sdks] 一文提供了有关多种 SDK 的信息，你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序。

若要完成本教程，你需要以下各项：

+ Java SE 8。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Java。

+ Maven 3。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Maven。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../includes/iot-hub-get-started-create-hub.md)]

最后一步是在 IoT 中心边栏选项卡上单击“设置”，然后在“设置”边栏选项卡上单击“消息传送”。在“消息传送”边栏选项卡上，记下“与事件中心兼容的名称”和“与事件中心兼容的终结点”。创建 **read-d2c-messages** 应用程序时，将要用到这些值。

    ![][6]

现在，你已创建 IoT 中心并获取了 IoT 中心主机名、IoT 中心连接字符串、与事件中心兼容的名称及与事件中心兼容的终结点，接下来需要完成本教程的余下部分。

## 创建设备标识

在本部分中，你将创建一个 Java 控制台应用程序，用于在 IoT 中心的标识注册表中创建新的设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]的“设备标识注册表”部分。当你运行此控制台应用程序时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以使用这些信息标识设备本身。

1. 新建名为 iot-java-get-started 的空文件夹。在命令提示符下的 iot-java-get-started 文件夹中，使用以下命令创建名为 **create-device-identity** 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=create-device-identity -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 create-device-identity 文件夹。

3. 使用文本编辑器，打开 create-device-identity 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。这可让你在应用程序中使用 iothub-service-sdk 包：

    ```
    <dependency>
      <groupId>com.microsoft.azure.iothub-java-client</groupId>
      <artifactId>iothub-java-service-client</artifactId>
      <version>1.0.2</version>
    </dependency>
    ```
    
4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 create-device-identity\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

    ```
    import com.microsoft.azure.iot.service.exceptions.IotHubException;
    import com.microsoft.azure.iot.service.sdk.Device;
    import com.microsoft.azure.iot.service.sdk.RegistryManager;

    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. 将以下类级变量添加到 **App** 类，并将 **{yourhostname}** 和 **{yourhubkey}** 替换为前面记下的值：

    ```
    private static final String connectionString = "HostName={yourhostname};SharedAccessKeyName=iothubowner;SharedAccessKey={yourhubkey}";
    private static final String deviceId = "javadevice";
    
    ```
    
8. 修改 **main** 方法的签名以添加如下所示的异常：

    ```
    public static void main( String[] args ) throws IOException, URISyntaxException, Exception
    ```
    
9. 添加以下代码作为 **main** 方法的主体。此代码将在 IoT 中心标识注册表中创建名为 javadevice 的设备（如果还没有该设备）。然后，显示稍后需要用到的设备 ID 和密钥：

    ```
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
    System.out.println("Device id: " + device.getDeviceId());
    System.out.println("Device key: " + device.getPrimaryKey());
    ```

10. 保存并关闭 App.java 文件。

11. 若要使用 Maven 生成 **create-device-identity** 应用程序，请在命令提示符下的 create-device-identity 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

12. 若要使用 Maven 运行 **create-device-identity** 应用程序，请在命令提示符下的 create-device-identity 文件夹中执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

13. 记下**设备 ID** 和**设备密钥**。稍后在创建连接到作为设备的 IoT 中心的应用程序时需要这些数据。

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志（可用于禁用对单个设备的访问）。如果应用程序需要存储其他特定于设备的元数据，则应使用特定于应用程序的存储。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

## 接收设备到云的消息

在本部分中，你将创建一个 Java 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与[事件中心][lnk-event-hubs-overview]兼容的终结点，以让你读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。[处理设备到云的消息][lnk-process-d2c-tutorial]教程介绍了如何大规模处理设备到云的消息。[事件中心入门][lnk-eventhubs-tutorial]教程更详细介绍了如何处理来自事件中心的消息，此教程也适用于与 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 在命令提示符下，在创建设备标识部分中创建的 iot-java-get-started 文件夹中，使用以下命令创建名为 **read-d2c-messages** 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-d2c-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 read-d2c-messages 文件夹。

3. 使用文本编辑器，打开 read-d2c-messages 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。这可让你在应用程序中使用 eventhubs-client 包，以从事件中心兼容的终结点进行读取：

    ```
    <dependency> 
        <groupId>com.microsoft.azure</groupId> 
        <artifactId>azure-eventhubs</artifactId> 
        <version>0.7.1</version> 
    </dependency>
    ```

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 read-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

    ```
    import java.io.IOException;
    import com.microsoft.azure.eventhubs.*;
    import com.microsoft.azure.servicebus.*;
    
    import java.io.IOException;
    import java.nio.charset.Charset;
    import java.time.*;
    import java.util.Collection;
    import java.util.concurrent.ExecutionException;
    import java.util.function.*;
    import java.util.logging.*;
    ```

7. 将以下类级变量添加到 **App** 类。将 **{youriothubkey}**、**{youreventhubcompatibleendpoint}** 和 **{youreventhubcompatiblename}** 替换为前面记下的值：

    ```
    private static String connStr = "Endpoint={youreventhubcompatibleendpoint};EntityPath={youreventhubcompatiblename};SharedAccessKeyName=iothubowner;SharedAccessKey={youriothubkey}";
    ```

8. 将以下 **receiveMessages** 方法添加到 **App** 类。此方法创建 **EventHubClient** 实例以连接到与事件中心兼容的终结点，然后以异步方式创建 **PartitionReceiver** 实例，以便从事件中心分区读取。它将持续循环并输出消息详细信息，直到应用程序终止。

    ```
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
                    System.out.println(String.format("| Device ID: %s", receivedEvent.getProperties().get("iothub-connection-device-id")));
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
    ```

    > [AZURE.NOTE] 在创建开始运行后只读取发送到 IoT 中心的消息的接收方时，此方法将使用筛选器。这很适合测试环境，因为这样可以看到当前的消息集，但在生产环境中，代码应该要确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-process-d2c-tutorial]教程。

11. 修改 **main** 方法的签名以添加如下所示的异常：

    ```
    public static void main( String[] args ) throws IOException
    ```

10. 在 **App** 类的 **main** 方法中添加以下代码。此代码将创建两个（**EventHubClient** 和 **PartitionReceiver**）实例并让你在处理完消息后关闭应用程序：

    ```
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
    ```

    > [AZURE.NOTE] 此代码假设已在 F1（免费）层创建 IoT 中心。免费 IoT 中心有“0”和“1”这两个分区。

13. 保存并关闭 App.java 文件。

14. 若要使用 Maven 生成 **read-d2c-messages** 应用程序，请在命令提示符下的 read-d2c-messages 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

## 创建模拟设备应用程序

在本部分中，你将创建一个 Java 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 在命令提示符下，在创建设备标识部分中创建的 iot-java-get-started 文件夹中，使用以下命令创建名为 **simulated-device** 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 simulated-device 文件夹。

3. 使用文本编辑器，打开 simulated-device 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。这样，你便可以使用应用程序中的 iothub-java-client 包来与 IoT 中心通信，并将 Java 对象序列化为 JSON：

    ```
    <dependency>
      <groupId>com.microsoft.azure.iothub-java-client</groupId>
      <artifactId>iothub-java-device-client</artifactId>
      <version>1.0.2</version>
    </dependency>
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.3.1</version>
    </dependency>
    ```

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

    ```
    import com.microsoft.azure.iothub.DeviceClient;
    import com.microsoft.azure.iothub.IotHubClientProtocol;
    import com.microsoft.azure.iothub.Message;
    import com.microsoft.azure.iothub.IotHubStatusCode;
    import com.microsoft.azure.iothub.IotHubEventCallback;
    import com.microsoft.azure.iothub.IotHubMessageResult;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.security.InvalidKeyException;
    import java.util.Random;
    import javax.naming.SizeLimitExceededException;
    import com.google.gson.Gson;
    ```

7. 将以下类级变量添加到 **App** 类，并将 **{youriothubname}** 替换为你的 IoT 中心名称，将 **{yourdeviceid}** 和 **{yourdevicekey}** 替换为创建设备标识部分中生成的设备值：

    ```
    private static String connString = "HostName={youriothubname}.azure-devices.cn;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.AMQPS;
    private static boolean stopThread = false;
    ```

    本示例应用程序在实例化 **DeviceClient** 对象时使用 **protocol** 变量。你可以使用 HTTPS 或 AMQPS 协议来与 IoT 中心通信。

8. 在 **App** 类中添加以下嵌套 **TelemetryDataPoint** 类，以指定设备要发送到 IoT 中心的遥测数据：

    ```
    private static class TelemetryDataPoint {
      public String deviceId;
      public double windSpeed;
        
      public String serialize() {
        Gson gson = new Gson();
        return gson.toJson(this);
      }
    }
    ```

9. 在 **App** 类中添加以下嵌套 **EventCallback** 类，以显示 IoT 中心在处理来自模拟设备的消息时返回的确认状态。处理消息时，此方法还会通知应用程序中的主线程：

    ```
    private static class EventCallback implements IotHubEventCallback
    {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to message with status " + status.name());
      
        if (context != null) {
          synchronized (context) {
            context.notify();
          }
        }
      }
    }
    ```

10. 在 **App** 类中添加以下嵌套 **MessageSender** 类。此类中的 **run** 方法将生成要发送到 IoT 中心的示例遥测数据，并在发送下一条消息之前等待确认：

    ```
    private static class MessageSender implements Runnable {
      public volatile boolean stopThread = false;

      public void run()  {
        try {
          double avgWindSpeed = 10; // m/s
          Random rand = new Random();
          DeviceClient client;
          client = new DeviceClient(connString, protocol);
          client.open();
        
          while (!stopThread) {
            double currentWindSpeed = avgWindSpeed + rand.nextDouble() * 4 - 2;
            TelemetryDataPoint telemetryDataPoint = new TelemetryDataPoint();
            telemetryDataPoint.deviceId = "myFirstDevice";
            telemetryDataPoint.windSpeed = currentWindSpeed;
      
            String msgStr = telemetryDataPoint.serialize();
            Message msg = new Message(msgStr);
            System.out.println(msgStr);
        
            Object lockobj = new Object();
            EventCallback callback = new EventCallback();
            client.sendEventAsync(msg, callback, lockobj);
    
            synchronized (lockobj) {
              lockobj.wait();
            }
            Thread.sleep(1000);
          }
          client.close();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }
    }
    ```

    IoT 中心确认前面的消息一秒后，此方法将发送新的设备到云消息。该消息包含具有 deviceId 的 JSON 序列化对象和一个随机生成的编号，用于模拟风速传感器。

11. 将 **main** 方法替换为以下代码，该代码创建用于向 IoT 中心发送设备到云消息的线程：

    ```
    public static void main( String[] args ) throws IOException, URISyntaxException {
    
      MessageSender ms0 = new MessageSender();
      Thread t0 = new Thread(ms0);
      t0.start(); 
    
      System.out.println("Press ENTER to exit.");
      System.in.read();
      ms0.stopThread = true;
    }
    ```

12. 保存并关闭 App.java 文件。

13. 若要使用 Maven 生成 **simulated-device** 应用程序，请在命令提示符下的 simulated-device 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling（暂时性故障处理）][lnk-transient-faults]中所述实施重试策略（例如指数性的回退）。
## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在命令提示符下的 read-d2c 文件夹中，运行以下命令以开始监视 IoT 中心的第一个分区：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"  -Dexec.args="0"
    ```

    ![][7]

1. 在命令提示符下的 read-d2c 文件夹中，运行以下命令以开始监视 IoT 中心的第二个分区：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"  -Dexec.args="1"
    ```

    ![][7]

2. 在 simulated-device 文件夹中的命令提示符下，运行以下命令开始将遥测数据发送到 IoT 中心：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![][8]

3. [Azure 门户][lnk-portal]中的“使用情况”磁贴显示了发送到中心的消息数：

    ![][43]

## 后续步骤

在本教程中，你已在门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已使用此设备标识来让模拟设备应用向中心发送设备到云的消息，并创建了用于显示中心所接收消息的应用。可以使用以下教程继续探索 IoT 中心功能和其他 IoT 方案：

- [使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]介绍了如何将消息发送到设备，并处理 IoT 中心生成的传送反馈。
- [处理设备到云的消息][lnk-process-d2c-tutorial]介绍了如何可靠地处理来自设备的遥测数据和交互消息。
- [从设备上载文件][lnk-upload-tutorial]介绍了一种模式，该模式利用云到设备的消息来帮助从设备上载文件。

<!-- Images. -->
[1]: ./media/iot-hub-java-java-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-java-java-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-java-java-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-java-java-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-java-java-getstarted/create-iot-hub5.png
[6]: ./media/iot-hub-java-java-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-java-java-getstarted/runapp1.png
[8]: ./media/iot-hub-java-java-getstarted/runapp2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/java-devbox-setup.md
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[lnk-upload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload/

[lnk-hub-sdks]: /documentation/articles/iot-hub-sdks-summary/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-resource-groups]: /documentation/articles/resource-group-portal/
[lnk-portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0307_2016-->