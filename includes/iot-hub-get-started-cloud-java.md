## 创建设备标识

在本部分中，你将创建一个 Java 控制台应用程序，用于在 IoT 中心的标识注册表中创建新的设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]的“设备识别注册表”部分。当你运行此控制台应用程序时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以用于标识设备本身。

1. 新建名为 iot-java-get-started 的空文件夹。在 iot-java-get-started 文件夹中，于命令提示符处使用以下命令创建名为 create-device-identity 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=create-device-identity -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 create-device-identity 文件夹。

3. 使用文本编辑器，打开 create-device-identity 文件夹中的 pom.xml 文件，并在 dependencies 节点中添加以下依赖性。这可让你在应用程序中使用 iothub-service-sdk 包：

    ```
    <dependency>
      <groupId>com.microsoft.azure.iothub-java-client</groupId>
      <artifactId>iothub-java-service-client</artifactId>
      <version>1.0.2</version>
    </dependency>
    ```
    
4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 create-device-identity\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 import 语句：

    ```
    import com.microsoft.azure.iot.service.exceptions.IotHubException;
    import com.microsoft.azure.iot.service.sdk.Device;
    import com.microsoft.azure.iot.service.sdk.RegistryManager;

    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. 将以下类级变量添加到 App 类，并将 {yourhubname} 和 {yourhubkey} 替换为前面记下的值：

    ```
    private static final String connectionString = "HostName={yourhubname}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey={yourhubkey}";
    private static final String deviceId = "javadevice";
    
    ```
    
8. 修改 main 方法的签名以添加如下所示的异常：

    ```
    public static void main( String[] args ) throws IOException, URISyntaxException, Exception
    ```
    
9. 添加以下代码作为 main 方法的主体。此代码将在 IoT 中心标识注册表中创建名为 javadevice 的设备（如果还没有设备）。然后，显示稍后需要用到的设备 ID 和密钥：

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

11. 若要使用 Maven 生成 create-device-identity应用程序，请在命令提示符下于 create-device-identity 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

12. 若要使用 Maven 运行 create-device-identity应用程序，请在命令提示符下于 create-device-identity 文件夹中执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

13. 记下设备 ID 和设备密钥。稍后在创建连接到作为设备的 IoT 中心的应用程序时需要这些数据。

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志让你禁用对单个设备的访问。如果应用程序需要存储其他设备特定的元数据，则应使用应用程序特定的存储。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

## 接收设备到云的消息

在本部分中，你将创建一个 Java 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与[事件中心][lnk-event-hubs-overview]兼容的终结点，以让你读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。[处理设备到云的消息][lnk-processd2c-tutorial]教程说明了如何大规模处理设备到云的消息。[事件中心入门][lnk-eventhubs-tutorial]教程更详细说明了如何处理来自事件中心的消息，此教程也适用于 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 在“创建设备标识”部分中创建的 iot-java-get-started 文件夹中，于命令提示符下使用以下命令创建名为 read-d2c-messages 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-d2c-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 read-d2c-messages 文件夹。

3. 使用文本编辑器，打开 read-d2c-messages 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖性。这可让你在应用程序中使用 eventhubs-client 包，以从事件中心兼容的终结点进行读取：

    ```
    <dependency>
      <groupId>com.microsoft.eventhubs.client</groupId>
      <artifactId>eventhubs-client</artifactId>
      <version>1.0</version>
    </dependency>
    ```

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 read-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

    ```
    import java.io.IOException;
    import com.microsoft.eventhubs.client.Constants;
    import com.microsoft.eventhubs.client.EventHubClient;
    import com.microsoft.eventhubs.client.EventHubEnqueueTimeFilter;
    import com.microsoft.eventhubs.client.EventHubException;
    import com.microsoft.eventhubs.client.EventHubMessage;
    import com.microsoft.eventhubs.client.EventHubReceiver;
    import com.microsoft.eventhubs.client.ConnectionStringBuilder;
    ```

7. 将以下类级变量添加到 **App** 类：

    ```
    private static EventHubClient client;
    private static long now = System.currentTimeMillis();
    ```

8. 在 **App** 类中添加以下嵌套类。应用程序将创建两个线程来运行 **MessageReceiver**，以从事件中心读取两个分区中的消息：

    ```
    private static class MessageReceiver implements Runnable
    {
        public volatile boolean stopThread = false;
        private String partitionId;
    }
    ```

9. 将以下构造函数添加到 **MessageReceiver** 类：

    ```
    public MessageReceiver(String partitionId) {
        this.partitionId = partitionId;
    }
    ```

10. 将以下 **run** 方法添加到 **MessageReceiver** 类。此方法将创建 **EventHubReceiver** 实例以从事件中心数据分区进行读取。它会不断循环并在控制台中列显消息详细信息，直到 **stopThread** 为 true。

    ```
    public void run() {
      try {
        EventHubReceiver receiver = client.getConsumerGroup(null).createReceiver(partitionId, new EventHubEnqueueTimeFilter(now), Constants.DefaultAmqpCredits);
        System.out.println("** Created receiver on partition " + partitionId);
        while (!stopThread) {
          EventHubMessage message = EventHubMessage.parseAmqpMessage(receiver.receive(5000));
          if(message != null) {
            System.out.println("Received: (" + message.getOffset() + " | "
                + message.getSequence() + " | " + message.getEnqueuedTimestamp()
                + ") => " + message.getDataAsString());
          }
        }
        receiver.close();
      }
      catch(EventHubException e) {
        System.out.println("Exception: " + e.getMessage());
      }
    }
    ```

    > [AZURE.NOTE] 在创建开始运行后只读取发送到 IoT 中心的消息的接收方时，此方法将使用筛选器。这很适合测试环境，因为这样可以看到当前的消息集，但在生产环境中，代码应该要确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-processd2c-tutorial]教程。

11. 修改 **main** 方法的签名以添加如下所示的异常：

    ```
    public static void main( String[] args ) throws IOException
    ```

12. 在 **App** 类的 **main** 方法中添加以下代码。此代码将创建 **EventHubClient** 实例以连接到 IoT 中心上的事件中心兼容的终结点。然后，创建两个线程以从两个分区进行读取。将 **{youriothubkey}**、**{youreventhubcompatiblenamespace}** 和 **{youreventhubcompatiblename}** 替换为前面记下的值。**{youreventhubcompatiblenamespace}** 占位符的值来自事件中心兼容的终结点，其格式为 **xxxxnamespace.servicebus.chinacloudapi.cn**。

    ```
    String policyName = "iothubowner";
    String policyKey = "{youriothubkey}";
    String namespace = "{youreventhubcompatiblenamespace}";
    String name = "{youreventhubcompatiblename}";
    try {
      ConnectionStringBuilder csb = new ConnectionStringBuilder(policyName, policyKey, namespace);
      client = EventHubClient.create(csb.getConnectionString(), name);
    }
    catch(EventHubException e) {
        System.out.println("Exception: " + e.getMessage());
    }
    
    MessageReceiver mr0 = new MessageReceiver("0");
    MessageReceiver mr1 = new MessageReceiver("1");
    Thread t0 = new Thread(mr0);
    Thread t1 = new Thread(mr1);
    t0.start(); t1.start();

    System.out.println("Press ENTER to exit.");
    System.in.read();
    mr0.stopThread = true;
    mr1.stopThread = true;
    client.close();
    ```

    > [AZURE.NOTE] 此代码假设已在 F1（免费）层创建 IoT 中心。免费 IoT 中心有“0”和“1”这两个分区。如果使用另一种定价层创建 IoT 中心，则应调整代码，为每个分区创建 **MessageReceiver**。

13. 保存并关闭 App.java 文件。

14. 若要使用 Maven 生成 **read-d2c-messages** 应用程序，请在命令提示符下于 read-d2c-messages 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```



<!-- Links -->

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/
[lnk-processd2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

<!---HONumber=Mooncake_0321_2016-->