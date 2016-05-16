## 创建模拟设备应用程序

在本部分中，你将创建一个 Java 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 在“创建设备标识”部分中创建的 iot-java-get-started 文件夹中，于命令提示符下使用以下命令创建名为 **simulated-device** 的新 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符下，导航到新的 simulated-device 文件夹。

3. 使用文本编辑器，打开 simulated-device 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖性。这样，你便可以使用应用程序中的 iothub-java-client 包来与 IoT 中心通信，并将 Java 对象序列化为 JSON：

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
    private static String connString = "HostName={youriothubname}.azure-devices.net;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}";
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


10. 在 **App** 类中添加以下嵌套 **MessageSender** 类。此类中的 **run** 方法将生成发送到 IoT 中心的示例遥测数据，并在发送下一条消息之前等待确认：


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

13. 若要使用 Maven 生成 **simulated-device** 应用程序，请在命令提示符下于 simulated-device 文件夹中执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章[暂时性故障处理][lnk-transient-faults]中所述实现重试策略（例如指数退让）。

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!---HONumber=Mooncake_0321_2016-->