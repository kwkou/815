<properties
    pageTitle="Azure IoT 中心的云到设备的消息 (Java) | Azure"
    description="如何使用 Azure IoT SDK for Java 将云到设备的消息从 Azure IoT 中心发送到设备。修改模拟设备应用以接收云到设备消息，并修改后端应用以发送云到设备消息。"
    services="iot-hub"
    documentationcenter="java"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="7f785ea8-e7c2-40c5-87ef-96525e9b9e1e"
    ms.service="iot-hub"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/23/2016"
    wacn.date="01/13/2017"
    ms.author="dobett" />  


# 使用 IoT 中心发送云到设备的消息 \(Java\)

[AZURE.INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

## 介绍
Azure IoT 中心是一项完全托管的服务，有助于在数百万台设备和单个解决方案后端之间实现安全可靠的双向通信。[IoT 中心入门]教程介绍了如何创建 IoT 中心和在其中预配设备标识，并介绍了如何编写用于发送设备到云消息的模拟设备应用。

本教程是在 [Get started with IoT Hub]（IoT 中心入门）的基础上制作的。其中了说明了如何：

* 通过 IoT 中心，将云到设备的消息从解决方案后端发送到单个设备。
* 在设备上接收云到设备的消息。
* 通过解决方案后端，请求确认收到从 IoT 中心发送到设备的消息（*反馈*）。

若要深入了解云到设备的消息，可参阅 [IoT 中心开发人员指南][IoT Hub developer guide - C2D]。

在本教程的最后，将运行两个 Java 控制台应用：

* **simulated-device**，这是在 [Get started with IoT Hub]（IoT 中心入门）中创建的应用的修改版本，可连接到 IoT 中心并接收云到设备的消息。
* **send-c2d-messages**，它通过 IoT 中心将云到设备的消息发送到模拟设备应用，然后接收其传送确认。

> [AZURE.NOTE]
IoT 中心通过 Azure IoT 设备 SDK 对许多设备平台和语言（包括 C、Java 和 Javascript）提供 SDK 支持。有关如何将设备连接到本教程中的代码（通常是连接到 Azure IoT 中心）的逐步说明，请参阅 [Azure IoT Developer Center]（Azure IoT 开发人员中心）。
> 
> 

若要完成本教程，需具备以下先决条件：

+ Java SE 8。<br/>

+ Maven 3。<br/>

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

## 在模拟设备应用上接收消息
在本部分中，将修改 [IoT 中心入门]中创建的模拟设备应用，接收来自 IoT 中心的云到设备消息。

1. 使用文本编辑器打开 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

2. 在 **App** 类中添加以下 **MessageCallback** 类作为嵌套类。设备从 IoT 中心接收消息时，将调用 **execute** 方法。在本示例中，设备始终在其完成消息时通知 IoT 中心：

    
        private static class MessageCallback implements
        com.microsoft.azure.iothub.MessageCallback {
          public IotHubMessageResult execute(Message msg, Object context) {
            System.out.println("Received message from hub: "
              + new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET));
    
            return IotHubMessageResult.COMPLETE;
          }
        }
    

3. 修改 **main** 方法以创建 **MessageCallback** 实例，并在打开客户端之前调用 **setMessageCallback**，如下所示：

    
        client = new DeviceClient(connString, protocol);
    
        MessageCallback callback = new MessageCallback();
        client.setMessageCallback(callback, null);
        client.open();
    
   > [AZURE.NOTE]
   如果使用 HTTP（而不使用 MQTT 或 AMQP）作为传输，**DeviceClient** 实例不会经常检查 IoT 中心发来的消息（时间间隔小于 25 分钟）。若要深入了解 MQTT、AMQP 和 HTTP 支持间的差异以及 IoT 中心限制，请参阅 [IoT 中心开发人员指南][IoT Hub developer guide - C2D]。
   > 
   > 

## 发送云到设备的消息
在本部分中，会创建 Java 控制台应用，用于向模拟设备应用发送云到设备的消息。需使用 [IoT 中心入门]教程中添加的设备的设备 ID。还需要中心的 IoT 中心连接字符串（位于 [Azure 门户预览]）。

1. 在命令提示符处，使用以下命令创建名为 **send-c2d-messages** 的 Maven 项目。请注意，这是一条很长的命令：

    
        mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=send-c2d-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    

2. 在命令提示符下，导航到新的 send-c2d-messages 文件夹。

3. 使用文本编辑器，打开 send-c2d-messages 文件夹中的 pom.xml 文件，并在 **dependencies** 节点中添加以下依赖项。这样即可使用应用程序中的 **iothub-java-service-client** 包来与 IoT 中心服务通信：

    
        <dependency>
          <groupId>com.microsoft.azure.iothub-java-client</groupId>
          <artifactId>iothub-java-service-client</artifactId>
          <version>1.0.10</version>
        </dependency>
    

4. 保存并关闭 pom.xml 文件。

5. 使用文本编辑器打开 send-c2d-messages\src\main\java\com\mycompany\app\App.java 文件。

6. 在该文件中添加以下 **import** 语句：

    
        import com.microsoft.azure.iot.service.sdk.*;
        import java.io.IOException;
        import java.net.URISyntaxException;
    

7. 将以下类级变量添加到 **App** 类，并将 **{yourhubconnectionstring}** 和 **{yourdeviceid}** 替换为前面记下的值：

    
        private static final String connectionString = "{yourhubconnectionstring}";
        private static final String deviceId = "{yourdeviceid}";
        private static final IotHubServiceClientProtocol protocol = IotHubServiceClientProtocol.AMQP;
    
    
8. 将 **main** 方法替换为以下用于连接到 IoT 中心的代码，将消息发送到设备，然后等待设备已接收并处理消息的通知：

    
        public static void main(String[] args) throws IOException,
            URISyntaxException, Exception {
          ServiceClient serviceClient = ServiceClient.createFromConnectionString(
            connectionString, protocol);
          
          if (serviceClient != null) {
            serviceClient.open();
            FeedbackReceiver feedbackReceiver = serviceClient
              .getFeedbackReceiver(deviceId);
            if (feedbackReceiver != null) feedbackReceiver.open();
    
            Message messageToSend = new Message("Cloud to device message.");
            messageToSend.setDeliveryAcknowledgement(DeliveryAcknowledgement.Full);
    
            serviceClient.send(deviceId, messageToSend);
            System.out.println("Message sent to device");
    
            FeedbackBatch feedbackBatch = feedbackReceiver.receive(10000);
            if (feedbackBatch != null) {
              System.out.println("Message feedback received, feedback time: "
                + feedbackBatch.getEnqueuedTimeUtc().toString());
            }
    
            if (feedbackReceiver != null) feedbackReceiver.close();
            serviceClient.close();
          }
        }
    
   > [AZURE.NOTE]
   为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。
   > 
   > 

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 simulated-device 文件夹的命令提示符处，运行以下命令以发送遥测数据至 IoT 中心并侦听中心发出的云到设备消息：

    
        mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    

    ![运行模拟设备应用][img-simulated-device]

2. 在 send-c2d-messages 文件夹的命令提示符处，运行以下命令以发送云到设备的消息并等待反馈确认：

    
        mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    

    ![运行命令以发送云到设备的消息][img-send-command]

## 后续步骤

在本教程中，你已学习如何发送和接收云到设备的消息。

若要查看使用 IoT 中心完成端到端解决方案的示例，请参阅 [Azure IoT 套件]。

若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

<!-- Images -->
[img-simulated-device]: ./media/iot-hub-java-java-c2d/receivec2d.png
[img-send-command]: ./media/iot-hub-java-java-c2d/sendc2d.png
<!-- Links -->

[Get started with IoT Hub]: /documentation/articles/iot-hub-java-java-getstarted/
[IoT 中心入门]: /documentation/articles/iot-hub-java-java-getstarted/
[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide-messaging/
[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[Azure IoT Developer Center]: /develop/iot
[lnk-free-trial]: /pricing/1rmb-trial/

[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Azure 门户预览]: https://portal.azure.cn
[Azure IoT 套件]: /documentation/services/iot-suite/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and code-->