<properties
	pageTitle="IoT 中心设备管理设备作业 | Azure"
	description="Azure IoT 中心设备管理教程，描述如何使用设备作业来执行远程固件更新等操作。"
	services="iot-hub"
	documentationCenter=".net"
	authors="ellenfosborne"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>

# 教程：如何使用设备作业更新设备固件（预览版）

[AZURE.INCLUDE [iot-hub-device-management-job-selector](../includes/iot-hub-device-management-jobs-selector.md)]

## 介绍
Azure IoT 设备管理允许你使用设备作业与物理设备进行交互。一旦确定设备克隆（物理设备的服务表示形式），即可使用设备作业与其相应物理设备进行交互。设备作业允许对多台设备上的复杂进程进行协调。此进程可能包括多个步骤和长时间运行操作。

Azure IoT 中心设备管理当前提供 6 种类型的设备作业（如果客户需要，将增加额外的作业）：

- **固件更新**：更新物理设备上的固件（或 OS 映像）。
- **重启**：重启物理设备。
- **恢复出厂设置**：将物理设备上的固件（或 OS 映像）还原为设备上存储的出厂提供的备份映像。
- **配置更新**：配置在物理设备上运行的 IoT 中心客户端代理。
- **读取设备属性**：获取物理设备上的最新设备属性值。
- **写入设备属性**：更改物理设备上的设备属性。

有关如何使用上述每个作业的详细信息，请参阅 [API 文档][lnk-apidocs]。

可查询作业历史记录以了解已启动的作业的状态。有关示例查询，请参阅[our query expression library（我们的查询表达式库）][lnk-query-samples]。

## 使用设备作业执行固件更新：体系结构

下图演示了与单个物理设备进行交互的固件更新设备作业。

![][img-architecture]

固件更新设备作业的步骤如下：

1.  基于云的 IoT 解决方案启动固件更新设备作业，并提供固件包位置的 URI。IoT 解决方案负责将固件包放置在设备可下载的位置。

2.  物理设备接收到此 URI 时，将自动开始从所提供的 URI 进行下载。

3.  设备上的代码接收来自 IoT 中心的请求，并从提供的位置 URI 下载固件包。

4.  设备作业接收下载完成的设备状态消息后，即指示设备应用已下载的固件映像。

5.  设备应用固件映像后，将重启、重新连接到 IoT 中心，并通知 IoT 中心已应用新固件，设备作业就此完成。

## 运行固件更新示例

下面的示例扩展了 [Azure IoT 中心设备管理入门][lnk-get-started]教程的功能。从使各模拟设备运行开始，它将启动一个作业来更新所有模拟设备上的固件。

### 先决条件 

运行此示例之前，请务必完成 [Azure IoT 中心设备管理入门][lnk-get-started]中的步骤。这意味着必须运行模拟设备。如果你以前完成了该进程，现在请重启模拟设备。

### 启动示例

若要启动该示例，需要运行 `jobClient_scheduleJob.js`。这将在所有模拟设备上启动固件更新进程。按照以下步骤启动示例：

1.  从克隆 **azure-iot-sdks** 存储库的根文件夹，导航到 **azure-iot-sdks/node/service/samples** 目录。  

2.  打开 **jobClient\_scheduleJob.js** 并将占位符替换为你的 IoT 中心连接字符串。

4.  运行 `node jobClient_scheduleJob.js`。

应可看到命令行窗口中的输出显示了七个设备作业，它们跟踪着针对六个模拟设备的模拟固件更新。

### 启动设备作业

一般通过在 **JobClient** 实例上使用多个 **schedule&lt;JobName&gt;** 方法来启动设备作业。在固件更新示例中，我们调用 **scheduleFirmwareUpdate** 方法。我们传递具有 6 个设备 ID 的数组后，将启动 7 个设备作业，正在更新的每个设备各使用一个作业，另有一父设备作业用于跟踪这 6 个作业。

在下面的代码段中，启动了固件更新作业。调用将 **deviceId** 值的字符串数组当做参数，表示我们想要更新的设备。

```
jobClient.scheduleFirmwareUpdate(jobId, deviceIds, packageURI, timeoutInMinutes, callback);
```

### 设备模拟器实现详细信息

在前面的部分中，我们从服务角度演示了固件更新的实现细节（如何启动设备作业并查询其状态）。现在，我们将从设备方面讨论相应实现。

Azure IoT 中心设备管理客户端库处理设备与服务之间的通信，因此只需实现设备特定逻辑即可。它由两个部分组成：

- 实现设备特定的固件更新过程：这包括写入设备特定的逻辑以下载固件包，以及在下列适当回调中应用更新。在模拟的示例中，在[示例][lnk-github-firmware]中对此进行了实现：

  ```
  object_firmwareupdate *obj = get_firmwareupdate_object(0);
  obj->firmwareupdate_packageuri_write_callback = start_firmware_download;
  // platform specific code
  obj->firmwareupdate_update_execute_callback = start_firmware_update;
  //platform specific code
  ```

- 将固件更新过程告知服务：这涉及修改固件更新状态和固件更新结果字段。通过调用 **set\_firmwareupdate\_state** 和 **set\_firmwareupdate\_updateresult** API 实现此目的。在模拟的示例中，在[示例][lnk-github-firmware]中对此进行了实现：

## 后续步骤

若要了解有关 Azure IoT 中心设备管理功能的详细信息，可学习以下教程：

- Azure IoT 中心 DM 客户端库提供了使用 [Intel Edison 设备][lnk-edison]的端到端示例。

- [如何使用设备克隆][lnk-twin-tutorial]

- [如何使用查询查找设备克隆][lnk-tutorial-queries]

<!-- Images and links -->

[img-architecture]: ./media/iot-hub-device-management-device-jobs/image1.png
[img-output1]: ./media/iot-hub-device-management-device-jobs/image2.png
[img-output2]: ./media/iot-hub-device-management-device-jobs/image3.png
[img-properties]: ./media/iot-hub-device-management-device-jobs/image4.png

[lnk-apidocs]: /documentation/articles/iot-hub-sdks-summary/
[lnk-twin-tutorial]: /documentation/articles/iot-hub-device-management-device-twin/
[lnk-tutorial-queries]: /documentation/articles/iot-hub-device-management-device-query/
[lnk-edison]: https://github.com/Azure/azure-iot-sdks/tree/dmpreview/c/iotdm_client/samples/iotdm_edison_sample
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-github-firmware]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/c/iotdm_client/samples/iotdm_simple_sample/iotdm_simple_sample.c
[lnk-query-samples]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/doc/get_started/dm_queries/query-samples.md


<!---HONumber=Mooncake_0523_2016-->