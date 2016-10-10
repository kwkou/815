<properties
	pageTitle="处理 IoT 中心设备到云的消息 (Java) | Azure"
	description="遵照该 Java 教程了解处理 IoT 中心设备到云消息的有用模式。"
	services="iot-hub"
	documentationCenter="java"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="java"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/01/2016"
     ms.author="dobett"
     wacn.date="10/10/2016"/>

# 教程：如何使用 Java 处理 IoT 中心设备到云的消息

[AZURE.INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。其他教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息][lnk-c2d]）介绍了如何使用 IoT 中心的设备到云和云到设备的基本消息传递功能。

本教程以 [IoT 中心入门]中演示的代码为基础，呈现两种可用于处理设备到云消息的可缩放的模式：

- 在 [Azure Blob 存储]中可靠地存储设备到云的消息。常见的是 *冷路径* 分析，该情况下在 blob 中存储要输入到分析进程中的遥测数据。这些进程可由 [HDInsight (Hadoop)] 堆栈等工具驱动。

- 可靠地处理设备到云的 *交互式* 消息。如果设备到云的消息因为应用程序后端中的一组操作而立即触发，则表示这些消息是交互式的。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，*数据点* 消息则仅送入分析引擎。例如，设备中存储便于日后分析的温度遥测是数据点消息。

IoT 中心公开[事件中心][lnk-event-hubs]兼容的终结点来接收设备到云的消息，因此本教程使用 [EventProcessorHost] 实例。此实例：

* 在 Azure blob 存储中可靠地存储 *数据点* 消息。
* 将设备到云的 *交互式* 消息转发到 Azure [服务总线队列]进行即时处理。

服务总线可以帮助确保可靠处理交互式消息，因为它提供了各消息的检查点，以及基于时间范围的重复数据删除。

> [AZURE.NOTE] **EventProcessorHost** 实例只是其中一种处理交互式消息的方法。其他选项包括 [Azure Service Fabric][lnk-service-fabric] 和 [Azure 流分析][lnk-stream-analytics]。

在本教程最后，会运行 3 个 Java 控制台应用：

* **simulated-device**（[IoT 中心入门]教程中创建的应用的修改版本）每秒发送一次设备到云的数据点消息，每 10 秒发送一次设备到云的交互式消息。此应用使用 AMQPS 协议来与 IoT 中心通信。
* **process-d2c-messages** 使用 [EventProcessorHost] 类检索事件中心兼容的终结点中的消息。然后将数据点消息可靠地存储在 Azure Blob 存储中，将交互式消息转发到服务总线队列。
* **process-interactive-messages** 从服务总线队列中剔除交互式消息。

> [AZURE.NOTE] IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。若要了解如何将本教程中的模拟设备替换为物理设备，以及如何将设备连接到 IoT 中心，请参阅 [Azure IoT 开发人员中心]。

本教程直接适用于使用事件中心兼容消息的其他方式，如 [HDInsight (Hadoop)] 项目。有关详细信息，请参阅 [Azure IoT 中心开发人员指南 - 设备到云]。

若要完成本教程，您需要以下各项：

+ [IoT 中心入门]教程的完整工作版本。

+ Java SE 8。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Java。

+ Maven 3。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Maven。

+ 有效的 Azure 帐户。<br/>如果没有 Azure 订阅，只需几分钟即可创建一个[试用帐户](/pricing/1rmb-trial/)。

应具备 [Azure 存储]和 [Azure 服务总线]的一些基础知识。


## 从模拟设备发送交互式消息

在本部分中，会修改 [IoT 中心入门]教程中创建的模拟设备应用程序，向 IoT 中心发送设备到云的交互式消息。

1. 使用文本编辑器打开 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。本文件包含用于 [IoT 中心入门]教程中创建的 **simulated-device** 应用的代码。

2. 向 **App** 类添加以下嵌套类：

    ```
    private static class InteractiveMessageSender implements Runnable {
      public void run() {
        try {
          while (true) {
            String msgStr = "Alert message!";
            Message msg = new Message(msgStr);
            msg.setMessageId(java.util.UUID.randomUUID().toString());
            msg.setProperty("messageType", "interactive");
            System.out.println("Sending interactive message: " + msgStr);

            Object lockobj = new Object();
            EventCallback callback = new EventCallback();
            client.sendEventAsync(msg, callback, lockobj);

            synchronized (lockobj) {
              lockobj.wait();
            }
            Thread.sleep(10000);
          }
        } catch (InterruptedException e) {
          System.out.println("Finished sending interactive messages.");
        }
      }
    }
    ```

    此类类似于 **simulated-device** 项目中的 **MessageSender** 类。唯一的区别在于现在设置 **MessageId** 系统属性和 **messageType** 自定义属性。
    代码将向 **MessageId** 属性分配全局唯一标识符 (UUID)。服务总线可使用此标识符来删除收到的重复消息。本示例使用 **messageType** 属性来区分交互式消息和数据点消息。应用程序将在消息属性而不是在消息正文中传递此信息，因此事件处理器不需要反序列化消息来执行消息路由。

    > [AZURE.NOTE] 有必要在设备代码中创建用于删除重复交互式消息的 **MessageId**。间歇性网络通信或其他故障可能会导致多次重复传输来自该设备的相同消息。还可将 UUID 换用为语义消息 ID，如相关消息数据字段的哈希。

3. 修改 **main** 方法，按以下代码片段中所示发送交互式消息和数据点消息：

    ````
    MessageSender sender = new MessageSender();
    InteractiveMessageSender interactiveSender = new InteractiveMessageSender();

    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(sender);
    executor.execute(interactiveSender);
    ````

4. 保存并关闭 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施指数退让等重试策略。

5. 若要使用 Maven 生成 **simulated-device** 应用程序，请在 simulated-device 文件夹中的命令提示符下执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

## 处理设备到云的消息

在本部分中，将创建一个 Java 控制台应用，用于处理来自 IoT 中心的设备到云的消息。Iot 中心公开与[事件中心]兼容的终结点，让应用程序能读取设备到云的消息。本教程使用 [EventProcessorHost] 类来处理控制台应用中的这些消息。若要深入了解如何处理来自事件中心的消息，请参阅[事件中心入门]教程。

实现数据点消息的可靠存储或交互式消息的转发时，遇到的主要挑战是事件处理依赖消息使用者来提供进度的检查点。此外，为达到高吞吐量，应在从事件中心读取时大批量地提供检查点。如果失败且还原到先前的检查点，此方法可能重复处理大量消息。本教程将介绍如何将 Azure 存储空间写入及服务总线重复数据删除时间范围与 **EventProcessorHost** 检查点进行同步。

为了可靠地将消息写入 Azure 存储空间，本示例使用[块 blob][Azure Block Blobs] 的单个块提交功能。事件处理器将消息累积在内存中，直到应该提供检查点。例如，在消息的累积缓冲区达到 4 MB 的最大块大小之后，或者在超过服务总线重复数据删除时间范围之后。然后，在检查点之前，代码将新块提交到 Blob。

事件处理器使用事件中心消息偏移作为块 ID。借助此机制，事件处理器可在向存储空间提交新块之前执行重复数据删除检查，处理提交块和检查点之间可能发生的崩溃。

> [AZURE.NOTE] 本教程使用单个存储帐户来写入从 IoT 中心检索的所有消息。若要确定解决方案中是否需使用多个 Azure 存储帐户，请参阅 [Azure 存储可缩放性指导原则]。

应用程序利用服务总线重复数据删除功能，在处理交互式消息时避免重复项。模拟的设备向每个交互式消息标记唯一的 **MessageId**。借助此 ID，服务总线可确保在指定的重复数据删除时间范围内，仅向接收方发送一条带相同 **MessageId** 的消息。此重复数据删除功能和服务总线队列所提供的每一消息完成语义，使其能够很容易地实现可靠的交互消息处理。

为确保未在重复数据删除时间范围外重新提交任何消息，代码会将 **EventProcessorHost** 检查点机制与服务总线队列重复数据删除时间范围进行同步。同步方式是在每次超出重复数据删除时间范围时（本教程中为 1 小时），至少强制执行一次检查点。

> [AZURE.NOTE] 本教程使用单个分区服务总线队列来处理所有检索自 IoT 中心的交互式消息。若要深入了解如何使用服务总线队列来满足解决方案的扩展性要求，请参阅 [Azure 服务总线]文档。

### 预配 Azure 存储帐户和服务总线队列

若要使用 [EventProcessorHost] 类，必须具有 Azure 存储帐户以使该类能记录检查点信息。可使用现有的存储帐户，或按照[关于 About Azure 存储]中的说明来创建新帐户。请记下存储帐户连接字符串。

> [AZURE.NOTE] 复制和粘贴存储帐户连接字符串时，请务必不要包含空格。

你还需要服务总线队列来可靠处理交互式消息。可在 1 小时的重复数据删除时间范围内，以编程方式创建队列，如[如何使用服务总线队列][Service Bus queue]中所述。还可按以下步骤使用 [Azure 经典管理门户][lnk-classic-portal]：

1. 单击左下角的“新建”。然后单击“应用程序服务”>“服务总线”>“队列”>“自定义创建”。输入名称 **d2ctutorial**，选择区域，再使用现有命名空间或新建一个。记下命名空间名称，本教程稍后需要使用。在下一页中，选择“启用重复检测”，再将“重复检测历史记录期限”设为 1 小时。然后单击右下角的复选标记保存你的队列配置。

    ![在 Azure 门户中创建队列][30]

2. 在服务总线队列列表中单击“d2ctutorial”，然后单击“配置”。创建分别名为 **send** 和 **listen** 的共享访问策略，前者带“发送”权限，后者带“侦听”权限。记下这两个策略的**主键**，本教程稍后需要使用。完成后，单击底部的“保存”。

    ![在 Azure 门户中配置队列][31]

### 创建事件处理器

在本部分中，会创建一个 Java 应用程序，用于处理事件中心兼容的终结点发出的消息。

首先是添加名为 **process-d2c-messages** 的 Maven 项目，它会接收 IoT 中心事件中心兼容的终结点发出的设备到云的消息，并将这些消息路由到其他后端服务。

1. 在 [IoT 中心入门]教程中创建的 iot-java-get-started 文件夹中，在命令提示符处使用以下命令创建名为 **process-d2c-messages** 的 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=process-d2c-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符处，导航到新的 process-d2c-messages 文件夹。

3. 使用文本编辑器打开 process-d2c-messages 文件夹中的 pom.xml 文件，并向 **dependencies** 节点添加以下依赖项。借助这些依赖项，可使用应用程序中的 azure-eventhubs、azure-eventhubs-eph 和 azure-servicebus 包与 IoT 中心和服务总线队列进行交互：

    ```
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-eventhubs</artifactId>
      <version>0.8.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-eventhubs-eph</artifactId>
      <version>0.8.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-servicebus</artifactId>
      <version>0.9.4</version>
    </dependency>
    ```

4. 保存并关闭 pom.xml 文件。

接下来是向项目添加 **ErrorNotificationHandler** 类。

1. 使用文本编辑器创建 process-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\ErrorNotificationHandler.java 文件。向文件添加以下代码，以显示来自 **EventProcesssorHost** 实例的错误消息：

    ```
    package com.mycompany.app;

    import java.util.function.Consumer;
    import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;

    public class ErrorNotificationHandler implements
        Consumer<ExceptionReceivedEventArgs> {
      @Override
      public void accept(ExceptionReceivedEventArgs t) {
        System.out.println("EventProcessorHost: Host " + t.getHostname()
            + " received general error notification during " + t.getAction() + ": "
            + t.getException().toString());
      }
    }
    ```

2. 保存并关闭 ErrorNotificationHandler.java 文件。

现即可添加会实现 **IEventProcessor** 接口的类。**EventProcessorHost** 类调用此类来处理从 IoT 中心收到的设备到云的消息。此类中的代码实现逻辑，以在 Blob 容器中可靠地存储消息，并将交互式消息转送到服务总线队列。

**onEvents** 方法设置 **latestEventData** 变量，该变量会跟踪此事件处理器读取的最新消息的偏移量和序列号。请记住，每个处理器负责单个分区。然后，**onEvents** 方法从 IoT 中心接收一批消息，并按以下方式处理：发送交互式消息到服务总线队列，并将数据点消息附加到 **toAppend** 内存缓冲区。如果内存缓冲区达到 4 MB 的块限制，或者超过重复数据删除时间范围（本教程中为上个检查点后的 1 小时），该方法则会触发检查点。

**AppendAndCheckPoint** 方法首先为要附加到 blob 的块生成 **blockId**。Azure 存储要求所有块 ID 的长度均相同，以便此方法用前置零填补偏移。如果 blob 中已有带此 ID 的块，此方法会将其改为当前缓冲区的内容。

> [AZURE.NOTE] 为了简化代码，本教程使用了每个分区的单个 Blob 文件来存储消息。实际上，会在某段时间后或在文件达到特定大小后创建其他文件，从而实现文件滚动。请记住，Azure 块 blob 最多可容纳 195 GB 的数据。

再下来是实现 **IEventProcessor** 接口：

1. 使用文本编辑器创建 process-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\EventProcessor.java 文件。

2. 向 EventProcessor.java 文件添加以下导入和类定义。**EventProcessor** 实现可定义事件中心客户端行为的 **IEventProcessor** 接口：

    ```
    package com.mycompany.app;

    import java.io.ByteArrayInputStream;
    import java.io.ByteArrayOutputStream;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.nio.charset.StandardCharsets;
    import java.time.Duration;
    import java.time.Instant;
    import java.util.ArrayList;
    import java.util.Base64;
    import java.util.concurrent.ExecutionException;

    import com.microsoft.azure.eventhubs.EventData;
    import com.microsoft.azure.eventprocessorhost.*;
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;
    import com.microsoft.windowsazure.services.servicebus.*;
    import com.microsoft.windowsazure.services.servicebus.models.BrokeredMessage;

    public class EventProcessor implements IEventProcessor {

    }
    ```

3. 向 **EventProcessor** 类添加以下方式以使用 **IEventProcessor** 接口：

    ```
    @Override
    public void onOpen(PartitionContext context) throws Exception {
      System.out.println("EventProcessorHost: Partition "
          + context.getPartitionId() + " is opening");
    }

    @Override
    public void onClose(PartitionContext context, CloseReason reason)
        throws Exception {
      System.out.println("EventProcessorHost: Partition "
          + context.getPartitionId() + " is closing for reason "
          + reason.toString());
    }

    @Override
    public void onError(PartitionContext context, Throwable error) {
      System.out.println("EventProcessorHost: Partition "
          + context.getPartitionId() + " onError: " + error.toString());
    }

    @Override
    public void onEvents(PartitionContext context, Iterable<EventData> messages)
        throws Exception {
    }
    ```

4. 向 **EventProcessor** 类添加以下类级变量：

    ```
    public static CloudBlobContainer blobContainer;
    public static ServiceBusContract serviceBusContract;

    // Use a smaller MAX_BLOCK_SIZE value to test.
    final private int MAX_BLOCK_SIZE = 4 * 1024 * 1024;
    final private Duration MAX_CHECKPOINT_TIME = Duration.ofHours(1);

    private ByteArrayOutputStream toAppend = new ByteArrayOutputStream(
        MAX_BLOCK_SIZE);
    private Instant start = Instant.now();
    private EventData latestEventData;
    ```

5. 向 **EventProcessor** 类添加带以下签名的 **AppendAndCheckPoint** 方法：

    ```
    private void AppendAndCheckPoint(PartitionContext context)
      throws URISyntaxException, StorageException, IOException,
      IllegalArgumentException, InterruptedException, ExecutionException {
    }
    ```

6. 向 **AppendAndCheckPoint** 方法添加以下代码，用以检索分区中的当前消息偏移量和序列号：

    ```
    String currentOffset = latestEventData.getSystemProperties().getOffset();
    Long currentSequence = latestEventData.getSystemProperties().getSequenceNumber();
    System.out
        .printf(
            "\nAppendAndCheckPoint using partition: %s, offset: %s, sequence: %s\n",
            context.getPartitionId(), currentOffset, currentSequence);
    ```

7. 在 **AppendAndCheckPoint** 方法中，使用当前偏移值以创建下一个要存到 blob 的块的 **BlockEntry** 实例：

    ```
    Long blockId = Long.parseLong(currentOffset);
    String blockIdString = String.format("startSeq:%1$025d", blockId);
    String encodedBlockId = Base64.getEncoder().encodeToString(
        blockIdString.getBytes(StandardCharsets.US_ASCII));
    BlockEntry block = new BlockEntry(encodedBlockId);
    ```

8. 在 **AppendAndCheckPoint** 方法中，将最新消息集上传到块 blob 并检索当前的块列表：

    ```
    String blobName = String.format("iothubd2c_%s", context.getPartitionId());
    CloudBlockBlob currentBlob = blobContainer.getBlockBlobReference(blobName);

    currentBlob.uploadBlock(block.getId(),
        new ByteArrayInputStream(toAppend.toByteArray()), toAppend.size());
    ArrayList<BlockEntry> blockList = currentBlob.downloadBlockList();
    ```

9. 在 **AppendAndCheckPoint** 方法中，在新 blob 中创建初始块或附加块到现有 blob：

    ```
    if (currentBlob.exists()) {
      // Check if we should append new block or overwrite existing block
      BlockEntry last = blockList.get(blockList.size() - 1);
      if (blockList.size() > 0 && !last.getId().equals(block.getId())) {
        System.out.printf("Appending block %s to blob %s\n", blockId, blobName);
        blockList.add(block);
      } else {
        System.out.printf("Overwriting block %s in blob %s\n", blockId,
            blobName);
      }
    } else {
      System.out.printf("Creating initial block %s in new blob: %s\n", blockId,
          blobName);
      blockList.add(block);
    }
    currentBlob.commitBlockList(blockList);
    ```

10. 最后在 **AppendAndCheckPoint** 方法中，在分区上创建检查点，并准备好保存下一个消息块：

    ```
    context.checkpoint(latestEventData);

    // Reset everything after the checkpoint.
    toAppend.reset();
    start = Instant.now();
    System.out.printf("Checkpointed on partition id: %s\n",
        context.getPartitionId());
    ```

11. 在 **onEvents** 方法中，添加以下代码以接收来自 IoT 中心终结点的消息并将交互式消息转发给服务总线队列。然后，在块占用完或达到超时时调用 **AppendAndCheckPoint** 方法：

    ```
    if (messages != null) {
      for (EventData eventData : messages) {
        latestEventData = eventData;
        byte[] data = eventData.getBody();
        if (eventData.getProperties().containsKey("messageType")
            && eventData.getProperties().get("messageType")
                .equals("interactive")) {
          String messageId = (String) eventData.getSystemProperties().get(
              "message-id");
          BrokeredMessage message = new BrokeredMessage(data);
          message.setMessageId(messageId);
          serviceBusContract.sendQueueMessage("d2ctutorial", message);
          continue;
        }
        if (toAppend.size() + data.length > MAX_BLOCK_SIZE
            || Duration.between(start, Instant.now()).compareTo(
                MAX_CHECKPOINT_TIME) > 0) {
          AppendAndCheckPoint(context);
        }
        toAppend.write(data);
      }
    }
    ```

12. 最后在 **onEvents** 方法中，如果在 IoT 中心未发送任何消息时达到超时，则添加“else if”子句来调用 **AppendAndCheckPoint**：

    ```
    else if ((toAppend.size() > 0)
        && Duration.between(start, Instant.now())
            .compareTo(MAX_CHECKPOINT_TIME) > 0) {
      AppendAndCheckPoint(context);
    }
    ```

13. 将更改保存到 EventProcessor.java 文件。

**process-d2c-messages** 项目的最后一个任务是，向可实例化 **EventProcessorHost** 实例的 **main** 方法添加代码。

1. 使用文本编辑器打开 process-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

2. 在该文件中添加以下 **import** 语句：

    ```
    import com.microsoft.azure.eventprocessorhost.*;
    import com.microsoft.azure.servicebus.ConnectionStringBuilder;
    import com.microsoft.azure.storage.CloudStorageAccount;
    import com.microsoft.azure.storage.StorageException;
    import com.microsoft.azure.storage.blob.CloudBlobClient;
    import com.microsoft.windowsazure.Configuration;
    import com.microsoft.windowsazure.services.servicebus.ServiceBusConfiguration;
    import com.microsoft.windowsazure.services.servicebus.ServiceBusService;

    import java.net.URISyntaxException;
    import java.security.InvalidKeyException;
    import java.util.concurrent.*;
    ```

3. 向 **App** 类添加类级变量。将 **{yourstorageaccountconnectionstring}** 替换为之前在[预配 Azure 存储帐户和服务总线队列](#provision-an-azure-storage-account-and-a-service-bus-queue)部分中记下的 Azure 存储帐户连接字符串：

    ```
    private final static String storageConnectionString = "{yourstorageaccountconnectionstring}";
    ```

4. 向 **App** 类添加类级变量，并将 **{yourservicebusnamespace}** 替换为服务总线命名空间，将 **{yourservicebussendkey}** 替换为队列的 **send** 键。先前在[预配 Azure 存储帐户和服务总线队列](#provision-an-azure-storage-account-and-a-service-bus-queue)部分中记下了命名空间和 **listen** 键：

    ```
    private final static String serviceBusNamespace = "{yourservicebusnamespace}";
    private final static String serviceBusSasKeyName = "send";
    private final static String serviceBusSASKey = "{yourservicebussendkey}";
    private final static String serviceBusRootUri = ".servicebus.chinacloudapi.cn";
    ```

5. 将以下类级变量添加到 **App** 类。将 **{youreventhubcompatibleendpoint}** 替换为事件中心兼容的终结点名称。该终结点名称类似于 **ihs....namespace**，因此要删除 **sb://** 前缀和 **.servicebus.chinacloudapi.cn/** 后缀。将 **{youreventhubcompatiblename}** 替换为事件中心兼容的名称。将 **{youriothubkey}** 替换为 **iothubowner** 键。在 Java 版 Azure IoT 中心*教程的[创建 IoT 中心][lnk-create-an-iot-hub] section in the *部分中记下了这些值：

    ```
    private final static String consumerGroupName = "$Default";
    private final static String namespaceName = "{youreventhubcompatibleendpoint}";
    private final static String eventHubName = "{youreventhubcompatiblename}";
    private final static String sasKeyName = "iothubowner";
    private final static String sasKey = "{youriothubkey}";
    ```

6. 如下修改 **main** 方法的签名：

    ```
    public static void main(String args[]) throws InvalidKeyException,
      URISyntaxException, StorageException {
    }
    ```

7. 向 **main** 方法添加以下代码，获取到存储消息的 blob 容器的引用：

    ```
    System.out.println("Process D2C messages using EventProcessorHost");
    CloudStorageAccount account = CloudStorageAccount
        .parse(storageConnectionString);
    CloudBlobClient client = account.createCloudBlobClient();
    EventProcessor.blobContainer = client
        .getContainerReference("d2cjavatutorial");
    EventProcessor.blobContainer.createIfNotExists();
    ```

8. 向 **main** 方法添加以下代码，获取到服务总线服务的引用：

    ```
    Configuration config = ServiceBusConfiguration
        .configureWithSASAuthentication(serviceBusNamespace,
            serviceBusSasKeyName, serviceBusSASKey, serviceBusRootUri);
    EventProcessor.serviceBusContract = ServiceBusService.create(config);
    ```

9. 在 **main** 方法中，配置并创建 **EventProcessorHost** 实例。**setInvokeProcessorAfterReceiveTimeout** 选项确保即使没有要处理的消息，**EventProcessorHost** 实例也调用 **IEventProcessor** 接口中的 **onEvents** 方法。之后在达到超时时，该方法会始终调用 **AppendAndCheckPoint** 方法。

    ```
    ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder(
        namespaceName, eventHubName, sasKeyName, sasKey);
    EventProcessorHost host = new EventProcessorHost(eventHubName,
        consumerGroupName, eventHubConnectionString.toString(),
        storageConnectionString);
    EventProcessorOptions options = new EventProcessorOptions();
    options.setExceptionNotification(new ErrorNotificationHandler());
    options.setInvokeProcessorAfterReceiveTimeout(true);
    ```

10. 在 **main** 方法中，向 **EventProcessorHost** 实例注册 **IEventProcessor** 实现：

    ```
    try {
      System.out.println("Registering host named " + host.getHostName());
      host.registerEventProcessor(EventProcessor.class, options).get();
    } catch (Exception e) {
      System.out.print("Failure while registering: ");
      if (e instanceof ExecutionException) {
        Throwable inner = e.getCause();
        System.out.println(inner.toString());
      } else {
        System.out.println(e.toString());
      }
      System.out.println(e.toString());
    }
    ```

11. 最后，向 **main** 方法添加逻辑以关闭 **EventProcessorHost** 实例：

    ```
    System.out.println("Press enter to stop");
    try {
      System.in.read();
      host.unregisterEventProcessor();

      System.out.println("Calling forceExecutorShutdown");
      EventProcessorHost.forceExecutorShutdown(120);
    } catch (Exception e) {
      System.out.println(e.toString());
      e.printStackTrace();
    }

    System.out.println("End of sample");
    ```

12. 保存并关闭 process-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件夹。

13. 若要使用 Maven 生成 **process-d2c-messages** 应用程序，请在 process-d2c-messages 文件夹的命令提示符处执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

## 接收交互式消息

在本部分中，会编写一个 Java 控制台应用，用于接收来自服务总线队列的交互式消息。

首先是添加名为 **process-interactive-messages** 的 Maven 项目，它会接收服务总线队列上 **EventProcessor** 实例发出的消息。

1. 在 [IoT 中心入门]教程中创建的 iot-java-get-started 文件夹中，在命令提示符处使用以下命令创建名为 **process-interactive-messages** 的 Maven 项目。请注意，这是一条很长的命令：

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=process-interactive-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. 在命令提示符处，导航到新的 process-interactive-messages 文件夹。

3. 使用文本编辑器打开 process-interactive-messages 文件夹中的 pom.xml 文件，并向 **dependencies** 节点添加以下依赖项。借助该依赖项，可使用应用程序中的 azure-servicebus 包与服务总线队列进行交互：

    ```
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-servicebus</artifactId>
      <version>0.9.4</version>
    </dependency>
    ```

4. 保存并关闭 pom.xml 文件。

接下来是添加代码以检索服务总线队列中的消息。

1. 使用文本编辑器打开 process-interactive-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

2. 向文件添加以下 `import` 语句：

    ```
    import java.io.IOException;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;

    import com.microsoft.windowsazure.Configuration;
    import com.microsoft.windowsazure.exception.ServiceException;
    import com.microsoft.windowsazure.services.servicebus.*;
    import com.microsoft.windowsazure.services.servicebus.models.*;
    ```

3. 向 **App** 类添加以下类级变量，并将 **{yourservicebusnamespace}** 替换为服务总线命名空间，将 **{yourservicebuslistenkey}** 替换为队列的 **listen** 键。先前在[预配 Azure 存储帐户和服务总线队列](#provision-an-azure-storage-account-and-a-service-bus-queue)部分中记下了命名空间和 **listen** 键：

    ```
    private final static String serviceBusNamespace = "{yourservicebusnamespace}";
    private final static String serviceBusSasKeyName = "listen";
    private final static String serviceBusSASKey = "{yourservicebuslistenkey}";
    private final static String serviceBusRootUri = ".servicebus.chinacloudapi.cn";
    private final static String queueName = "d2ctutorial";
    private static ServiceBusContract service = null;
    ```

4. 向 **App** 类添加以下嵌套类以接收队列中的消息：

    ```
    private static class MessageReceiver implements Runnable {
      public void run() {
        ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
        try {
          while (true) {
            ReceiveQueueMessageResult resultQM = service.receiveQueueMessage(
                queueName, opts);
            BrokeredMessage message = resultQM.getValue();
            if (message != null && message.getMessageId() != null) {
              System.out.println("MessageID: " + message.getMessageId());
              System.out.print("From queue: ");
              byte[] b = new byte[200];
              String s = null;
              int numRead = message.getBody().read(b);
              while (-1 != numRead) {
                s = new String(b);
                s = s.trim();
                System.out.print(s);
                numRead = message.getBody().read(b);
              }
              System.out.println();
            } else {
              Thread.sleep(1000);
            }
          }
        } catch (InterruptedException e) {
          System.out.println("Finished.");
        } catch (ServiceException e) {
          System.out.println("ServiceException: " + e.getMessage());
        } catch (IOException e) {
          System.out.println("IOException: " + e.getMessage());
        }
      }
    }
    ```

5. 如下修改 **main** 方法的签名：

    ```
    public static void main(String args[]) throws ServiceException, IOException {
    }
    ```

6. 在 **main** 方法中，添加以下代码，开始侦听新消息：

    ```
    System.out.println("Process interactive messages");

    Configuration config = ServiceBusConfiguration
        .configureWithSASAuthentication(serviceBusNamespace,
            serviceBusSasKeyName, serviceBusSASKey, serviceBusRootUri);
    service = ServiceBusService.create(config);

    MessageReceiver receiver = new MessageReceiver();

    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(receiver);

    System.out.println("Press ENTER to exit.");
    System.in.read();
    executor.shutdownNow();
    ```

7. 保存并关闭 process-interactive-messages\\src\\main\\java\\com\\mycompany\\app\\App.java 文件夹。

8. 若要使用 Maven 生成 **process-interactive-messages** 应用程序，请在 process-interactive-messages 文件夹的命令提示符处执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

## 运行应用程序

现在即可运行 3 个应用程序。

1.	若要运行 **process-interactive-messages** 应用程序，请在命令提示符或外壳处导航到 process-interactive-messages 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 process-interactive-messages][processinteractive]

2.	若要运行 **process-d2c-messages** 应用程序，请在命令提示符或外壳处导航到 process-d2c-messages 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 process-d2c-messages][processd2c]

3.	若要运行 **simulated-device** 应用程序，请在命令提示符或外壳处导航到 simulated-device 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 simulated-device][simulateddevice]

> [AZURE.NOTE] 若要查看 blob 文件中的更新，需要将 **StoreEventProcessor** 类中的 **MAX\_BLOCK\_SIZE** 常量降为较小值，例如 **1024**。此更改很有用，原因是模拟设备发出的数据需要一些时间才能达到块大小限制。块大小更小时，可更快查看正创建和更新的 blob。但是，使用较大的块可以提高应用程序的可缩放性。

## 后续步骤

本教程介绍了如何使用 [EventProcessorHost] 类可靠地处理设备到云的数据点消息和交互式消息。

[如何使用 IoT 中心发送云到设备的消息][lnk-c2d]介绍如何从后端向设备发送消息。

若要查看使用 IoT 中心完成端到端解决方案的示例，请参阅 [Azure IoT 套件][lnk-suite]。

若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

<!-- Images. -->
[simulateddevice]: ./media/iot-hub-java-java-process-d2c/runsimulateddevice.png
[processinteractive]: ./media/iot-hub-java-java-process-d2c/runprocessinteractive.png
[processd2c]: ./media/iot-hub-java-java-process-d2c/runprocessd2c.png

[30]: ./media/iot-hub-java-java-process-d2c/createqueue2.png
[31]: ./media/iot-hub-java-java-process-d2c/createqueue3.png

<!-- Links -->

[Azure Blob 存储]: /documentation/articles/storage-dotnet-how-to-use-blobs/
[Azure 数据工厂]: /documentation/services/data-factory/
[HDInsight (Hadoop)]: /documentation/services/hdinsight/
[Service Bus queue]: /documentation/articles/service-bus-dotnet-get-started-with-queues/
[服务总线队列]: /documentation/articles/service-bus-dotnet-get-started-with-queues/

[Azure IoT 中心开发人员指南 - 设备到云]: /documentation/articles/iot-hub-devguide/#d2c

[Azure 存储]: /documentation/services/storage/
[Azure 服务总线]: /documentation/services/service-bus/

[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT 中心入门]: /documentation/articles/iot-hub-java-java-getstarted/
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: /documentation/services/service-fabric/
[lnk-stream-analytics]: /documentation/services/stream-analytics/
[lnk-event-hubs]: /documentation/services/event-hubs/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh675232.aspx

<!-- Links -->
[关于 About Azure 存储]: /documentation/articles/storage-create-storage-account/#create-a-storage-account
[事件中心入门]: /documentation/articles/event-hubs-java-ephjava-getstarted/
[Azure 存储可缩放性指导原则]: /documentation/articles/storage-scalability-targets/
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[事件中心]: /documentation/articles/event-hubs-overview/
[EventProcessorHost]: https://github.com/Azure/azure-event-hubs/tree/master/java/azure-eventhubs-eph
[Event Hubs Programming Guide]: https://github.com/Azure/azure-event-hubs/blob/master/java/readme.md
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-c2d]: /documentation/articles/iot-hub-java-java-process-d2c/
[lnk-suite]: /documentation/suites/iot-suite/

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/java-devbox-setup.md
[lnk-create-an-iot-hub]: /documentation/articles/iot-hub-java-java-getstarted/#create-an-iot-hub

<!---HONumber=Mooncake_0926_2016-->